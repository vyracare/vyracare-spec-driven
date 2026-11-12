# Modernizacao de Runtimes e Frameworks

Auditoria realizada em 11 de novembro de 2026 para definir a evolucao dos frontends Angular e das APIs .NET sem romper Module Federation, Design System, esteiras ou AWS Lambda.

## Estado encontrado

### Frontend

- shell, MFEs e template usam Angular `20.3`;
- o Design System usa Angular CLI `20.3.13` e TypeScript `5.8`, enquanto as aplicacoes usam TypeScript `5.9`;
- as aplicacoes usam Jest `29` e `jest-preset-angular` entre `14.1` e `14.6`;
- o shell usa `@angular-architects/module-federation` `20.x`;
- o ambiente local esta executando Node `25.2.1`, uma linha impar e fora das faixas suportadas pelo Angular 22;
- existem versoes diferentes do Design System entre os MFEs, embora o shell compartilhe uma instancia singleton em runtime.

Em 11 de novembro de 2026, a versao estavel mais recente publicada no npm e Angular `22.2.6`. Angular 22 exige Node `^22.22.3`, `^24.15.0` ou `^26.0.0`, TypeScript `>=6.0 <6.1` e RxJS `^6.5.3` ou `^7.4.0`. Angular 20 permanece em LTS somente ate 28 de novembro de 2026.

Fontes oficiais:

- [Angular: versionamento e suporte](https://angular.dev/reference/releases)
- [Angular: compatibilidade de Node, TypeScript e RxJS](https://next.angular.dev/reference/versions)
- [Angular Update Guide](https://angular.dev/update)

### Backend

- todas as APIs e o template apontam para `net10.0`;
- as funcoes AWS Lambda usam o runtime gerenciado `dotnet10`;
- a maquina possui SDKs `8.0.421` e `10.0.301`, sem `global.json` comum;
- ha divergencia de dependencias entre APIs e template, inclusive MongoDB Driver `2.24` no template e `3.12` nas APIs;
- pacotes de teste e integracao AWS tambem estao em versoes diferentes.

.NET 11 foi publicado em 10 de novembro de 2026 como STS. .NET 10 continua LTS, esta na versao de runtime `10.0.15` e possui suporte ate novembro de 2028. O AWS Lambda oferece runtime gerenciado `dotnet10` e nao oferece `dotnet11`; a politica da AWS prioriza runtimes .NET LTS.

Fontes oficiais:

- [.NET: politica e ciclo de suporte](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)
- [.NET 11: downloads oficiais](https://dotnet.microsoft.com/en-us/download/dotnet/11.0)
- [AWS Lambda: runtimes suportados](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)

## Decisao vigente

### Frontend: migrar para Angular 22

A migracao e recomendada e deve ocorrer antes do encerramento do LTS do Angular 20. Ela nao sera feita como uma substituicao mecanica simultanea em todos os repositorios.

Ordem de migracao:

1. padronizar Node `24` LTS em versao compativel (`>=24.15`) nas maquinas e esteiras;
2. atualizar o template Angular e confirmar o caminho Angular 20 -> 21 -> 22 com `ng update`;
3. atualizar o Design System, seus peers, Jest e Storybook; publicar uma nova major do pacote;
4. atualizar o shell e Module Federation, mantendo o shell como fornecedor da instancia singleton;
5. atualizar um MFE piloto e executar teste integrado shell + remoto;
6. atualizar os demais MFEs em ondas pequenas;
7. atualizar templates e esteiras somente depois que o caminho estiver comprovado;
8. promover por `dev`, `hml` e `prod`, com rollback por artefato.

Cada onda deve validar build, testes, carregamento de `remoteEntry`, navegacao direta, guards, Design System singleton, mobile responsivo e bundle de producao.

Depois da compatibilidade basica, a revisao de uso do framework deve priorizar:

- zoneless real, removendo o carregamento residual de Zone.js e seus alertas nos testes;
- signals para estado local derivado, sem converter indiscriminadamente streams HTTP ou eventos complexos;
- `takeUntilDestroyed` e APIs de lifecycle do Angular para evitar subscriptions manuais;
- control flow nativo e `@defer` em areas pesadas quando houver ganho mensuravel;
- standalone components e lazy loading por rota;
- clientes HTTP tipados e tratamento consistente de loading, vazio e erro;
- preservacao das regras de componentes macro no Design System.

Recursos experimentais nao entram no baseline de producao apenas por serem novos. A adocao exige estabilidade, caso de uso e teste de regressao.

### Backend: permanecer em .NET 10 LTS

O backend nao deve migrar para `net11.0` enquanto depender do runtime gerenciado do Lambda. Usar .NET 11 exigiria runtime customizado ou imagem de container, aumentaria manutencao e nao ampliaria a janela de suporte em relacao ao .NET 10 LTS.

A modernizacao do backend deve:

1. atualizar o SDK .NET 10 e os patches ASP.NET Core para a linha `10.0.15` ou posterior compativel;
2. adicionar `global.json` para reproduzir builds locais e de CI;
3. centralizar versoes comuns com `Directory.Packages.props` depois de validar todos os repositorios;
4. atualizar dependencias por familias, separando patches de majors com breaking changes;
5. alinhar template e APIs na mesma major do MongoDB Driver;
6. atualizar pacotes AWS Lambda e AWS SDK com testes de serializacao, Parameter Store e cold start;
7. rever Swashbuckle, JWT, testes e analyzers sem misturar todas as majors em um unico incremento;
8. manter `dotnet10` no Terraform e nas funcoes ate existir suporte gerenciado e uma decisao arquitetural para uma futura LTS.

## Ondas propostas

| Onda | Escopo | Saida esperada |
|---|---|---|
| 1 | Baseline e toolchains | Node LTS fixado, SDK .NET 10 fixado, matrizes de CI atualizadas |
| 2 | Design System Angular 22 | pacote major publicado e catalogo validado |
| 3 | Shell + MFE piloto | Module Federation e navegacao integrados em Angular 22 |
| 4 | Demais MFEs e template | todos os frontends alinhados e sem Zone.js residual |
| 5 | Patches .NET 10 | APIs, template e testes em dependencias compativeis atuais |
| 6 | Consolidacao | `dev`, `hml`, mobile WebView e producao validados |

## Onda 1 implementada

Em 12 de novembro de 2026, o baseline de toolchains foi aplicado:

- Node.js `24.15.0` foi fixado em `.nvmrc` e em `engines.node` (`>=24.15.0 <25`) no shell, nos MFEs, no Design System, no app mobile e no template Angular;
- as esteiras Angular, mobile e os workflows locais que configuram Node passaram a usar `24.15.0` e `actions/setup-node@v4` onde aplicavel;
- o SDK .NET `10.0.404` foi fixado por `global.json`, com `rollForward: latestPatch`, no template e em todas as APIs;
- as esteiras .NET passaram a instalar explicitamente o SDK `10.0.404`, mantendo `net10.0` e o runtime Lambda `dotnet10`;
- o template Angular deixou de declarar diretamente `@module-federation/enhanced` e `@module-federation/runtime-core`; esses pacotes sao internos da integracao e a declaracao duplicava o Webpack, causando incompatibilidade no build;
- o lockfile do template foi reconciliado com npm `11.12.1` e validado por instalacao limpa.

Validacoes da onda:

- builds aprovados no dashboard, finance, proceedings, profile, user, shell e Design System com Node `24.15.0`;
- testes e bootstrap mobile aprovados;
- instalacao limpa do template Angular aprovada; o build do template bruto continua intencionalmente indisponivel ate a substituicao dos placeholders de geracao;
- testes aprovados nas APIs authentication, client, finance, proceedings, tenancy e appointments com SDK `10.0.404`;
- o template .NET bruto resolve o SDK correto, mas somente compila depois que seus placeholders de nome e recurso forem materializados.

O Node `25` eventualmente instalado na maquina nao define mais o baseline do produto. Desenvolvedores e CI devem selecionar a versao indicada em `.nvmrc`.

## Onda 2 implementada

Em 12 de novembro de 2026, o `vyracare-design-system` foi migrado, em sequencia, de Angular 20 para Angular 21 e Angular 22 por meio das migrations oficiais do Angular CLI.

Baseline entregue:

- Angular, Angular CLI e `ng-packagr` `22.2.6`;
- TypeScript `6.0.3`;
- Storybook `10.6.7`;
- Jest e ambiente JSDOM `30.5.2`, `jest-preset-angular` `17.0.3` e `@types/jest` `30.0.0`;
- peer dependencies publicas do pacote em Angular `^22.0.0`;
- playground e testes configurados para execucao zoneless, sem carregar Zone.js no bootstrap da aplicacao;
- control flow nativo aplicado pelas migrations oficiais;
- configuracao TypeScript atualizada para `moduleResolution: Bundler` e caminhos relativos compativeis com TypeScript 6.

A alteracao de peer dependency de Angular 20 para Angular 22 e breaking change. A major `2.0.0` foi publicada no CodeArtifact ao concluir a onda; consumidores Angular 20 permanecem na major `1.x` ate a respectiva onda de migracao.

Validacoes da onda:

- 43 suites e 153 testes aprovados, com cobertura global de statements em `97,52%`;
- build da biblioteca Angular aprovado;
- build de producao do playground aprovado;
- build estatico do Storybook aprovado;
- arvore de dependencias diretas valida por `npm ls`;
- auditoria das dependencias de producao sem vulnerabilidades conhecidas.

Excecoes temporarias e divida controlada:

- `@angular-devkit/build-angular` e `zone.js` continuam como dependencias de desenvolvimento porque o builder Angular do Storybook ainda os declara como peers; eles nao sao carregados pelo playground nem pela configuracao zoneless dos testes;
- `@angular/platform-browser-dynamic` permanece somente para satisfazer o peer do Storybook;
- a esteira nao pode instalar com `--omit=optional`: Angular 22 e `ng-packagr` dependem dos bindings nativos opcionais do Rolldown para a plataforma do runner;
- o `@storybook/test-runner` legado permanece enquanto os testes de stories forem migrados para a integracao Playwright/Vitest suportada pelo Storybook; `@storybook/testing-library`, que estava sem uso, foi removido;
- os alertas de tamanho de bundle do catalogo Storybook e de arquivos de teste incluidos em sua compilacao sao internos ao catalogo e nao afetam o bundle publicado da biblioteca.

## Onda 3 implementada

Em 12 de novembro de 2026, o `vyracare-app-shell` e o `vyracare-app-profile-mfe` foram migrados, em sequencia, de Angular 20 para Angular 21 e Angular 22 pelas migrations oficiais do Angular CLI. O profile foi escolhido como MFE piloto por exercitar formularios, rotas protegidas, consumo do Design System e carregamento remoto.

Baseline entregue nos dois consumidores:

- Angular e Angular CLI `22.2.6`, Angular Material/CDK `22.2.7` no shell e TypeScript `6.0.3`;
- `@angular-architects/module-federation` `22.0.0` e `@angular-builders/custom-webpack` `22.0.1`;
- `@vyracare/design-system` `2.0.0`, compartilhado pelo shell como singleton;
- Jest e JSDOM `30.5.2` com `jest-preset-angular` `17.0.3`;
- execucao zoneless, sem Zone.js na aplicacao ou nos testes;
- control flow nativo e configuracoes de compilador aplicadas pelas migrations oficiais;
- remocao de dependencias redundantes de Module Federation, Karma, Jasmine, `ngx-build-plus` e `platform-browser-dynamic` sem uso;
- reload da pagina encapsulado por token injetavel, permitindo teste em JSDOM 30 e mantendo seguranca para SSR;
- testes de polling convertidos de `fakeAsync` baseado em Zone.js para timers controlados pelo Jest.

Validacoes da onda:

- instalacao limpa por `npm ci --legacy-peer-deps` aprovada nos dois repositorios;
- build de desenvolvimento e build de producao aprovados no shell e no profile;
- 16 suites e 113 testes aprovados no shell, com `97,44%` de statements e `90,96%` de branches;
- 6 suites e 30 testes aprovados no profile, com `99,49%` de statements e `90,19%` de branches;
- `remoteEntry.js` do profile gerado em producao com `7,82 kB`, expondo `./Routes` e `./App`;
- prova HTTP local aprovada: shell em `4200` e remote em `4203` responderam `200` e o contrato do remote foi encontrado;
- arvore direta aprovada por `npm ls` e auditoria de dependencias de producao sem vulnerabilidades conhecidas nos dois projetos;
- pipeline de feature do shell alinhado ao reusable workflow Angular.

Excecoes temporarias e divida controlada:

- o Module Federation tradicional permanece sobre Webpack e `@angular-builders/custom-webpack`; Angular 22 marca esse builder como depreciado. A migracao para Application Builder deve ser feita junto com uma avaliacao de Native Federation, sem misturar essa troca estrutural com a atualizacao de framework;
- `extractLicenses` permanece desabilitado no shell e no profile porque o plugin de licencas do builder Webpack falha durante o build Angular 22. Dependencias e licencas continuam rastreaveis pelos manifests e lockfiles, mas a geracao de arquivo agregado precisa ser restaurada na migracao para o builder moderno;
- o dev-server Webpack responde a raiz e aos assets, mas nao aplica fallback para uma URL profunda requisitada diretamente, como `/login`. CloudFront deve continuar configurado para fallback SPA; a validacao local de navegacao direta deve ser refeita ao migrar o builder;
- os builds locais registraram aviso porque a maquina estava em Node `25.2.1`; o contrato dos projetos e das esteiras permanece Node `24.15.0`, que e a linha LTS suportada;
- o profile ainda possui alerta de budget no SCSS de cadastro de funcionario (`4,77 kB` contra aviso de `4 kB`) e arquivos SSR/environments incluidos sem uso. Esses alertas nao bloqueiam a onda, mas devem ser tratados antes da consolidacao.

## Onda 4A: Finance

Em 12 de novembro de 2026, o MFE Finance foi migrado de Angular 20 para 22 pelas migrations oficiais, mantendo Module Federation 22 e Design System 2.0. A execucao dos testes e zoneless. A validacao ocorreu em copia isolada fora do OneDrive com Node 24.15.0: 10 suites e 23 testes passaram, o build de producao gerou `remoteEntry.js` e a instalacao limpa pelo lockfile foi aprovada. A auditoria das dependencias de producao, apos atualizacao do lockfile, nao apresentou vulnerabilidades conhecidas.

As excecoes de Webpack e extracao de licencas descritas na onda 3 tambem se aplicam ao Finance. O teste funcional integrado ao shell, permissoes e telas financeiras permanece necessario antes da promocao para dev. A instalacao do lockfile original falhava por uma dependencia ausente; o lockfile foi reconciliado nesta onda.

Uma segunda instalacao limpa do lockfile final foi aprovada com Node 24.15.0. O servidor local do Finance compilou e respondeu `200` em `remoteEntry.js`, com as exposicoes `./Routes` e `./App`. A inspecao das rotas confirmou guards de leitura e de escrita, mas nao substitui o teste funcional autenticado no navegador. O repositorio privado `vyracare/vyracare-app-finance-mfe` foi criado e o commit de migracao enviado na branch de feature; as Actions permanecem desativadas ate a configuracao de segredos e infraestrutura.

## Onda 4B: Dashboard

O MFE Dashboard foi migrado de Angular 20 para Angular 22 e Design System 2 usando as migrations oficiais intermediarias e Module Federation 22. Os testes usam ambiente zoneless e timers do Jest. Em copia isolada com Node 24.15.0, passaram 37 testes em 9 suites, build de producao, instalacao limpa pelo lockfile e auditoria de dependencias de producao. O build gerou `remoteEntry.js`. As excecoes de Webpack e extracao de licencas da onda 3 tambem se aplicam; a validacao funcional autenticada junto ao shell ainda e necessaria antes da promocao.

## Onda 4C: Procedimentos

O MFE de Procedimentos foi migrado para Angular 22 e Design System 2 com as migrations oficiais e Module Federation 22. Em copia isolada com Node 24.15.0 passaram 19 testes em 5 suites, build de producao, instalacao limpa pelo lockfile e auditoria de dependencias de producao. O build gerou `remoteEntry.js`. As excecoes de Webpack e extracao de licencas descritas na onda 3 permanecem, assim como a necessidade de validacao funcional autenticada com o shell antes da promocao.

## Onda 4D: Pacientes

O MFE de Pacientes foi migrado para Angular 22 e Design System 2 com as migrations oficiais e Module Federation 22. Em copia isolada com Node 24.15.0 passaram 38 testes em 10 suites, build de producao, instalacao limpa pelo lockfile e auditoria de dependencias de producao. O build gerou `remoteEntry.js`. O SCSS da listagem atingiu 4,62 kB, acima do limite de aviso de 4 kB, e arquivos legados de funcionario e SSR foram apontados como nao utilizados; sua remocao requer avaliacao separada. As excecoes de Webpack e extracao de licencas da onda 3 e a validacao funcional autenticada com o shell permanecem pendentes antes da promocao.

## Criterios de aceite

- nenhum frontend usa uma versao Angular fora de suporte;
- Node, Angular, TypeScript, Jest e Module Federation possuem matriz explicita e reproduzivel;
- shell e remotos compartilham versoes compativeis de Angular e Design System;
- componentes macro novos continuam nascendo no Design System;
- APIs permanecem em runtime oficialmente gerenciado pela AWS;
- SDK e dependencias .NET sao reproduziveis e atualizados sem saltos de major nao auditados;
- documentacao, testes, commits e pushes acompanham cada onda.
