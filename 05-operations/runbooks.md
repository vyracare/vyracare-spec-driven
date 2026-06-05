# Runbooks e Diagnostico

## 1. Erro `500` em API de auth

### Verificacoes

1. Confirmar API Gateway correto por ambiente
2. Confirmar Lambda correta por ambiente
3. Confirmar alias `live`
4. Ler CloudWatch da Lambda
5. Conferir `MONGO_PARAMETER_NAME` e `JWT_PARAMETER_NAME`
6. Conferir o formato real do parametro no Systems Manager Parameter Store

### Comandos uteis

- `aws apigatewayv2 get-apis`
- `aws apigatewayv2 get-routes --api-id <id>`
- `aws apigatewayv2 get-integrations --api-id <id>`
- `aws lambda get-function-configuration --function-name <name>`
- `aws lambda get-alias --function-name <name> --name live`
- `aws logs tail /aws/lambda/<name> --since 30m --format short`
- `aws ssm get-parameter --name <name> --with-decryption`

### Causa recorrente ja observada

Secret de Mongo salvo em formato incorreto:

- `{ConnectionString:...}`
- JSON com BOM

## 2. Erro de environment Angular invalido

### Sintomas

- `TS1117`
- `TS1136`
- chave duplicada
- virgula dupla

### Verificacoes

1. Abrir o `environments.dev.ts`, `environments.hml.ts` ou `environments.prod.ts`
2. Conferir se a propriedade foi duplicada
3. Conferir se ficou `,,`
4. Revisar a esteira `.NET` que atualizou o consumer

## 3. Esteira sem jobs no GitHub Actions

### Causa recorrente

Workflow reusable chamado com input obrigatorio faltando ou com input invalido.

### Verificacoes

1. Comparar `with:` do workflow chamador
2. Comparar `inputs:` do reusable workflow
3. Confirmar se a branch do evento tem a versao atual do YAML

## 4. Falha de push ou criacao de release branch

### Sintoma

- `403`
- `Permission denied to github-actions[bot]`

### Verificacao

Garantir:

- `persist-credentials: false`
- `token: ${{ secrets.PAT_TOKEN }}`
- `git remote set-url origin` com `PAT_TOKEN` onde houver `push`

## 5. HML nao encontra arquivo `environments.hml.ts`

### Verificacao

1. Confirmar se o repositorio Angular ja recebeu o arquivo
2. Confirmar configuracao `hml` em `angular.json`
3. Confirmar se a branch `release/*` tem o arquivo versionado

## 6. Testes .NET falhando por `FactAttribute`

### Causa recorrente observada

Projeto de testes sem `global using Xunit;`

### Acao

Adicionar `GlobalUsings.cs` no projeto de testes.

## 7. Falha no build Angular por URL antiga de API em spec

### Causa recorrente

Teste hardcoded com API Gateway antigo.

### Correcao esperada

Usar `environment.apiUrl` no spec em vez de URL fixa.

## 8. Swagger responde `404` em API publicada

### Verificacoes

1. Confirmar se a API publicada tem as rotas:
   - `ANY /swagger`
   - `ANY /swagger/{proxy+}`
2. Confirmar a integration do API Gateway apontando para a Lambda `:live`
3. Testar:
   - `GET /swagger/index.html`
   - `GET /swagger/v1/swagger.json`
4. Confirmar se o Swagger funciona por invocacao direta da Lambda quando houver duvida sobre o gateway

### Causa recorrente ja observada

No caso da auth, o Terraform dedicado nao provisionava as rotas de Swagger inicialmente.

### Comandos uteis

- `aws apigatewayv2 get-routes --api-id <id>`
- `aws apigatewayv2 get-integrations --api-id <id>`
- `curl -i https://<api-id>.execute-api.us-east-1.amazonaws.com/swagger/index.html`
- `curl -i https://<api-id>.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json`
