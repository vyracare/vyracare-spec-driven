# Esteiras Mobile

## Repositorio base

As reusable workflows vivem em `vyracare-infra-pipes-mobile`.

## CI nativa

`mobile-ci.yml` executa:

- Node.js 22 e `npm ci`;
- testes do contrato de update;
- validacao da configuracao Capacitor e da chave publica;
- composicao do fallback local;
- `cap sync` para Android e iOS;
- APK debug com Java 21 em Linux;
- build de simulador iOS sem code signing em runner macOS.

O workflow `vyracare-app-mobile/.github/workflows/ci.yml` consome essa esteira em pushes para `main` e pull requests.

## Publicacao de live update

`live-update-publish.yml` recebe um artefato web previamente testado e:

1. valida canal e URL HTTPS;
2. confirma `index.html` na raiz;
3. cria o ZIP;
4. calcula SHA-256;
5. assina o arquivo com RSA/SHA-256 usando a chave privada do GitHub;
6. cria o manifesto com compatibilidade nativa;
7. publica primeiro o ZIP imutavel;
8. publica por ultimo o manifesto do canal;
9. invalida apenas o manifesto no CloudFront.

A ordem ZIP -> manifesto impede o app de observar um release ainda incompleto.

## Integracao com a esteira Angular

`vyracare-infra-pipes-angular/.github/workflows/cd-angular.yml` possui os inputs opcionais:

- `publish-mobile-update`;
- `mobile-repository`, com padrao `vyracare/vyracare-app-mobile`.

Quando habilitado no shell, depois do deploy web bem-sucedido o workflow:

- publica `mobile-shell-dist` por sete dias;
- converte o ambiente no canal mobile correspondente;
- envia `repository_dispatch` com repositorio de origem, run ID, canal e bundle ID.

O workflow `publish-web-update.yml` do app mobile baixa esse artefato, localiza o `index.html`, injeta o runtime mobile e chama a reusable workflow de publicacao.

O gancho permanece `false` por padrao na esteira reutilizavel. O shell o habilita explicitamente por ambiente; `develop` e o primeiro ambiente ativado, enquanto `hml` e `production` continuam opt-in apos validacao e aprovacao.

## Configuracao exigida para ativacao

No `vyracare-app-mobile`:

### Secrets

- `OTA_SIGNING_PRIVATE_KEY`: ja criado; chave RSA privada de assinatura;
- `PAT_TOKEN`: token ja adotado pelas automacoes cross-repo, com leitura de Actions no repositorio do shell;
- `AWS_ROLE_ARN`: role assumida por OIDC para publicar no bucket e invalidar o CloudFront.

### Variables

- `MOBILE_UPDATES_BUCKET`;
- `MOBILE_UPDATES_CDN_URL`;
- `MOBILE_UPDATES_CLOUDFRONT_ID`;
- `AWS_REGION`;
- `MOBILE_ANDROID_MIN_BUILD` e `MOBILE_ANDROID_MAX_BUILD`;
- `MOBILE_IOS_MIN_BUILD` e `MOBILE_IOS_MAX_BUILD`.

Depois de validar `dev`, o shell pode definir `publish-mobile-update: true` nos chamadores de CD, promovendo separadamente para `hml` e `production`.

## Validacao da infraestrutura

O workflow manual `live-update-smoke.yml` cria um bundle sintetico marcado como compativel apenas com o build `999999`. Assim ele testa assinatura, OIDC, S3, manifesto e invalidacao sem poder ser ativado por um aplicativo distribuivel.

Em 9 de outubro de 2026, o run `37948195870` concluiu com sucesso. Tambem foram confirmados externamente:

- download do manifesto e do ZIP pela URL CloudFront;
- igualdade entre o SHA-256 baixado e o checksum do manifesto;
- validade da assinatura RSA com a chave publica embarcada;
- plano Terraform sem divergencias apos o provisionamento.

Na mesma data, o PR `vyracare-app-shell#103` foi integrado em `develop`. O deploy do shell `37950056115` publicou o artefato e disparou automaticamente o workflow mobile `37950229669`. O canal `dev` passou a apontar para o bundle real `8cb8f16c4cee-37950056115`, compativel com os builds Android/iOS `1`; o ZIP servido pela CDN teve checksum e assinatura RSA validados novamente.

O repositorio usa subject OIDC imutavel do GitHub (`owner@id/repository@id`). Uma trust policy baseada somente no nome textual do repositorio nao funciona quando `use_immutable_subject` esta habilitado.

## Releases das lojas

O Android usa o package name `br.com.vyracare.app`, `compileSdk 36` e `targetSdk 36`. O workflow `Build Android Internal Release` recebe o run ID de um deploy testado do shell, incorpora o canal solicitado e chama `android-store-build.yml` para gerar um AAB assinado.

A chave de upload Android e suas senhas ficam exclusivamente nos secrets `ANDROID_UPLOAD_*`. O certificado publico versionado possui SHA-256 `68:34:0A:61:01:0C:64:BC:35:D0:5D:FD:B0:F1:0E:D4:F7:12:F7:A7:76:93:B3:58:E0:B1:02:87:72:47:0B:97`. O Google Play deve gerenciar a app signing key; a chave do VyraCare e somente a upload key.

O primeiro AAB, `vyracare-1.0.0-1.aab`, foi produzido no run `37951987234`, com SHA-256 `11683656bec7d2a2e34eaf9a8c984a1d9f63d74e21e95db4029c0f2c28072487`. O artefato incorpora o shell do run `37950056115` e o canal `dev`.

O envio inicial permanece manual porque o registro do aplicativo, a adesao ao Play App Signing e as declaracoes obrigatorias precisam existir no Play Console. Depois da primeira release interna, a API do Google Play pode ser habilitada para automatizar novos uploads com uma service account de menor privilegio.

As esteiras de loja devem ser manuais ou disparadas por tag, exigir environment protegido e nunca compartilhar os secrets usados para OTA.
