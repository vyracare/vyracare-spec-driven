# Auditoria de Conformidade Arquitetural

## Escopo

Auditoria executada em 7 de outubro de 2026 sobre o shell, quatro MFEs de dominio, Design System e cinco APIs .NET. O objetivo foi confrontar as alteracoes funcionais e responsivas com as arquiteturas frontend e backend, validar dependencias e registrar dividas tecnicas sem mascarar alertas.

## Resultado frontend

- shell e MFEs preservam a separacao por dominio e carregamento por Module Federation;
- `@vyracare/design-system` esta em `^0.11.0` e configurado como singleton no shell e em todos os MFEs;
- navegacao mobile pertence ao shell; controles, modais, icones, feedback e estados visuais compartilhados permanecem no Design System;
- adaptacoes semanticas de tabelas, formularios e barras de acoes permanecem nos MFEs consumidores;
- os templates dos quatro MFEs nao possuem `input`, `select`, `textarea` ou `button` nativos; campos multilinha usam o novo `vc-textarea`, buscas de listagem usam `vc-search` no modo de acao e os demais casos reutilizam `vc-input`, `vc-select`, `vc-checkbox`, campos semanticos e `vc-button`;
- o shell preserva subrotas em falhas de Module Federation por meio de fallback curinga, e os cinco servidores locais responderam com HTTP 200 nas portas `4200` a `4204`;
- feedback de sucesso e falha usa o toast compartilhado, e gravacoes navegam somente depois da confirmacao da API;
- build dos cinco aplicativos consumidores, build da biblioteca e build do Storybook foram aprovados;
- 349 testes frontend foram aprovados: Dashboard 37, Procedimentos 19, Perfil 30, Shell 79, Pacientes 38 e Design System 146.

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
- a instalacao local ainda reporta vulnerabilidades transitivas nas cadeias de ferramentas frontend legadas; a correcao deve ocorrer em uma atualizacao dedicada e validada, sem aplicar `npm audit fix --force` automaticamente;
- diretorios locais `bin`, `obj` e `publish-net10`, alem de locks locais de Terraform, sao artefatos de diagnostico e nao fazem parte dos commits.

Esses itens nao alteram contratos nem impedem build ou testes, mas devem ser reduzidos em entregas dedicadas. Novas mudancas nao devem ampliar budgets, duplicar componentes compartilhados ou incluir artefatos gerados para silenciar os avisos.

## Criterio de conformidade

Uma entrega permanece conforme quando respeita a responsabilidade de cada repositorio, compila, passa nos testes do dominio, nao introduz segredo ou vulnerabilidade conhecida e atualiza esta especificacao quando altera um contrato compartilhado. Alertas preexistentes devem ser reportados de forma explicita e nao ocultados por relaxamento de limites globais.

## Incremento multi-tenant de 9 de outubro de 2026

O primeiro incremento SaaS foi revisado contra os mesmos criterios:

- o novo `vyracare-api-tenancy` preserva o padrao `Application/Domain/Infrastructure`, portas e adapters Mongo;
- a duracao do trial esta centralizada em `TenancyOptions`, com 30 dias por padrao e teste unitario dedicado;
- autenticacao coordena o onboarding por `ITenancyProvisioner`, sem acoplar o handler ao `HttpClient`;
- falha de provisionamento compensa a identidade recem-criada e responde `503`;
- segredo backend-backend vem de configuracao/Parameter Store e nao e exposto ao Angular;
- JWT passou a emitir `tenant_id`, `membership_id`, `tenant_role` e `plan`;
- a API client resolve contexto a partir do principal autenticado, rejeita token incompleto e inclui tenant em todos os filtros Mongo de pacientes e funcionarios;
- unicidade de CPF e e-mail passou a ser composta por tenant, preservando registros legados fora do indice parcial;
- o shell reutiliza `vc-input` e `vc-button` no onboarding e nao calcula datas comerciais no navegador;
- contas autenticadas sem tenant sao desviadas para um onboarding recuperavel, que preserva a identidade e emite novo JWT depois do provisionamento;
- builds e testes dos quatro repositorios alterados foram aprovados no fechamento do incremento.

Limites ainda abertos, tratados como proximos incrementos e nao como capacidade entregue:

- troca de tenant para usuarios com mais de uma membership;
- tela de convite e aceite no shell;
- migracao da gestao antiga de funcionarios da auth para memberships;
- propagacao do `TenantContext` para procedimentos, agenda e financeiro;
- migracao assistida dos registros legados que ainda nao possuem `tenantId`;
- infraestrutura AWS e pipeline do novo servico de tenancy.

Enquanto esses itens nao forem concluidos, o piloto multi-tenant deve ficar restrito ao onboarding de proprietario e ao dominio de clientes. Nao se deve anunciar isolamento integral da plataforma.
