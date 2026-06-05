# Esteiras .NET

## Repositorio base

As esteiras reutilizaveis .NET vivem em:

- `vyracare-infra-pipes-dot-net`

Workflows principais:

- `ci-auth-dot-net.yml`
- `ci-generic-dot-net.yml`
- `cd-auth-dot-net.yml`
- `cd-generic-dot-net.yml`

## Especializacao

### Auth
`vyracare-api-authentication` usa esteira dedicada porque:

- provisiona Cognito
- tem rotas explicitas de auth
- atualiza shell e `app-user-mfe`

### Generic
`vyracare-api-client` e `vyracare-api-proceedings` usam a esteira generica.

## Fluxo por repositorio .NET

Cada API agora segue o mesmo desenho logico do Angular:

- `pr-to-develop.yml`
- `pr-to-release.yml`
- `pr-to-main.yml`
- `publish.yml`

## Fluxo detalhado

### PR to develop

Mantem o gate de qualidade antes da promocao para `develop`.

### Push em `develop`

Workflow:

- publica em `dev`
- cria ou reaproveita `release/<versionamento>`
- abre PR de `develop` para `release/<versionamento>`

### Push em `release/*`

Workflow:

- detecta se a release esta a frente da `main`
- roda CI novamente
- publica em `hml`
- abre PR para `main`

### Push em `main`

Workflow:

- publica em `prod`

## CI .NET

Os workflows de CI foram segregados em dois jobs:

### `test-dotnet`

- checkout
- setup .NET
- restore do projeto de testes
- `dotnet test`

### `build-dotnet`

- depende de `test-dotnet`
- restore do projeto principal
- `dotnet build`
- `dotnet publish`
- zip da lambda

Resultado:

- se o teste falhar, o build nao inicia

## CD .NET

Os workflows de CD fazem:

- checkout de infra
- checkout do backend
- setup .NET
- restore/build/publish para `linux-x64`
- resolucao de contexto por branch
- export de variaveis Terraform
- `terraform init/plan/apply`
- import de recursos existentes quando necessario
- export de `API_GATEWAY_URL`
- sincronizacao opcional com MFE consumidor

## Branch -> recursos

### `develop`

- suffix `-dev`
- database `vyracare_db_dev`
- parametros `/vyracare/shared/mongo-dev` e `/vyracare/shared/jwt-signing-dev`

### `release/*`

- suffix `-hml`
- database `vyracare_db_hml`
- parametros `/vyracare/shared/mongo-hml` e `/vyracare/shared/jwt-signing-hml`

### `main`

- sem suffix
- database `vyracare_db`
- parametros `/vyracare/shared/mongo-prod` e `/vyracare/shared/jwt-signing-prod`

## Atualizacao de frontends consumidores

Quando uma API publica, a esteira pode:

- atualizar `apiUrl` em MFEs consumidores
- atualizar `authApiUrl` no `vyracare-app-user-mfe`
- escolher o arquivo correto conforme a branch:
  - `environments.dev.ts` para `develop`
  - `environments.ts` apenas como fallback para repositorios que ainda nao receberam o arquivo `dev`
  - `environments.hml.ts`
  - `environments.prod.ts`

## Ponto sensivel

Os parametros precisam estar serializados corretamente em JSON e sem BOM.

Problemas ja observados:

- parametro salvo como `{ConnectionString:...}`
- parametro salvo com BOM no inicio

Ambos levam a `500` em runtime se o bootstrap tentar usar o texto bruto como connection string.
