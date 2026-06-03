# Naming de Recursos AWS

Este documento consolida o padrao de naming usado no ecossistema Vyracare.

O objetivo e reduzir drift entre Terraform, pipelines, console e documentacao.

## Regra geral

Sempre que possivel, o nome base do recurso deve derivar do nome do repositorio de produto.

Exemplos:

- `vyracare-api-authentication`
- `vyracare-api-client`
- `vyracare-api-proceedings`
- `vyracare-app-shell`
- `vyracare-app-user-mfe`

## Sufixos por ambiente

### Backend

| Ambiente | Sufixo esperado |
| --- | --- |
| Dev | `-dev` |
| HML | `-hml` |
| Prod | sem sufixo |

### Frontend

| Ambiente | Regra |
| --- | --- |
| Dev | bucket com `-dev` |
| HML | bucket sem sufixo, path `/blue/<timestamp>` |
| Prod | bucket sem sufixo, path `/green/<timestamp>` |

## Lambdas

Padrao:

- dev: `<repo-normalizado>-dev`
- hml: `<repo-normalizado>-hml`
- prod: `<repo-normalizado>`

Exemplos:

- `vyracare-api-authentication-dev`
- `vyracare-api-authentication-hml`
- `vyracare-api-authentication`

## API Gateway HTTP

Padrao:

- dev: `<repo-normalizado>-dev`
- hml: `<repo-normalizado>-hml`
- prod: `<repo-normalizado>`

Exemplos:

- `vyracare-api-client-dev`
- `vyracare-api-client-hml`
- `vyracare-api-client`

## Buckets Angular

Padrao:

- dev: `<repo>-dev`
- hml/prod: `<repo>`

Exemplos:

- `vyracare-app-shell-dev`
- `vyracare-app-shell`

## CloudFront

O nome amigavel nao e o principal contrato operacional.

Os contratos mais importantes no frontend sao:

- dominio publicado
- bucket de origem
- `OriginPath` por ambiente

Regra operacional:

- dev: aponta para bucket `-dev` e versao ativa na raiz
- hml: aponta para bucket sem sufixo em `/blue/<timestamp>`
- prod: aponta para bucket sem sufixo em `/green/<timestamp>`

## Databases Mongo

Padrao:

- dev: `vyracare_db_dev`
- hml: `vyracare_db_hml`
- prod: `vyracare_db`

## Secrets Manager

### Mongo

- `vyracare/shared/mongo-dev`
- `vyracare/shared/mongo-hml`
- `vyracare/shared/mongo-prod`

### JWT

- `vyracare/shared/jwt-signing-dev`
- `vyracare/shared/jwt-signing-hml`
- `vyracare/shared/jwt-signing-prod`

## CloudWatch Log Groups

Padrao esperado:

- `/aws/lambda/<function-name>`

Exemplos:

- `/aws/lambda/vyracare-api-authentication-dev`
- `/aws/lambda/vyracare-api-authentication-hml`
- `/aws/lambda/vyracare-api-authentication`

## Swagger routes

Toda API publicada deve manter:

- `ANY /swagger`
- `ANY /swagger/{proxy+}`

Isso nao e naming de recurso em si, mas faz parte do contrato minimo da exposicao publica.

## Regra de ouro

Se um recurso novo nao segue o naming acima, ele precisa ser tratado como excecao documentada, nao como variacao informal.
