# Gestao Financeira e Notificacoes Operacionais

## Objetivo

Este documento define o primeiro incremento operacional do dominio financeiro e o contrato de notificacoes exibido pelo shell. O incremento substitui atalhos sem destino e notificacoes demonstrativas por dados persistidos, isolados por tenant e consumidos por um MFE dedicado.

## Fronteiras do dominio

- `vyracare-api-finance` e proprietaria de lancamentos, boletos, resumos e alertas financeiros;
- `vyracare-api-appointments` continua proprietaria dos lembretes de atendimentos;
- `vyracare-app-finance-mfe` e proprietario das telas financeiras;
- `vyracare-app-dashboard-mfe` consome somente o resumo financeiro;
- `vyracare-app-shell` agrega notificacoes operacionais e coordena navegacao e leitura;
- o Design System fornece controles, cards, feedback e superficies genericas, sem regras financeiras.

## Modelo de dados

### Movimentacao financeira

- `id` e `tenantId` imutaveis;
- `type`: `Revenue` ou `Expense`;
- `status`: `Planned`, `Confirmed` ou `Cancelled`;
- `amount` decimal positivo;
- `description` obrigatoria;
- `kind`: `ClientReceipt`, `ProductPurchase`, `Withdrawal`, `ProLabore`, `OperatingExpense`, `OtherRevenue` ou `OtherExpense`;
- `paymentMethod`: `Pix`, `Cash`, `DebitCard`, `CreditCard`, `BankTransfer`, `Boleto` ou `Other`;
- `counterparty`: cliente, fornecedor ou socio relacionado;
- `category`: centro de custo ou classificacao interna;
- `notes`: observacoes opcionais;
- `installmentNumber`, `installmentCount` e `installmentGroupId` identificam parcelas relacionadas quando aplicavel;
- `occurredAt` em UTC;
- `createdAt` e `updatedAt` em UTC.

### Recebimento / conta a receber

- `id` e `tenantId` imutaveis;
- descricao, cliente e valor total positivo;
- forma de pagamento explicita, incluindo Pix e dinheiro;
- quantidade de parcelas entre 1 e 60;
- no cadastro parcelado, a API divide o valor total, corrige eventual residuo na ultima parcela e cria vencimentos mensais agrupados por `installmentGroupId`;
- vencimento em UTC;
- `status`: `Pending`, `Paid` ou `Cancelled`;
- `notificationReadAt` nulo enquanto o alerta estiver nao lido;
- `createdAt` e `updatedAt` em UTC.

Documentos legados sem `tenantId` nao podem ser retornados por consultas autenticadas. A migracao ou atribuicao desses documentos exige procedimento administrativo explicito; nunca e inferida pelo navegador.

## Contratos da API financeira

Todas as rotas exigem JWT valido e contexto completo de tenant.

- `GET /api/finance/dashboard/summary?month=yyyy-MM`;
- `GET /api/finance/entries?month=yyyy-MM&type=&status=&kind=&search=`;
- `POST /api/finance/entries`;
- `PATCH /api/finance/entries/{id}/status`;
- `GET /api/finance/invoices?status=&search=`;
- `POST /api/finance/invoices`;
- `PATCH /api/finance/invoices/{id}/status`;
- `GET /api/finance/notifications/unread`;
- `POST /api/finance/notifications/{invoiceId}/read`.

Filtros sao opcionais, normalizados no backend e sempre compostos com `tenantId`. Identificadores de outro tenant recebem `404`, sem revelar a existencia do recurso.

## MFE financeiro

O novo remoto usa o nome `vyracare-app-finance-mfe`, porta local `4205` e rota raiz `/financeiro`.

Subpaginas do modulo:

