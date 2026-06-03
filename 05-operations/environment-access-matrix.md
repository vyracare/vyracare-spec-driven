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

## Design system publicado

| Recurso | URL |
| --- | --- |
| Playground publicado | `https://dvgk2zjit84dn.cloudfront.net` |
| Storybook publicado | `https://d2pws2zjq41jau.cloudfront.net` |

## Remote entries do shell

### Dev

| MFE | Remote Entry |
| --- | --- |
| Dashboard | `https://d2injcyipyvtvj.cloudfront.net/remoteEntry.js` |
| User | `https://d1qmra1m75dpb9.cloudfront.net/remoteEntry.js` |
| Profile | `https://dof426tafo2g9.cloudfront.net/remoteEntry.js` |
| Proceedings | `https://d2k3jf7r93hruy.cloudfront.net/remoteEntry.js` |

### HML

| MFE | Remote Entry |
| --- | --- |
| Dashboard | `https://d3dtvolcohpeqn.cloudfront.net/remoteEntry.js` |
| User | `https://d2r6ze9ogurvf9.cloudfront.net/remoteEntry.js` |
| Profile | `https://d3ko4u9he32ku6.cloudfront.net/remoteEntry.js` |
| Proceedings | `https://d2k3jf7r93hruy.cloudfront.net/remoteEntry.js` |

### Prod

| MFE | Remote Entry |
| --- | --- |
| Dashboard | `https://d3dtvolcohpeqn.cloudfront.net/remoteEntry.js` |
| User | `https://d2r6ze9ogurvf9.cloudfront.net/remoteEntry.js` |
| Profile | `https://d3ko4u9he32ku6.cloudfront.net/remoteEntry.js` |
| Proceedings | `https://d358sdfw6lk7th.cloudfront.net/remoteEntry.js` |

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

## Convencoes operacionais

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

### Convencao de commit

As mensagens de commit da organizacao devem ser escritas em portugues.

Exemplos:

- `feat: adiciona matriz de ambientes e acessos`
- `fix: corrige exposicao de swagger na autenticacao`
- `docs: atualiza convencao de commits no spec-driven`
