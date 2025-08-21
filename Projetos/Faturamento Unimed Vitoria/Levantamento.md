Será criado um novo fluxo para solicitação de autorização onde irá gerar uma senha para que possamos enviar posteriormente no faturamento.

[[RN-AUT-8.56.0 (1) (1).pdf]]

### Desenho do fluxo de integração Fase 1
[[Faturamento Unimed Vitoria.excalidraw]]

## EndPoints

Credenciais válidas para obtenção de token:

- **POST** `https://nucleos-aut-wsd.unimedvitoria.com.br/api/UnioToken`
- Body (exemplo):
```
{ 
	"applicationIdFrom": "df706f5f-8e91-463c-b933-2ca25727d904",
    "username": "drOnlineTeste",
    "password": "123456",
    "loginType": 1
}

```

Endpoint de solicitação de senha:

EndPoint de envio de faturamento:
## pontos levantados

- Exemplo do json, pois a imagens está péssima, não da para identificar a estrutura ( preferencia um curl)
- No documento não tem os endpoints de autorização e de envio do faturamento, apenas o de token.
- O envio da solicitação será feita no final do atendimento?
- Qual será a estrutura de retorno, teremos Glosa para ser tratada?
- Caso for necessário cancelar uma solicitação da senha, tem endpoint para isso?