- `/financeiro`: visao consolidada do mes;
- `/financeiro/recebimentos`: pagamentos de clientes, parcelas e vencimentos;
- `/financeiro/recebimentos/novo`: cadastro de recebimento a vista ou parcelado;
- `/financeiro/pagamentos`: despesas, retiradas e pro-labore;
- `/financeiro/pagamentos/novo`: cadastro de saida preconfigurado como despesa operacional;
- `/financeiro/compras`: compras de produtos e insumos;
- `/financeiro/compras/nova`: cadastro de saida preconfigurado como compra de produto;
- `/financeiro/movimentacoes`: livro de caixa consolidado;
- `/financeiro/movimentacoes/novo`: cadastro generico de entrada ou saida;
- rotas legadas de `lancamentos`, `receitas`, `despesas` e `boletos` redirecionam para as novas subpaginas.

O shell apresenta `Financeiro` como um grupo expansivel no menu principal. As subrotas sao exibidas logo abaixo do item pai, usando o segundo nivel generico do `VcSidebar`; o MFE nao cria uma segunda coluna de navegacao. O grupo abre ao ser selecionado e permanece expandido enquanto uma rota financeira estiver ativa. Em telas menores, os destinos continuam disponiveis no menu completo do shell, sem competir com a navegacao inferior.

As telas seguem breadcrumb, hero, card de conteudo, busca e barras de acao ja adotados nos outros MFEs. Formularios usam componentes do `@vyracare/design-system`; validacoes de negocio permanecem no MFE e na API. No mobile, listas viram cards e acoes preservam alvos de toque e gutters laterais.

## Notificacoes dinamicas

O shell consulta periodicamente as fontes autenticadas e agrega um contrato visual unico:

- agenda: lembretes vencidos de atendimentos ainda nao lidos;
- financeiro: boletos pendentes vencidos ou com vencimento nos proximos tres dias.

Regras:

- nenhum item demonstrativo e exibido;
- o badge corresponde exatamente ao total de itens nao lidos renderizados;
- fontes indisponiveis retornam lista vazia sem inventar notificacoes;
- identificadores recebem prefixo de dominio para evitar colisao;
- selecionar um item marca a notificacao como lida e navega para o contexto correspondente;
- `Ver todas` navega para uma pagina do shell que apresenta a mesma colecao agregada;
- a consulta aplica deduplicacao e nao publica toasts repetidos por falha recorrente;
- a notificacao nativa do navegador pode ser exibida uma vez por sessao, mas nao marca o item como lido automaticamente.

Para agenda, `notificationSentAt` continua representando entrega nativa. Um novo `notificationReadAt` representa leitura no centro de notificacoes. Para financeiro, somente `notificationReadAt` e necessario neste incremento.

## Seguranca e multi-tenancy

- `TenantContextMiddleware` rejeita requisicoes autenticadas sem `tenant_id`, `membership_id` ou `tenant_role`;
- repositorios recebem `ITenantContext` e iniciam todos os filtros por `tenantId`;
- indices relevantes incluem `tenantId` como primeiro campo;
- respostas nunca incluem connection strings, claims internas ou dados de outro tenant;
- testes cobrem leitura, alteracao e notificacoes entre dois tenants.

### Matriz de permissao financeira

O acesso e avaliado com claims assinadas pelo backend e aplicado em tres camadas: visibilidade no shell, guards do shell/MFE e policies da API. Ocultar o menu nunca substitui a autorizacao da API.

| Contexto do usuario | Consulta (`Finance.Read`) | Gestao (`Finance.Manage`) |
|---|---:|---:|
| `tenant_role` Owner ou Administrator | sim | sim |
| `access_level` Administrador | sim | sim |
| `access_level` Gestor | sim | nao, exceto setor Financeiro |
| `job_role` Financeiro + Gestor/Operacional | sim | sim |
| `job_role` Financeiro + Leitura | sim | nao |
| demais perfis | nao | nao |

