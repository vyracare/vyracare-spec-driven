# Arquitetura do Aplicativo Mobile

## Decisao

O VyraCare usa um unico container Capacitor para Android e iOS no repositorio `vyracare-app-mobile`. O aplicativo embarca uma versao funcional do shell Angular e pode receber atualizacoes assinadas da camada web.

Esta abordagem atende ao objetivo de publicar a mesma solucao Angular na web e no mobile sem duplicar os MFEs. Ela nao usa `server.url` em producao e nao reduz o aplicativo a um navegador apontado permanentemente para o site.

## Componentes

| Componente | Responsabilidade |
| --- | --- |
| `vyracare-app-shell` | host Angular, navegacao e composicao dos MFEs |
| MFEs Angular | capacidades de negocio compartilhadas entre web e mobile |
| `vyracare-app-mobile` | projetos Android/iOS, bridge nativa, bundle local e runtime de atualizacao |
| `vyracare-infra-pipes-mobile` | validacao nativa e publicacao de live updates |
| S3 + CloudFront de updates | bundles imutaveis e manifesto atual de cada canal |

## Identidade visual nativa

O arquivo-mestre da marca fica em `vyracare-app-mobile/resources/logo.png`, com
fundo transparente e area de seguranca para as mascaras dos launchers. Android
usa icones legacy, round e adaptive nas densidades oficiais; iOS usa o AppIcon
opaco de `1024x1024`. As telas de abertura clara e escura reutilizam o mesmo
simbolo sobre fundos neutros, sem depender do bundle remoto.

Qualquer alteracao futura do simbolo deve regenerar conjuntamente os icones e
splashes de Android e iOS para impedir divergencia entre plataformas. O favicon
do bootstrap local e dos frontends deve derivar do mesmo arquivo-mestre.

## Infraestrutura provisionada

| Recurso | Identificador |
| --- | --- |
| Conta/regiao AWS | `510253726006` / `us-east-1` |
| Bucket de updates | `vyracare-mobile-updates-510253726006` |
| Distribuicao CloudFront | `E3OIPWV9FLA5GO` |
| URL publica | `https://d2jn6zv1s8t2bh.cloudfront.net` |
| Role de publicacao | `vyracare-mobile-updates-github-actions` |
| Estado Terraform | `s3://vyracare-terraform-state-510253726006/mobile/live-updates/terraform.tfstate` |

O bucket permanece privado e so pode ser lido pela distribuicao via Origin Access Control. A role aceita OIDC apenas do subject imutavel do repositorio mobile e dos environments `dev`, `hml` e `production`; nenhuma access key permanente e usada nessa publicacao.

## Fluxo de execucao

1. O binario abre o bundle web local incorporado.
2. O runtime chama `LiveUpdate.ready()` imediatamente. Isso confirma que o bundle atual inicializou dentro do limite de 10 segundos.
3. O app consulta por HTTPS o manifesto do canal.
4. O manifesto so e aceito quando o canal e a faixa do build nativo sao compativeis.
5. O plugin baixa o ZIP e verifica sua assinatura RSA. O SHA-256 tambem acompanha o contrato para integridade e auditoria.
6. O bundle passa a ser o proximo bundle, sem recarregamento forcado da WebView.
7. A atualizacao e ativada na proxima abertura. Se ela nao confirmar prontidao, o plugin volta ao bundle incorporado e bloqueia o bundle defeituoso.

## Canais

| Branch Angular | Ambiente web | Canal mobile |
| --- | --- | --- |
| `develop` | `dev` | `dev` |
| `release/*` | `hml` | `hml` |
| `main` | `prod` | `production` |

Cada canal aponta para um manifesto pequeno e mutavel. Os ZIPs ficam em `releases/<bundle-id>.zip`, sao imutaveis e recebem cache longo.

## Contrato do manifesto

O schema versionado vive em `vyracare-app-mobile/config/release-manifest.schema.json` e exige:

- `schemaVersion` igual a `1`;
- canal `dev`, `hml` ou `production`;
- `bundleId` imutavel;
- data de publicacao;
- URL HTTPS do ZIP;
- checksum SHA-256;
- assinatura RSA em Base64;
- faixa minima e maxima de build Android e iOS.

## Fronteira entre update web e release nativa

Pode entrar por live update:

- HTML, CSS, JavaScript, fontes e imagens;
- correcoes e funcionalidades que usam apenas capacidades ja existentes no binario;
- mudancas de shell e MFEs compativeis com os plugins embarcados.

Exige nova versao nas lojas:

- inclusao, remocao ou atualizacao de plugin Capacitor;
- novas permissoes Android/iOS;
- alteracoes em Gradle, Xcode, entitlements, manifestos nativos ou identificadores;
- qualquer capacidade que o binario instalado ainda nao possua.

## Decisoes de seguranca

- a chave privada de OTA existe somente no secret `OTA_SIGNING_PRIVATE_KEY`;
- a chave publica fica embarcada e versionada;
- o app rejeita bundles nao assinados ou fora da compatibilidade nativa;
- o bucket nao e fonte de confianca: a assinatura continua obrigatoria mesmo sobre HTTPS;
- nenhum segredo, dado pessoal, certificado privado ou credencial pode integrar um bundle web;
- producao deve usar environment protegido e aprovacao antes de mover o manifesto do canal;
- o app sempre preserva um bundle local para abertura e recuperacao.

## Conformidade com lojas

Live update nao autoriza mudar a finalidade do aplicativo nem introduzir capacidade nativa sem revisao. O produto deve manter experiencia propria, navegacao mobile, tratamento de conectividade, politica de privacidade e exclusao de conta.

Referencias normativas e tecnicas consultadas em outubro de 2026:

- Capacitor: <https://capacitorjs.com/docs>
- Capawesome Live Update: <https://capawesome.io/docs/sdks/capacitor/live-update/>
- Apple App Review Guidelines: <https://developer.apple.com/app-store/review/guidelines/>
- Google Play WebView and Affiliate Spam: <https://support.google.com/googleplay/android-developer/answer/9899034>

## Evolucao prevista

Push notifications, biometria, camera e armazenamento seguro devem entrar como incrementos nativos separados. Quando push for priorizado, o backend de notificacoes deve registrar tokens FCM/APNs por usuario, tenant, dispositivo e ambiente, sem acoplar esse contrato aos MFEs.
