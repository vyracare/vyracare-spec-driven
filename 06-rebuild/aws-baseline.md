# Baseline de AWS e Dados

Este documento descreve o baseline minimo de AWS, dados e secrets para reerguer o ecossistema.

## Servicos usados

- S3
- CloudFront
- Lambda
- API Gateway HTTP
- IAM
- Secrets Manager
- Cognito
- CloudWatch
- CodeArtifact

## Naming esperado

### Angular

- bucket `dev`: `<repo>-dev`
- bucket compartilhado `hml/prod`: `<repo>`
- CloudFront `dev`: aponta para bucket `-dev`
- CloudFront `hml`: aponta para bucket sem sufixo em `/blue/<timestamp>`
- CloudFront `prod`: aponta para bucket sem sufixo em `/green/<timestamp>`

### .NET

- Lambda `dev`: `<repo-normalizado>-dev`
- Lambda `hml`: `<repo-normalizado>-hml`
- Lambda `prod`: `<repo-normalizado>`
- API Gateway `dev`: `<repo-normalizado>-dev`
- API Gateway `hml`: `<repo-normalizado>-hml`
- API Gateway `prod`: `<repo-normalizado>`

## Dados

### MongoDB Atlas

Databases esperadas:

- `vyracare_db_dev`
- `vyracare_db_hml`
- `vyracare_db`

Separacao atual:

- ambientes compartilham cluster
- o isolamento principal esta no nome do banco

## Secrets Manager

Secrets esperados:

### Mongo

- `vyracare/shared/mongo-dev`
- `vyracare/shared/mongo-hml`
- `vyracare/shared/mongo-prod`

Formato:

```json
{"ConnectionString":"mongodb+srv://..."}
```

### JWT

- `vyracare/shared/jwt-signing-dev`
- `vyracare/shared/jwt-signing-hml`
- `vyracare/shared/jwt-signing-prod`

Formato:

```json
{"Key":"..."}
```

## Variaveis de ambiente de Lambda

As Lambdas devem receber:

- `Mongo__Database`
- `MONGO_SECRET_NAME`
- `JWT_SECRET_NAME`

## Swagger

Toda API publicada deve expor no gateway:

- `ANY /swagger`
- `ANY /swagger/{proxy+}`

Sem isso, a aplicacao pode ter Swagger habilitado mas continuar inacessivel externamente.

## Cognito

O baseline de autenticacao depende de Cognito para a API de auth.

Ao reconstruir a plataforma, nao trate `vyracare-api-authentication` como API generica pura. Ela tem dependencias adicionais de runtime e provisionamento.

## CodeArtifact

O design system depende de CodeArtifact para distribuicao do pacote `@vyracare/design-system`.

Sem esse repositorio operacional:

- shell e MFEs nao restauram dependencias corretamente
- pipelines Angular podem falhar antes do build