Operacoes de consulta exigem `Finance.Read`. Criacao e mudanca de status exigem `Finance.Manage` e retornam `403` quando o token nao possui permissao. O shell remove Financeiro do menu e da busca global para usuarios sem leitura; acesso direto a `/financeiro` redireciona ao dashboard.

## Criterios de aceite

- o shell carrega o MFE financeiro pela porta `4205` em ambiente local;
- recebimentos a vista ou parcelados, compras, despesas, retiradas e pro-labore podem ser classificados e acompanhados;
- o item Financeiro do menu principal expande visao geral, recebimentos, pagamentos, compras e caixa, sem uma barra lateral adicional;
- usuarios sem `Finance.Read` nao visualizam nem acessam o modulo e usuarios somente leitura nao recebem acoes de mutacao;
- o resumo do dashboard reflete somente dados confirmados do tenant autenticado;
- nenhum recurso financeiro de outro tenant pode ser consultado ou alterado;
- o sino permanece sem badge quando nao existem notificacoes;
- o badge e a lista usam a mesma quantidade real de itens nao lidos;
- selecionar uma notificacao a remove da lista apos confirmacao da API;
- agenda e financeiro continuam funcionais quando uma das fontes de notificacao estiver indisponivel;
- suites .NET e Angular permanecem verdes;
- templates nao recriam inputs, selects, textareas ou botoes quando houver componente equivalente no Design System.

## Entrega incremental

1. contratos, persistencia e isolamento por tenant na API financeira;
2. contratos de leitura de notificacao na agenda e no financeiro;
3. MFE financeiro e integracao com o shell;
4. agregador dinamico de notificacoes no shell;
5. testes integrados, auditoria arquitetural e atualizacao de pipelines/ambientes.

## Estado implementado em 2026-11-11

- API Finance: contratos de lancamentos e boletos, resumo mensal, leitura de notificacoes, contexto de tenant e indices tenant-first implementados;
- API Appointments: contexto de tenant, notificacoes nao lidas independentes da entrega nativa e indices tenant-first implementados;
- MFE Finance: visao geral, listas, buscas, formularios e mudancas de status implementados com Design System e responsividade mobile;
- Shell: rota local do financeiro, menu, busca global, pagina de notificacoes e agregador com polling de 60 segundos implementados;
- badge: derivado diretamente do tamanho da mesma colecao nao lida exibida pelo `vc-navbar`;
- falhas de uma fonte: isoladas com colecao vazia, sem notificacoes ficticias e sem bloquear a outra fonte;
- logout: encerra o polling e limpa a colecao local;
- validacao: 18 testes Angular do MFE financeiro com 100% de linhas e 92% de branches; suites .NET de Finance e Appointments verdes.

## Evolucao implementada em 2026-11-11

- dominio: movimentacoes passaram a registrar natureza, forma de pagamento, contraparte, centro de custo e observacoes;
- recebimentos: cadastro aceita cliente, Pix, dinheiro, cartoes, transferencia ou boleto e gera de 1 a 60 parcelas mensais;
- operacao: compras de produtos, retiradas, pro-labore e despesas operacionais possuem classificacoes proprias;
- navegacao: dropdown expansivel no menu principal do shell e subpaginas de recebimentos, pagamentos, compras e caixa;
- seguranca: policies `Finance.Read` e `Finance.Manage` na API, guards no shell/MFE e menu/busca filtrados;
- compatibilidade: URLs antigas redirecionam para as novas subpaginas;
- validacao: 24 testes no MFE financeiro, 113 no shell e 11 na API financeira, todos verdes; builds Angular e .NET aprovados.

### Pendencia de infraestrutura

O `vyracare-app-finance-mfe` existe localmente e ainda nao possui repositorio remoto, bucket, distribuicao CloudFront ou URL de `remoteEntry` publicada. O shell habilita `/financeiro` somente quando `financeRemoteEntry` nao estiver vazio. Isso mantem `dev`, `hml` e `prod` estaveis ate o provisionamento explicito da nova aplicacao.
