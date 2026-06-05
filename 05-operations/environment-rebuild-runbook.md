# Runbook de Recriacao por Ambiente

Este runbook descreve o caminho operacional para recriar `dev`, `hml` ou `prod` sem depender de memoria tribal.

## Objetivo

Garantir que um ambiente possa ser refeito preservando:

- naming esperado
- branch flow
- URLs publicadas
- Lambda, Gateway e CloudFront coerentes
- banco e parametros corretos por ambiente

## Regra base

Antes de recriar um ambiente, confirme:

1. qual ambiente sera afetado
2. quais repositorios dependem dele
3. se o problema e de frontend, backend, dados ou IAM
4. se a recriacao e total ou parcial

## Recriacao de dev

### Escopo esperado

- shell em `develop`
- MFEs em `develop`
- APIs em `develop`
- banco `vyracare_db_dev`
- parametros `*-dev`

### Ordem recomendada

1. confirmar `vyracare_db_dev`
2. confirmar `vyracare/shared/mongo-dev`
3. confirmar `vyracare/shared/jwt-signing-dev`
4. republicar `vyracare-design-system` se houver impacto em pacote
5. publicar `vyracare-app-shell` em `develop`
6. publicar MFEs em `develop`
7. publicar APIs em `develop`
8. validar shell, auth, MFEs e Swagger

### Itens especificos de dev

- buckets `-dev` usam pastas versionadas na raiz
- nao usam `blue/green`
- Angular em `develop` usa `environment.dev.ts`
- .NET em `develop` atualiza consumers em `environment.dev.ts`

## Recriacao de hml

### Escopo esperado

- shell em `release/*`
- MFEs em `release/*`
- APIs em `release/*`
- banco `vyracare_db_hml`
- parametros `*-hml`

### Ordem recomendada

1. confirmar `vyracare_db_hml`
2. confirmar `vyracare/shared/mongo-hml`
3. confirmar `vyracare/shared/jwt-signing-hml`
4. criar ou reaproveitar `release/<versionamento>`
5. publicar shell em `release/*`
6. publicar MFEs em `release/*`
7. publicar APIs em `release/*`
8. validar shell, auth, MFEs e Swagger

### Itens especificos de hml

- Angular em `release/*` usa `environment.hml.ts`
- CloudFront `hml` aponta para `/blue/<timestamp>`
- .NET em `release/*` atualiza consumers em `environment.hml.ts`

## Recriacao de prod

### Escopo esperado

- shell em `main`
- MFEs em `main`
- APIs em `main`
- banco `vyracare_db`
- parametros `*-prod`

### Ordem recomendada

1. confirmar `vyracare_db`
2. confirmar `vyracare/shared/mongo-prod`
3. confirmar `vyracare/shared/jwt-signing-prod`
4. garantir que a release aprovada chegou em `main`
5. publicar shell em `main`
6. publicar MFEs em `main`
7. publicar APIs em `main`
8. validar shell, auth, MFEs e Swagger

### Itens especificos de prod

- Angular em `main` usa `environment.prod.ts`
- CloudFront `prod` aponta para `/green/<timestamp>`
- .NET em `main` atualiza consumers em `environment.prod.ts`

## Recriacao parcial por camada

### Apenas frontend

Use quando o problema estiver em:

- CloudFront
- bucket
- `remoteEntry.js`
- shell apontando para URL errada

Sequencia:

1. shell
2. MFEs afetados
3. validacao do `remoteEntry.js`

### Apenas backend

Use quando o problema estiver em:

- Lambda
- API Gateway
- Swagger
- parametros
- banco

Sequencia:

1. parametros
2. banco
3. Lambdas
4. APIs
5. Swagger

## Validacao minima apos recriacao

### Frontend

- shell abre
- rotas principais carregam
- `remoteEntry.js` responde `200`

### Backend

- endpoint principal responde
- Swagger UI responde
- Swagger JSON responde
- logs da Lambda sem falha de bootstrap

### Dados

- auth cadastra e autentica
- APIs leem banco correto
- ambiente nao aponta para parametro errado

## Erros recorrentes a vigiar

- parametro com JSON invalido
- parametro com BOM
- integration do API Gateway com `PayloadFormatVersion` errado
- rota de Swagger ausente
- consumer Angular atualizado no arquivo de environment errado
- release criada sem herdar correcoes recentes de `develop`

## Comandos uteis

- `aws apigatewayv2 get-apis`
- `aws apigatewayv2 get-routes --api-id <id>`
- `aws lambda get-function-configuration --function-name <name>`
- `aws lambda get-alias --function-name <name> --name live`
- `aws logs tail /aws/lambda/<name> --since 30m --format short`
- `aws ssm get-parameter --name <name> --with-decryption`
- `curl -i https://<api-id>.execute-api.us-east-1.amazonaws.com/swagger/index.html`
