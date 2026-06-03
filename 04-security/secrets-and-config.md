# Secrets e Configuracao

## Principio

Repositorio publico pode conter estrutura de configuracao. Nao pode conter segredo operacional real.

## Backend

### Configuracao versionada

`appsettings.json` deve conter:

- valores nao sensiveis
- nomes de secret
- configuracao estrutural

### Segredos reais

Segredos ficam em:

- AWS Secrets Manager
- variaveis de ambiente da Lambda apontando para o nome do secret

## Secrets padrao atuais

### Mongo

- `vyracare/shared/mongo-dev`
- `vyracare/shared/mongo-hml`
- `vyracare/shared/mongo-prod`

Formato esperado:

```json
{"ConnectionString":"mongodb+srv://..."}
```

### JWT

- `vyracare/shared/jwt-signing-dev`
- `vyracare/shared/jwt-signing-hml`
- `vyracare/shared/jwt-signing-prod`

Formato esperado:

```json
{"Key":"..."}
```

## Regras importantes

- JSON valido
- sem BOM
- sem chaves sem aspas
- sem multiline desnecessario

## Fall back ainda suportado

Os projetos mantem fallback para:

- `MONGO_URI`
- `JWT_KEY`

Mas o caminho preferencial atual e:

- `MONGO_SECRET_NAME`
- `JWT_SECRET_NAME`

## Frontend

Os arquivos de `environment` nao devem conter credenciais sensiveis.

Eles podem conter:

- URLs publicas de API
- `remoteEntry.js`
- identificadores publicos necessarios ao runtime
