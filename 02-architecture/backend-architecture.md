# Arquitetura Backend

## Modelo adotado

As APIs .NET seguem estes principios:

- separacao por dominio
- deploy serverless em AWS Lambda
- exposicao por API Gateway HTTP
- persistencia em MongoDB
- configuracao via appsettings + environment variables + Systems Manager Parameter Store

## Topologia atual

### APIs

- `vyracare-api-authentication`
- `vyracare-api-client`
- `vyracare-api-tenancy`
- `vyracare-api-proceedings`
- `vyracare-api-appointments`
- `vyracare-api-finance`

### Runtime

- `dotnet10`
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
- `Infrastructure` para persistence, parametros, DI e seguranca
- testes unitarios em projeto separado

## Configuracao

As APIs usam:

- `appsettings.json` com configuracao nao sensivel
- environment variables para override por ambiente
- AWS Systems Manager Parameter Store para valores sensiveis

As seis APIs usam `MongoDB.Driver` `3.12.0` ou superior dentro da mesma major validada. A atualizacao coordenada remove dependencias transitivas vulneraveis presentes na linha `2.24.0`. Antes de promover uma alteracao de pacote, cada API deve executar seus testes e `dotnet list package --vulnerable --include-transitive`; vulnerabilidades altas conhecidas bloqueiam a entrega.

## Fundacao multi-tenant

- `vyracare-api-tenancy` e o plano de controle de empresas, memberships, convites e trial;
- a autenticacao provisiona o primeiro tenant por uma porta interna e emite as claims de contexto;
- `vyracare-api-client` e o piloto do plano de dados e rejeita JWT autenticado sem contexto de tenant;
- procedimentos, agenda e financeiro ainda aguardam migracao e nao devem ser tratados como isolados por tenant;
- o contrato detalhado esta em [Fundacao multi-tenant](./multi-tenant-foundation.md).

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
- parametros `/vyracare/shared/mongo-dev` e `/vyracare/shared/jwt-signing-dev`

### HML

- Lambda com sufixo `-hml`
- API Gateway com sufixo `-hml`
- database `vyracare_db_hml`
- parametros `/vyracare/shared/mongo-hml` e `/vyracare/shared/jwt-signing-hml`

### Prod

- Lambda sem sufixo adicional
- API Gateway sem sufixo adicional
- database `vyracare_db`
- parametros `/vyracare/shared/mongo-prod` e `/vyracare/shared/jwt-signing-prod`

## Auth como caso especial

`vyracare-api-authentication` possui diferencias relevantes:

- Cognito e JWT sao parte do bootstrap
- integracao com `vyracare-app-shell`
- integracao explicita com `vyracare-app-user-mfe`
- workflow dedicado `cd-auth-dot-net.yml`

## APIs de indicadores do dashboard

- `vyracare-api-appointments` agrega agenda diaria, confirmacoes recentes, retornos pendentes e ocupacao semanal;
- `vyracare-api-finance` agrega receitas e despesas confirmadas do mes, variacao contra o mes anterior e boletos pendentes;
- ambas persistem datas em UTC e calculam janelas no fuso `America/Sao_Paulo`;
- os endpoints de resumo exigem o mesmo JWT emitido pela API de autenticacao.

O contrato detalhado esta em [Indicadores do Dashboard](./dashboard-indicators.md).

## Consultas para seletores operacionais

As APIs expõem consultas autenticadas e limitadas para os autocompletes de agendamento:

- Authentication pesquisa somente funcionarios ativos por nome, e-mail ou telefone e projeta uma resposta sem dados de credencial;
- Proceedings aceita filtros opcionais `search`, `activeOnly` e `limit`; `search` compara nome e codigo sem diferenciar maiusculas de minusculas;
- os termos usados em expressoes regulares sao escapados antes de chegar ao MongoDB;
- os limites sao normalizados no backend para evitar consultas abertas pelo autocomplete.

Os projetos principais de Authentication e Proceedings excluem a arvore dos respectivos projetos de testes dos itens de conteudo do Web SDK. Essa separacao evita que artefatos aninhados de teste ou publicacao sejam copiados para o build das APIs.

## Consulta de CEP

O `vyracare-api-client` atua como fachada autenticada para a API Busca CEP dos Correios:

- rota interna: `GET /api/client/addresses/postal-code/{cep}`;
- CEP normalizado para oito digitos antes da chamada externa;
- credencial Bearer mantida exclusivamente no backend;
- `400` para formato invalido, `404` para CEP inexistente e `503` para configuracao ausente, timeout ou falha do fornecedor;
- configuracoes sensiveis fornecidas por variaveis de ambiente/Parameter Store, nunca pelo MFE.

O documento MongoDB de pacientes ignora campos desconhecidos para manter compatibilidade de leitura com registros antigos que ainda contenham `rg` ou `whatsapp`.

## Ponto de atencao

As pipes e o bootstrap esperam que os parametros JSON estejam gravados em formato valido e sem BOM, por exemplo:

```json
{"ConnectionString":"mongodb+srv://..."}
```

Se o parametro for salvo como string mal serializada ou com BOM, a aplicacao pode tentar usar o JSON inteiro como connection string e responder `500`.
