# Runbook de Release Mobile

## Primeiro provisionamento

1. Criar bucket privado exclusivo para live updates.
2. Criar CloudFront com Origin Access Control e TLS.
3. Configurar CORS apenas se necessario para o mecanismo de download do dispositivo.
4. Criar role OIDC limitada a `s3:PutObject` nos prefixos `releases/` e `channels/`, e a invalidacao da distribuicao definida.
5. Preencher variables e secrets descritos em `03-delivery/mobile-pipelines.md`.
6. Proteger os environments `hml` e `production` com aprovadores.
7. Executar manualmente um release `dev` e testar em aparelho fisico.
8. Somente entao habilitar `publish-mobile-update: true` no CD do shell.

## Validacao de um update

- conferir sucesso do CI Angular e do deploy web;
- conferir sucesso da composicao mobile;
- baixar o manifesto e validar canal, bundle ID, checksum e faixa nativa;
- confirmar que a URL do ZIP e HTTPS e responde `200`;
- abrir o app com o bundle anterior, fechar e abrir novamente;
- confirmar o novo bundle e os fluxos de login, tenant, dashboard e navegacao inferior;
- testar sem rede para confirmar que o bundle local continua abrindo;
- observar rollback automatico em um bundle de teste que nao confirma `ready()`.

## Promocao

1. Publicar e validar em `dev`.
2. Promover o mesmo codigo para `release/*` e validar em `hml`.
3. Aprovar o environment de producao.
4. Publicar em `main`.
5. Monitorar autenticacao, erros JavaScript, falhas de download e taxa de rollback.

Nao se promove copiando o manifesto de outro canal manualmente. A esteira deve recriar o manifesto com a faixa de build correta e manter rastreabilidade do run.

## Rollback de live update

O rollback operacional consiste em publicar no manifesto do canal um bundle anterior, ainda compativel e assinado. Nunca sobrescrever o ZIP imutavel.

Se o bundle nem chega a inicializar, o `readyTimeout` restaura automaticamente o bundle incorporado e `autoBlockRolledBackBundles` impede nova tentativa daquele identificador.

Se o problema exige plugin, permissao ou codigo nativo diferente, interromper live updates e preparar release nas lojas.

## Incidente de chave

Se a chave privada OTA for exposta:

1. desabilitar a publicacao e restringir o bucket/canal;
2. preservar logs e identificar bundles publicados;
3. gerar novo par de chaves;
4. publicar um novo binario nas lojas com a nova chave publica;
5. nao reativar OTA para builds antigos, pois eles continuam confiando na chave comprometida;
6. revogar/remover o secret antigo depois da preservacao das evidencias necessarias.

## Release nativa

Nova release de loja e obrigatoria quando mudar plugin, permissao, entitlement, Gradle, Xcode ou contrato que o binario atual nao suporta. Antes do envio:

- incrementar `versionCode` Android e `CFBundleVersion` iOS;
- atualizar a faixa de compatibilidade de OTA;
- conferir privacy manifest, declaracoes de dados e exclusao de conta;
- validar Android com o target API vigente;
- testar upgrade sobre a versao publicada, nao apenas instalacao limpa;
- confirmar que o bundle incorporado funciona sem depender do canal remoto.
