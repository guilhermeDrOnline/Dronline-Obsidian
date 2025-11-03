<?php

  

namespace App\Jobs\Faturamento\SolicitacaoProcedimento;

  

use App\DadosFaturamento;

use App\Http\Controllers\UtilController;

use App\Jobs\Faturamento\queues\SolicitacaoAutorizacao;

use App\LoteFaturamento;

use App\Models\Lobby;

use App\StatusFaturamento;

use App\Subcontrato;

use Illuminate\Bus\Queueable;

use Illuminate\Contracts\Queue\ShouldQueue;

use Illuminate\Foundation\Bus\Dispatchable;

use Illuminate\Queue\InteractsWithQueue;

use Illuminate\Queue\SerializesModels;

use Illuminate\Support\Facades\DB;

  

class GerarSolicitacaoProcedimento implements ShouldQueue

{

use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

protected $lobby_id;

public function __construct($lobby_id)

{

$this->lobby_id = $lobby_id;

}

  

/**

* Execute the job.

*

* @return void

*/

public function handle()

{

$atendimentos = Lobby::withTrashed()

->leftJoin('atendimentos', 'atendimentos.id', 'lobby.atendimento_id')

->join('atendimento_cid', 'atendimento_cid.atendimento_id', 'atendimentos.id')

->join('cid_sub_categoria', 'cid_sub_categoria.id', 'atendimento_cid.cid_id')

->join('lobby_cargo_empresas', 'lobby_cargo_empresas.lobby_id', 'lobby.id')

->leftJoin('profissional_documento', 'profissional_documento.profissional_id', 'lobby_cargo_empresas.cargo_empresa_id')

->leftJoin('documentos', 'documentos.id', 'profissional_documento.documento_id')

->leftJoin('conselhos', 'conselhos.id', 'documentos.conselho_id')

->leftJoin('estados', 'estados.id', 'conselhos.estado_id')

->leftJoin('cbos', 'cbos.id', 'lobby_cargo_empresas.cbo_id')

->join('clientes', 'clientes.id', 'lobby.cliente_id')

->join('contratos', 'contratos.id', 'clientes.contrato_id')

->leftJoin('cargo_empresas', 'cargo_empresas.id', 'lobby_cargo_empresas.cargo_empresa_id')

->leftJoin('especialidades', 'especialidades.id', 'atendimentos.especialidade_id')

->where('lobby.id', $this->lobby_id)

->where('cid_sub_categoria.cid_id', 'not like', 'Z53%')

->whereNotIn('cid_sub_categoria.cid_id', ['ADMIN01'])

->select(

'lobby.agendamento',

'lobby.faturado',

'lobby.identification as carteirinha',

'especialidades.id as especialidade_id',

'cargo_empresas.nome as profissional_nome',

'cargo_empresas.cpf as profissional_cpf',

'documentos.nr_documento as profissional_crm',

'conselhos.cod_conselho',

'estados.cd_ibge as profissional_crm_uf',

'cbos.codigo as profissional_cbo',

'lobby.telemedicina_session_id',

'lobby.id as lobby_id',

'contratos.subcontrato_id',

'patient_room_entry_time as horario_entrada',

'patient_room_departure_time as horario_saida',

DB::raw('TIMESTAMPDIFF(MINUTE, patient_room_entry_time, patient_room_departure_time) AS duracao_consulta'),

DB::raw("CONCAT(COALESCE(cid_sub_categoria.cid_id, ''), ' - ', COALESCE(cid_sub_categoria.descricao, '')) AS indicacao_clinica")

)

->first();

  

if (!$atendimentos) {

logger()->info("Nenhum atendimento encontrado para lobby_id: " . $this->lobby_id);

return;

}

  

$rolereturn = $this->roleReturn($atendimentos->subcontrato_id);

  

if (!$rolereturn) {

logger()->error("Subcontrato não encontrado", [

'subcontrato_id' => $atendimentos->subcontrato_id

]);

return;

}

  
  

if ($rolereturn->prazo_retorno )

  

$loteGuia = LoteFaturamento::where('subcontrato_id', $atendimentos->subcontrato_id)

->where('ultimo_status_id','<>', StatusFaturamento::ERRO_LOTE)

->where('eletivo', $atendimentos->agendamento)

->where('fechado', false)

->first();

  

logger()->info("Lote Guia", [

'loteGuia' => $loteGuia

]);

if (!$loteGuia) {

  

$statusFat = StatusFaturamento::find(StatusFaturamento::EM_ABERTO);

  

$diafat = Subcontrato::where('id', $atendimentos->subcontrato_id)->select('fechamento_dias')->first();

  

$loteGuia = new LoteFaturamento();

$loteGuia->subcontrato_id = $atendimentos->subcontrato_id;

$loteGuia->ano_ref = date('Y');

$loteGuia->mes_ref = date('m');

$loteGuia->dia_mes = $diafat->fechamento_dias;

$loteGuia->fechado = false;

$loteGuia->ultimo_status_id = $statusFat->id;

$loteGuia->eletivo = $atendimentos->agendamento;

$loteGuia->save();

}

  

if (!DadosFaturamento::where('lobby_id', $this->lobby_id)->exists()) {

  

$statusSolicitacao = StatusFaturamento::find(StatusFaturamento::EM_ABERTO);

  

$novoDado = new DadosFaturamento();

$novoDado->lobby_id = $atendimentos->lobby_id;

$novoDado->subcontrato_id = $atendimentos->subcontrato_id;

$novoDado->telemedicina_session_id = $atendimentos->telemedicina_session_id;

  

$novoDado->carteirinha = explode('_', $atendimentos->carteirinha)[0];

$novoDado->profissional_nome = $atendimentos->profissional_nome;

$novoDado->profissional_cpf = $atendimentos->profissional_cpf;

$novoDado->profissional_crm = $atendimentos->profissional_crm;

$novoDado->cod_conselho = $atendimentos->cod_conselho;

$novoDado->profissional_crm_uf = $atendimentos->profissional_crm_uf;

$novoDado->profissional_cbo = $atendimentos->profissional_cbo ?? '225125';

$novoDado->especialidade_id = $atendimentos->especialidade_id;

  

$novoDado->horario_entrada = $atendimentos->horario_entrada;

$novoDado->horario_saida = $atendimentos->horario_saida;

$novoDado->duracao_consulta = $atendimentos->duracao_consulta;

$novoDado->indicacao_clinica = $atendimentos->indicacao_clinica;

$novoDado->lote_faturamento_id = $loteGuia->id;

$novoDado->ultimo_status_id = $statusSolicitacao->id;

$novoDado->sequencial = DadosFaturamento::where('lote_faturamento_id', $loteGuia->id)->count() + 1;

$novoDado->eletivo = $atendimentos->agendamento;

$novoDado->save();

  

if ($novoDado->sequencial > 97) {

  

$statusFat = StatusFaturamento::find(StatusFaturamento::EM_PROCESSAMENTO);

  

$loteGuia->fechado = true;

$loteGuia->ultimo_status_id = $statusFat->id;

$loteGuia->save();

  

}

  

$cd_atendimento = UtilController::getNumerosFaturamento(12);

  

SolicitacaoAutorizacao::dispatch(

$novoDado->subcontrato_id,

$novoDado->carteirinha,

$cd_atendimento,

$novoDado->id,

$novoDado->profissional_nome,

$novoDado->cod_conselho,

$novoDado->profissional_crm,

$novoDado->profissional_crm_uf,

$novoDado->profissional_cbo,

$novoDado->item_lote_id,

$novoDado->lote_faturamento_id,

$novoDado->indicacao_clinica,

$novoDado->eletivo

)

->onConnection('faturamento')->delay(now()->addMinutes(1));

  

Lobby::where('id', $this->lobby_id)->update(['faturado' => true]);

  

}

}

private function roleReturn($subcontrato_id)

{

$roles = Subcontrato::find($subcontrato_id);

  

}

}