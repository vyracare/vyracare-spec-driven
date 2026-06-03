# Arquitetura Backend

## Modelo adotado

As APIs .NET seguem estes principios:

- separacao por dominio
- deploy serverless em AWS Lambda
- exposicao por API Gateway HTTP
- persistencia em MongoDB
- configuracao via appsettings + environment variables + Secrets Manager

## Topologia atual

### APIs

- `vyracare-api-authentication`
- `vyracare-api-client`
- `vyracare-api-proceedings`

### Runtime

- `dotnet8`
- Lambda
- API Gateway HTTP API

### Persistencia

- MongoDB Atlas
- database segregado por ambiente

## Estrutura interna de codigo

O backend foi reorganizado para um modelo mais proximo de:

- vertical slice
- ports/adapters pragmatico

Na pratica isso significa:

- features organizadas por caso de uso
- `Common` para configuracao, resultados e utilitarios compartilhados
- `Infrastructure` para persistence, secrets, DI e seguranca
- testes unitarios em projeto separado

## Configuracao

As APIs usam:

- `appsettings.json` com configuracao nao sensivel
- environment variables para override por ambiente
- AWS Secrets Manager para valores sensiveis

## Documentacao de API

As APIs publicadas expõem Swagger UI e `swagger.json` por ambiente.

Padrao adotado:

- `https://<api-id>.execute-api.us-east-1.amazonaws.com/swagger/index.html`
- `https://<api-id>.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json`

No caso da auth, o suporte a Swagger depende de rotas explicitas no Terraform dedicado, porque essa API nao usa o modulo generico de rotas.

## Recursos por ambiente

### Dev

- Lambda com sufixo `-dev`
- API Gateway com sufixo `-dev`
- database `vyracare_db_dev`
- secrets `mongo-dev` e `jwt-signing-dev`

### HML

- Lambda com sufixo `-hml`
- API Gateway com sufixo `-hml`
- database `vyracare_db_hml`
- secrets `mongo-hml` e `jwt-signing-hml`

### Prod

- Lambda sem sufixo adicional
- API Gateway sem sufixo adicional
- database `vyracare_db`
- secrets `mongo-prod` e `jwt-signing-prod`

## Auth como caso especial

`vyracare-api-authentication` possui diferencias relevantes:

- Cognito e JWT sao parte do bootstrap
- integracao com `vyracare-app-shell`
- integracao explicita com `vyracare-app-user-mfe`
- workflow dedicado `cd-auth-dot-net.yml`

## Ponto de atencao

As pipes e o bootstrap esperam que secrets JSON estejam gravados em formato valido e sem BOM, por exemplo:

```json
{"ConnectionString":"mongodb+srv://..."}
```

Se o secret for salvo como string mal serializada ou com BOM, a aplicacao pode tentar usar o JSON inteiro como connection string e responder `500`.
