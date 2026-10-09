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

Para um smoke test sem risco de instalacao, executar `Live Update Infrastructure Smoke Test` no `vyracare-app-mobile`. O manifesto produzido usa build `999999`; ele valida a infraestrutura, mas deve ser substituido por um bundle real do shell antes de testar o aplicativo.

Depois do smoke test, a ativacao de `develop` deve passar pelo pull request protegido do shell. Nao contornar a exigencia de revisao da branch apenas para iniciar o deploy.

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

## Primeiro teste interno no Google Play

1. Criar o aplicativo `VyraCare` no Play Console com package name `br.com.vyracare.app`.
2. Habilitar Play App Signing, mantendo o Google como custodiante da app signing key.
3. Preencher acesso ao app, anuncios, classificacao indicativa, publico-alvo, seguranca de dados e politica de privacidade.
4. Baixar o artefato `vyracare-android-1.0.0-1` do run `37951987234` e conferir o arquivo `SHA256SUMS.txt`.
5. Criar uma release em `Testing > Internal testing` e enviar `vyracare-1.0.0-1.aab`.
6. Adicionar os e-mails dos testadores e compartilhar o opt-in link fornecido pelo Play Console.
7. Instalar exclusivamente pelo link de teste e validar login, tenant, dashboard, navegacao, modo offline e atualizacao assinada do canal `dev`.

Cada novo upload precisa usar `versionCode` maior que o anterior. A primeira release automatizada deve usar `2`, mesmo quando o `versionName` continuar na familia `1.0.x`.
