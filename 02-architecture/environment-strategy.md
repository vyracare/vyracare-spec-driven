# Estrategia de Ambientes

## Branch -> Ambiente

### `develop`
Representa `dev`.

### `release/*`
Representa `hml`.

### `main`
Representa `prod`.

## Frontend

### Dev

- bucket: `<repo>-dev`
- deploy path: `/<timestamp>`
- sem `blue/green`
- CloudFront aponta para a versao ativa na raiz

### HML

- bucket: `<repo>`
- deploy path: `/blue/<timestamp>`
- CloudFront `hml` aponta para `blue`

### Prod

- bucket: `<repo>`
- deploy path: `/green/<timestamp>`
- CloudFront `prod` aponta para `green`

## Backend

### Dev

- Lambda: `<repo-normalizado>-dev`
- API Gateway: `<repo-normalizado>-dev`
- Mongo DB: `vyracare_db_dev`
- Mongo secret: `vyracare/shared/mongo-dev`
- JWT secret: `vyracare/shared/jwt-signing-dev`

### HML

- Lambda: `<repo-normalizado>-hml`
- API Gateway: `<repo-normalizado>-hml`
- Mongo DB: `vyracare_db_hml`
- Mongo secret: `vyracare/shared/mongo-hml`
- JWT secret: `vyracare/shared/jwt-signing-hml`

### Prod

- Lambda: `<repo-normalizado>`
- API Gateway: `<repo-normalizado>`
- Mongo DB: `vyracare_db`
- Mongo secret: `vyracare/shared/mongo-prod`
- JWT secret: `vyracare/shared/jwt-signing-prod`

## Arquivos de ambiente Angular

### Local
- `src/environments/environments.ts`

### Dev
- `src/environments/environments.dev.ts`

### HML
- `src/environments/environments.hml.ts`

### Prod
- `src/environments/environments.prod.ts`

## Regras de promocao

1. `develop` publica em `dev`
2. `develop` gera `release/<versionamento>`
3. `release/*` publica em `hml`
4. `release/*` abre PR para `main`
5. `main` publica em `prod`

## Convencao de versionamento de release

O formato atual gerado automaticamente e:

- `release/vYYYY.MM.DD.<run_number>`

Exemplo:

- `release/v2026.06.02.18`

## Regra de CI

Promocao entre ambientes nao deve ignorar CI:

- `develop` ja passou pelo fluxo de PR anterior
- `release/*` roda CI novamente antes de publicar `hml`
- `main` e a etapa final de publicacao em `prod`
