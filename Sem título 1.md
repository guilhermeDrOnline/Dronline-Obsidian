<?php

  

namespace App\Jobs\Faturamento\queues;

  

use App\Adapters\RetornoSolicitacaoProcedimento;

use App\Adapters\RetornoSolicitacaoProcedimentoIpasgo;

use App\Adapters\RetornoSolicitacaoProcedimentoLeve;

use App\Adapters\SolicitacaoProcedimento;

use App\Adapters\SolicitacaoProcedimentoBestSenior;

use App\Adapters\SolicitacaoProcedimentoIpasgo;

use App\DadosFaturamento;

use App\Http\Controllers\UtilController;

use App\Jobs\Faturamento\commands\ObterAutorizacaoCommand;

use App\LogFaturamento;

use App\StatusFaturamento;

use App\Subcontrato;

use Illuminate\Bus\Queueable;

use Illuminate\Queue\SerializesModels;

use Illuminate\Queue\InteractsWithQueue;

use Illuminate\Contracts\Queue\ShouldQueue;

use Illuminate\Foundation\Bus\Dispatchable;

use SoapClient;

  

class SolicitacaoAutorizacao implements ShouldQueue

{

use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

  

private $subcontrato_id;

private $carteirinha;

private $codigo_atendimento;

private $dado_atendimento_id;

private $base_url;

private $tipo;

private $nomeProfissional;

private $conselhoProfissional;

private $numeroConselhoProfissional;

private $UF;

private $CBOS;

private $item_lote_id;

private $lote_faturamento_id;

private $indicacao_clinica;

private $eletivo;

  

/**

* Create a new job instance.

*

* @return void

*/

public function __construct(

$subcontrato_id,

$carteirinha,

$codigo_atendimento,

$dado_atendimento_id,

$nomeProfissional,

$conselhoProfissional,

$numeroConselhoProfissional,

$UF,

$CBOS,

$item_lote_id,

$lote_faturamento_id,

$indicacao_clinica,

$eletivo

) {

$this->subcontrato_id = $subcontrato_id;

$this->carteirinha = $carteirinha;

$this->dado_atendimento_id = $dado_atendimento_id;

$this->codigo_atendimento = $codigo_atendimento ?? UtilController::getNumerosFaturamento(12);

$this->nomeProfissional = $nomeProfissional;

$this->conselhoProfissional = $conselhoProfissional;

$this->numeroConselhoProfissional = $numeroConselhoProfissional;

$this->UF = $UF;

$this->CBOS = $CBOS;

$this->item_lote_id = $item_lote_id;

$this->lote_faturamento_id = $lote_faturamento_id;

$this->indicacao_clinica = $indicacao_clinica ?? '';

$this->eletivo = $eletivo ?? false;

}

  

/**

* Execute the job.

*

* @return void

*/

public function handle()

{

$subcontrato = Subcontrato::where('id', $this->subcontrato_id)->whereNotNull('faturamento_config')->where('habilitar_integracao_saw', true)->first();

  

$faturamento_config = json_decode(json_decode($subcontrato->faturamento_config));

$this->base_url = $faturamento_config->url->solicitacaoProcedimento;

$this->tipo = $faturamento_config->url->tipoSolicitacaoProcedimento;

  

if ($this->eletivo) {

$codigoTabela = $faturamento_config->especialidades->codigoTabela;

$codigoProcedimento = $faturamento_config->especialidades->codigoProcedimento;

$descricaoProcedimento = $faturamento_config->especialidades->descricaoProcedimento;

} else {

$codigoTabela = $faturamento_config->pronto_atendimento->codigoTabela;

$codigoProcedimento = $faturamento_config->pronto_atendimento->codigoProcedimento;

$descricaoProcedimento = $faturamento_config->pronto_atendimento->descricaoProcedimento;

}

  

$_cbos = preg_replace('/\D/', '225125', $this->CBOS);

$_cbos = str_pad(substr($_cbos, 0, 6), 6, '0', STR_PAD_LEFT);

  

try {

$parameters = new ObterAutorizacaoCommand(

'SOLICITACAO_PROCEDIMENTOS',

$this->codigo_atendimento,

$faturamento_config->dados_gerais->codigoPrestadorNaOperadora ?? '',

$faturamento_config->dados_gerais->registroANS ?? '',

$faturamento_config->dados_gerais->Padrao ?? '',

$faturamento_config->autenticacao->loginPrestador ?? '',

$faturamento_config->autenticacao->senhaPrestador ?? '',

UtilController::getNumerosFaturamento(20),

$faturamento_config->dados_gerais->tipoEtapaAutorizacao ?? '',

$this->carteirinha,

$faturamento_config->dados_gerais->atendimentoRN ?? '',

$faturamento_config->dados_gerais->nomeContratadoSolicitante ?? '',

  

// DADOS PROFISSIONAL

strtoupper(UtilController::stripAccents($this->nomeProfissional ?? '')),

$this->conselhoProfissional ?? '',

$this->numeroConselhoProfissional ?? '',

$this->UF ?? '',

$_cbos ?? '',

  

$faturamento_config->dados_gerais->caraterAtendimento ?? '',

$codigoTabela ?? '',

$codigoProcedimento ?? '',

$descricaoProcedimento ?? '',

1, // quantidade de procedimento (apenas 1)

$faturamento_config->dados_gerais->codigoPrestadorNaOperadora ?? '',

$faturamento_config->dados_gerais->CNES ?? '',

strtoupper(UtilController::stripAccents($this->indicacao_clinica ?? ''))

);

} catch (\Throwable $e) {

logger()->error('Erro ao instanciar ObterAutorizacaoCommand: ' . $e->getMessage(), [

'subcontrato_id' => $this->subcontrato_id,

'dado_atendimento_id' => $this->dado_atendimento_id,

]);

  

return;

}

logger()->info(

'Iniciando solicitação de autorização para o atendimento ID: ' . $this->dado_atendimento_id,

(array) $parameters

);

  

switch ($this->subcontrato_id) {

case '748':

$solicitacaoProcedimento = new SolicitacaoProcedimentoBestSenior($parameters);

break;

case '750':

$solicitacaoProcedimento = new SolicitacaoProcedimentoIpasgo($parameters);

break;

default:

$solicitacaoProcedimento = new SolicitacaoProcedimento($parameters);

}

  

$xml = $solicitacaoProcedimento->gerarXML();

  

logger()->info('XML Gerado: ' . $xml);

  

$this->solicitar($xml);

}

private function solicitar($xml)

{

$endpoint = preg_replace('/\?.*$/', '', $this->base_url);

$soapAction = $this->tipo;

  

$connTimeout = 300;

$readTimeout = 300;

  

$options = [

'trace' => true,

'exceptions' => true,

'connection_timeout' => $connTimeout,

'location' => $endpoint, // endereço do serviço

'uri' => 'http://www.w3.org/2003/05/soap-envelope', // namespace SOAP 1.2 padrão (ajuste se necessário)

'use' => SOAP_LITERAL,

'style' => SOAP_DOCUMENT,

];

  

ini_set('default_socket_timeout', (string) $readTimeout);

  

try {

// cria client em modo non-WSDL

$client = new \SoapClient(null, $options);

  

// envia o XML manualmente

$responseXml = $client->__doRequest(

$xml,

$endpoint,

$soapAction,

SOAP_1_1

);

  

$this->gerarLog($xml, $responseXml, 'solicitacao_procedimento');

} catch (\SoapFault $e) {

logger()->error('SoapFault (non-WSDL): ' . $e->getMessage(), [

'endpoint' => $endpoint,

'action' => $soapAction,

]);

$this->gerarLogErro($xml, $e);

} catch (\Exception $e) {

logger()->error('Exception (non-WSDL): ' . $e->getMessage(), [

'endpoint' => $endpoint,

'action' => $soapAction,

]);

$this->gerarLogErro($xml, $e);

}

}

  

private function gerarLogErro($xml, \Throwable $e)

{

$statusFat = StatusFaturamento::find(StatusFaturamento::ERRO);

  

DadosFaturamento::where('id', $this->dado_atendimento_id)->update([

'autorizacao' => false,

'sequencia_lote_auth' => $this->codigo_atendimento,

'liberado_envio_guia' => false,

'liberado_com_erro' => true,

'erros_guia' => 'Erro na solicitação: ' . $e->getMessage(),

'ultimo_status_id' => $statusFat->id,

]);

  

$novoLog = new LogFaturamento();

$novoLog->tipo = 'solicitacao_procedimento';

$novoLog->request = json_encode(['request' => $xml]);

$novoLog->response = json_encode(['response' => '', 'error' => $e->getMessage()]);

$novoLog->dados_faturamento_id = $this->dado_atendimento_id;

$novoLog->processado = false;

$novoLog->save();

}

  

private function gerarLog($xml, $resposta, $tipo)

{

$statusFat = StatusFaturamento::find(StatusFaturamento::FINALIZADO);

  

DadosFaturamento::where('id', $this->dado_atendimento_id)->update([

'autorizacao' => true,

'sequencia_lote_auth' => $this->codigo_atendimento,

'liberado_envio_guia' => true,

'liberado_com_erro' => false,

'ultimo_status_id' => $statusFat->id,

]);

  

$novoLog = new LogFaturamento();

$novoLog->tipo = $tipo;

$novoLog->request = json_encode(['request' => $xml]);

$novoLog->response = json_encode(['response' => $resposta]);

$novoLog->dados_faturamento_id = $this->dado_atendimento_id;

$novoLog->processado = true;

$novoLog->save();

  

switch ($this->subcontrato_id) {

case '750':

$retornoSolicitacaoProcedimento = new RetornoSolicitacaoProcedimentoIpasgo(

$resposta,

true,

$tipo,

$this->dado_atendimento_id,

$this->codigo_atendimento

);

break;

case '703':

$retornoSolicitacaoProcedimento = new RetornoSolicitacaoProcedimentoLeve(

$resposta,

true,

$tipo,

$this->dado_atendimento_id,

$this->codigo_atendimento

);

break;

default:

$retornoSolicitacaoProcedimento = new RetornoSolicitacaoProcedimento(

$resposta,

true,

$tipo,

$this->dado_atendimento_id,

$this->codigo_atendimento

);

}

$retornoSolicitacaoProcedimento->gerarRetorno();

  

}

}