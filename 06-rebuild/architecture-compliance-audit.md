# Auditoria de Conformidade Arquitetural

## Escopo

Auditoria executada em 7 de outubro de 2026 sobre o shell, quatro MFEs de dominio, Design System e cinco APIs .NET. O objetivo foi confrontar as alteracoes funcionais e responsivas com as arquiteturas frontend e backend, validar dependencias e registrar dividas tecnicas sem mascarar alertas.

## Resultado frontend

- shell e MFEs preservam a separacao por dominio e carregamento por Module Federation;
- `@vyracare/design-system` esta em `^0.10.0` e configurado como singleton no shell e em todos os MFEs;
- navegacao mobile pertence ao shell; controles, modais, icones, feedback e estados visuais compartilhados permanecem no Design System;
- adaptacoes semanticas de tabelas, formularios e barras de acoes permanecem nos MFEs consumidores;
- feedback de sucesso e falha usa o toast compartilhado, e gravacoes navegam somente depois da confirmacao da API;
- build dos cinco aplicativos consumidores, build da biblioteca e build do Storybook foram aprovados;
- 339 testes frontend foram aprovados: Dashboard 36, Procedimentos 19, Perfil 30, Shell 79, Pacientes 38 e Design System 137.

## Resultado backend

- as cinco APIs usam `.NET 10`, separacao `Features`, `Common` e `Infrastructure`, persistencia MongoDB por adapters e projetos de teste separados;
- autenticacao e autorizacao sao aplicadas antes do mapeamento dos controllers; endpoints anonimos sao excecoes explicitas de autenticacao ou saude;
- configuracoes sensiveis permanecem em variaveis de ambiente ou Parameter Store; a varredura nao encontrou credenciais reais versionadas;
- `MongoDB.Driver` foi atualizado de `2.24.0` para `3.12.0` nas cinco APIs;
- 41 testes backend foram aprovados: Appointments 4, Authentication 16, Client 14, Proceedings 5 e Finance 2;
- `dotnet list package --vulnerable --include-transitive` nao encontrou pacotes vulneraveis em nenhuma API depois da atualizacao.

## Dividas conhecidas nao bloqueantes

- builds Angular ainda informam arquivos SSR e alguns environments incluidos na compilacao sem uso direto nos bundles de Module Federation;
- `employee-registration.component.scss` e `patients-page.component.scss` permanecem acima do budget de aviso de `4 kB`; nao foi aumentado o limite para ocultar o alerta;
- os testes Angular em modo zoneless ainda carregam Zone.js e emitem o aviso `NG0914`;
- a maquina local usa uma versao impar nao LTS do Node.js; pipelines devem continuar usando a versao LTS definida pelas esteiras;
- diretorios locais `bin`, `obj` e `publish-net10`, alem de locks locais de Terraform, sao artefatos de diagnostico e nao fazem parte dos commits.

Esses itens nao alteram contratos nem impedem build ou testes, mas devem ser reduzidos em entregas dedicadas. Novas mudancas nao devem ampliar budgets, duplicar componentes compartilhados ou incluir artefatos gerados para silenciar os avisos.

## Criterio de conformidade

Uma entrega permanece conforme quando respeita a responsabilidade de cada repositorio, compila, passa nos testes do dominio, nao introduz segredo ou vulnerabilidade conhecida e atualiza esta especificacao quando altera um contrato compartilhado. Alertas preexistentes devem ser reportados de forma explicita e nao ocultados por relaxamento de limites globais.
