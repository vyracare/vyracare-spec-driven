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

O repositorio usa subject OIDC imutavel do GitHub (`owner@id/repository@id`). Uma trust policy baseada somente no nome textual do repositorio nao funciona quando `use_immutable_subject` esta habilitado.

## Releases das lojas

As esteiras de assinatura e envio a Google Play e App Store Connect dependem de contas, application records, keystore Android, issuer/key da App Store Connect e provisioning definitivo. Ate esses dados existirem, a CI gera artefatos nao assinados para validar os projetos sem armazenar credenciais ficticias.

Quando habilitadas, as esteiras de loja devem ser manuais ou disparadas por tag, exigir environment protegido e nunca compartilhar os secrets usados para OTA.
