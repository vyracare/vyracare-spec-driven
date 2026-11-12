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

## Criterios de aceite

- nenhum frontend usa uma versao Angular fora de suporte;
- Node, Angular, TypeScript, Jest e Module Federation possuem matriz explicita e reproduzivel;
- shell e remotos compartilham versoes compativeis de Angular e Design System;
- componentes macro novos continuam nascendo no Design System;
- APIs permanecem em runtime oficialmente gerenciado pela AWS;
- SDK e dependencias .NET sao reproduziveis e atualizados sem saltos de major nao auditados;
- documentacao, testes, commits e pushes acompanham cada onda.
