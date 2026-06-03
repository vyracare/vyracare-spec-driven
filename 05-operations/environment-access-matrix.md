# Matriz de Ambientes e Acessos

Esta matriz consolida os principais acessos, identificadores e recursos por ambiente.

## Regras desta matriz

- URLs podem mudar se recursos forem recriados.
- IDs de API Gateway podem mudar em reprovisionamento.
- nomes de secrets sao documentados, mas valores nao sao exibidos.
- o shell e o ponto principal de entrada da plataforma.

## Shell por ambiente

| Ambiente | URL principal |
| --- | --- |
| Local | `http://localhost:4200` |
| Dev | `https://dnhcnj7sdnfel.cloudfront.net` |
| HML | `https://d2ukbrzje889m2.cloudfront.net` |
| Prod | `https://d13ugmrrfi5a31.cloudfront.net` |

## APIs por ambiente

### Authentication

| Ambiente | API Gateway | API ID | Base URL | Swagger UI | Swagger JSON | Lambda | Database | Mongo Secret | JWT Secret |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Dev | `vyracare-api-authentication-dev` | `axswteu0u1` | `https://axswteu0u1.execute-api.us-east-1.amazonaws.com/api/auth` | `https://axswteu0u1.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://axswteu0u1.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-authentication-dev` | `vyracare_db_dev` | `vyracare/shared/mongo-dev` | `vyracare/shared/jwt-signing-dev` |
| HML | `vyracare-api-authentication-hml` | `jkvfvgsw4l` | `https://jkvfvgsw4l.execute-api.us-east-1.amazonaws.com/api/auth` | `https://jkvfvgsw4l.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://jkvfvgsw4l.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-authentication-hml` | `vyracare_db_hml` | `vyracare/shared/mongo-hml` | `vyracare/shared/jwt-signing-hml` |
| Prod | `vyracare-api-authentication` | `bj6riwfeni` | `https://bj6riwfeni.execute-api.us-east-1.amazonaws.com/api/auth` | `https://bj6riwfeni.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://bj6riwfeni.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-authentication` | `vyracare_db` | `vyracare/shared/mongo-prod` | `vyracare/shared/jwt-signing-prod` |

### Client

| Ambiente | API Gateway | API ID | Base URL | Swagger UI | Swagger JSON | Lambda | Database | Mongo Secret | JWT Secret |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Dev | `vyracare-api-client-dev` | `028k6c7rpb` | `https://028k6c7rpb.execute-api.us-east-1.amazonaws.com/api/client` | `https://028k6c7rpb.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://028k6c7rpb.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-client-dev` | `vyracare_db_dev` | `vyracare/shared/mongo-dev` | `vyracare/shared/jwt-signing-dev` |
| HML | `vyracare-api-client-hml` | `1vd2y2p3ni` | `https://1vd2y2p3ni.execute-api.us-east-1.amazonaws.com/api/client` | `https://1vd2y2p3ni.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://1vd2y2p3ni.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-client-hml` | `vyracare_db_hml` | `vyracare/shared/mongo-hml` | `vyracare/shared/jwt-signing-hml` |
| Prod | `vyracare-api-client` | `4uh1kerr9a` | `https://4uh1kerr9a.execute-api.us-east-1.amazonaws.com/api/client` | `https://4uh1kerr9a.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://4uh1kerr9a.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-client` | `vyracare_db` | `vyracare/shared/mongo-prod` | `vyracare/shared/jwt-signing-prod` |

### Proceedings

| Ambiente | API Gateway | API ID | Base URL | Swagger UI | Swagger JSON | Lambda | Database | Mongo Secret | JWT Secret |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Dev | `vyracare-api-proceedings-dev` | `eri1s9zq97` | `https://eri1s9zq97.execute-api.us-east-1.amazonaws.com/api/proceedings` | `https://eri1s9zq97.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://eri1s9zq97.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-proceedings-dev` | `vyracare_db_dev` | `vyracare/shared/mongo-dev` | `vyracare/shared/jwt-signing-dev` |
| HML | `vyracare-api-proceedings-hml` | `lsh2vmjq8k` | `https://lsh2vmjq8k.execute-api.us-east-1.amazonaws.com/api/proceedings` | `https://lsh2vmjq8k.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://lsh2vmjq8k.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-proceedings-hml` | `vyracare_db_hml` | `vyracare/shared/mongo-hml` | `vyracare/shared/jwt-signing-hml` |
| Prod | `vyracare-api-proceedings` | `ya4lnham1d` | `https://ya4lnham1d.execute-api.us-east-1.amazonaws.com/api/proceedings` | `https://ya4lnham1d.execute-api.us-east-1.amazonaws.com/swagger/index.html` | `https://ya4lnham1d.execute-api.us-east-1.amazonaws.com/swagger/v1/swagger.json` | `vyracare-api-proceedings` | `vyracare_db` | `vyracare/shared/mongo-prod` | `vyracare/shared/jwt-signing-prod` |

## Convenções operacionais

### Branch para ambiente

| Branch | Ambiente |
| --- | --- |
| `develop` | `dev` |
| `release/*` | `hml` |
| `main` | `prod` |

### Angular environment file por ambiente

| Contexto | Arquivo |
| --- | --- |
| Local | `src/environments/environments.ts` |
| Dev | `src/environments/environments.dev.ts` |
| HML | `src/environments/environments.hml.ts` |
| Prod | `src/environments/environments.prod.ts` |

### Convenção de commit

As mensagens de commit da organização devem ser escritas em português.

Exemplos:

- `feat: adiciona matriz de ambientes e acessos`
- `fix: corrige exposicao de swagger na autenticacao`
- `docs: atualiza convencao de commits no spec-driven`
