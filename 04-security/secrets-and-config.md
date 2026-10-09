# Parameter Store e Configuracao

## Principio

Repositorio publico pode conter estrutura de configuracao. Nao pode conter segredo operacional real.

## Backend

### Configuracao versionada

`appsettings.json` deve conter:

- valores nao sensiveis
- nomes de parametros
- configuracao estrutural

### Parametros reais

Valores sensiveis ficam em:

- AWS Systems Manager Parameter Store
- variaveis de ambiente da Lambda apontando para o nome do parametro

## Parametros padrao atuais

### Mongo

- `/vyracare/shared/mongo-dev`
- `/vyracare/shared/mongo-hml`
- `/vyracare/shared/mongo-prod`

Formato esperado:

```json
{"ConnectionString":"mongodb+srv://..."}
```

### JWT

- `/vyracare/shared/jwt-signing-dev`
- `/vyracare/shared/jwt-signing-hml`
- `/vyracare/shared/jwt-signing-prod`

Formato esperado:

```json
{"Key":"..."}
```

## Regras importantes

- JSON valido
- sem BOM
- sem chaves sem aspas
- sem multiline desnecessario

## Fallback ainda suportado

Os projetos mantem fallback para:

- `MONGO_URI`
- `JWT_KEY`

Mas o caminho preferencial atual e:

- `MONGO_PARAMETER_NAME`
- `JWT_PARAMETER_NAME`

## Compatibilidade de migracao

Por compatibilidade temporaria, os backends ainda aceitam:

- chaves antigas `Secrets:*` em configuracao
- `MONGO_SECRET_NAME`
- `JWT_SECRET_NAME`

O bootstrap normaliza nomes como `vyracare/shared/...` para `/vyracare/shared/...` antes de consultar o SSM.

## Frontend

Os arquivos de `environment` nao devem conter credenciais sensiveis.

Eles podem conter:

- URLs publicas de API
- `remoteEntry.js`
- identificadores publicos necessarios ao runtime

## Mobile

- a chave publica de verificacao OTA pode ser versionada e embarcada;
- a chave privada OTA fica somente em GitHub Secret e assina o ZIP dentro da esteira;
- certificados Apple, provisioning profiles, senhas e keystore Android sao secrets separados da chave OTA;
- bundles web sao artefatos publicos do cliente e nunca recebem credenciais ou dados pessoais;
- o acesso AWS da publicacao mobile deve preferir OIDC e permissao limitada aos prefixos do bucket e a distribuicao necessaria.
