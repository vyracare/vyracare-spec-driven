# Arquitetura Frontend

## Modelo adotado

O frontend do Vyracare segue uma arquitetura de:

- shell Angular
- micro-frontends Angular
- design system compartilhado
- publicacao em S3 + CloudFront

## Camadas principais

### Design system
Fornece o contrato visual e de componentes.

### App shell
Responsavel por:

- layout base
- navegacao
- autenticacao
- configuracao de remotos
- orquestracao dos MFEs

### MFEs
Cada MFE representa um dominio funcional.

Cada projeto segue o mesmo principio:

- `environments.ts` para desenvolvimento local
- `environments.dev.ts` para `dev`
- `environments.hml.ts` para `hml`
- `environments.prod.ts` para `prod`

## Identidade da marca

O simbolo oficial do VyraCare combina a letra `V` com um coracao em espaco
negativo. O gradiente parte do violeta, atravessa o rosa e usa azul-ciano como
acento, preservando a paleta ja adotada pela interface. O simbolo nao recebe
texto, borda ou sombra quando usado em areas pequenas.

O shell e todos os MFEs usam o mesmo favicon WebP de `48x48`. Os titulos das
abas seguem `VyraCare` no shell e `VyraCare | <dominio>` quando o MFE e aberto
isoladamente. Novas aplicacoes frontend devem reutilizar esse ativo, sem
recriar variacoes locais da marca.

## Cabecalho das telas internas

As telas funcionais autenticadas, com excecao da pagina inicial do Dashboard, seguem um cabecalho visual comum:

- breadcrumb iniciado por `Dashboard` para indicar o contexto de navegacao;
- bloco de apresentacao com fundo em gradiente suave, borda e cantos arredondados;
- selo superior em caixa alta para identificar o dominio da tela;
- titulo e descricao objetiva da atividade;

A pagina inicial do Dashboard permanece com o hero e os indicadores proprios. A regra do cabecalho comum se aplica atualmente a Funcionarios, Procedimentos, Pacientes, Cadastro de paciente, Prontuario do paciente e Atendimentos.

Acoes primarias de negocio, como `Cadastrar paciente` e `Cadastrar atendimento`, ficam na barra da secao de conteudo logo abaixo do cabecalho. O bloco branco lateral de mensagem foi removido dos cabecalhos padronizados para reduzir ruido visual e deixar a apresentacao concentrada no titulo e na descricao.

As paginas principais de Funcionarios, Procedimentos e Atendimentos seguem tambem o padrao de gestao de Pacientes: card de listagem, busca quando aplicavel, tabela e acao primaria na barra do card. As acoes de cadastro navegam para paginas dedicadas com breadcrumb, hero e formulario em card branco. Esse fluxo evita formularios principais em modal e reserva modais para tarefas auxiliares ou confirmacoes, como configurar uma notificacao. Os contratos, campos exibidos e cuidados de minimizacao de dados estao detalhados em `clinical-catalog-management.md` e `dashboard-indicators.md`.

Em larguras menores, o cabecalho e as barras de acao devem ser empilhados para evitar rolagem horizontal e preservar a legibilidade.

As buscas das listagens de Pacientes, Funcionarios e Procedimentos usam `vc-search` com o modo de acao habilitado. O componente organiza em uma unica linha o campo flexivel e o botao de busca, que usa somente o icone branco de alto contraste, nome acessivel e tokens do Design System. Enter e clique emitem o termo normalizado pelo mesmo contrato. Os MFEs fornecem apenas o rotulo, placeholder e tratamento da pesquisa, sem recriar input, botao, tooltip ou estilos locais.

## Busca global e navegacao assistida

A busca do navbar e uma busca global de destinos da aplicacao, coordenada pelo shell. Enquanto o usuario digita, o shell consulta um catalogo unico de rotas, rotulos, descricoes e sinonimos e entrega ao `vc-navbar` no maximo cinco sugestoes. O catalogo inclui Dashboard, agenda, novos atendimentos, pacientes, funcionarios e procedimentos, incluindo seus fluxos de cadastro.

O `vc-navbar` e responsavel somente pela apresentacao e interacao generica do autocomplete. Ele implementa o contrato acessivel `combobox`/`listbox`, foco visivel, fechamento por Escape, navegacao circular com setas e emissao da sugestao selecionada. O componente nao conhece rotas nem regras de dominio. Clique ou selecao explicita por teclado em uma sugestao navega diretamente para o destino resolvido pelo shell.

Enter sem uma sugestao destacada navega para `/busca?q=<termo>`. A pagina de busca apresenta todas as possibilidades encontradas e mantem os destinos como links navegaveis. Termos vazios nao disparam navegacao; consultas desconhecidas exibem um estado vazio orientando exemplos validos. A correspondencia ignora maiusculas, minusculas e acentos e tambem considera sinonimos, como `prontuario`, `equipe`, `consulta` e `tratamento`.

O catalogo de destinos pertence ao shell porque agrega rotas de varios MFEs. Os MFEs continuam proprietarios das telas carregadas pelas rotas e nao devem duplicar o autocomplete global. Futuras buscas por entidades de negocio, como nomes de pacientes ou codigos de procedimentos, devem estender o contrato por provedores de dominio sem transferir chamadas de API para o Design System.

## Experiencia mobile-first

O breakpoint de referencia para a navegacao compacta e `720px`. Abaixo dele, a aplicacao deve se comportar como uma experiencia de app, sem reduzir a pagina desktop dentro da viewport:

- o navbar permanece fixo no topo, mostra marca, notificacoes e acesso ao perfil na primeira linha e dedica a segunda linha a busca global; notificacao, avatar e menu do perfil usam dimensao `md`, com alvos circulares equivalentes de `2.5rem` para preservar equilibrio visual e area de toque; o sino usa glifo `lg` de `1.25rem` dentro desse mesmo alvo para compensar seu menor peso visual;
- o subtitulo da marca e os textos extensos do perfil sao ocultados, mas nome, papel e acoes continuam acessiveis pelo menu;
- uma barra inferior fixa oferece acesso direto a `Inicio`, `Agenda` e `Pacientes`, mais a acao `Menu`; cada item possui icone, rotulo, area de toque adequada, foco visivel e estado ativo derivado da rota mais especifica;
- `Menu` abre o sidebar como bottom sheet com os cadastros de Funcionarios e Procedimentos e os demais destinos disponiveis; a barra inferior fica oculta enquanto o painel estiver aberto e retorna depois do fechamento por backdrop, botao dedicado, selecao de rota ou tecla Escape;
- o conteudo recebe espacamento inferior suficiente para nao ficar encoberto pela navegacao e usa `100%` da largura disponivel;
- listas tabulares de Pacientes, Funcionarios, Procedimentos e Atendimentos viram cartoes rotulados, eliminando a rolagem horizontal como interacao principal;
- formularios usam uma coluna, controles com altura minima de `44px` e fonte de `1rem`, evitando zoom automatico e melhorando o toque;
- grupos de acoes distribuidos entre as extremidades preservam o gutter lateral da pagina, evitando botoes encostados nas bordas da viewport;
- modais operacionais sao apresentados como bottom sheets, limitados a `88dvh` e com rolagem interna; no rodape, uma acao isolada permanece a direita, duas acoes mantêm cancelar a esquerda e confirmar ou salvar a direita, e conjuntos com tres ou mais acoes sao empilhados para evitar overflow;
- titulos, cards, paineis e gaps sao reduzidos de forma consistente, sem remover hierarquia visual ou informacao funcional.

Entre `721px` e `1024px`, o navbar usa uma grade compacta com busca na segunda linha, enquanto o sidebar continua lateral. Acima desse intervalo, permanece o layout desktop. Novas telas devem implementar primeiro a largura pequena e adicionar complexidade somente nos breakpoints maiores.

As regras estruturais de navbar, notificacoes, campos e sidebar pertencem ao `@vyracare/design-system`. O posicionamento do menu inferior e a reserva da area util pertencem ao shell. Os MFEs sao responsaveis apenas pela adaptacao semantica de seu conteudo, como transformar tabelas em cartoes e reorganizar formularios.

### Criterios de aceite mobile

- nenhuma pagina gera rolagem horizontal em `320px`, `375px` ou `430px`;
- nenhuma acao essencial fica sob o acionador flutuante ou fora da viewport;
- busca, notificacoes, perfil e cinco destinos principais continuam acessiveis;
- campos, botoes e acoes por icone preservam foco visivel, rotulo acessivel e area de toque adequada;
- mudancas de orientacao e areas seguras do dispositivo nao encobrem conteudo;
- o layout desktop continua funcional sem alteracao de contrato dos componentes.

## Componentes de formulario compartilhados

O pacote `@vyracare/design-system` e a fonte dos controles compartilhados. A partir da versao `0.5.0`, alem dos controles base, ele fornece campos semanticos para os dados recorrentes dos dominios:

- `vc-autocomplete`: ControlValueAccessor com label, hint, erro, carregamento, vazio, navegacao por teclado, lista acessivel e eventos de pesquisa/selecao. Os resultados sao exibidos em um painel flutuante sobre o conteudo, ancorado na largura do campo e sem alterar a altura ou o fluxo do formulario;
- `vc-modal`: superficie acessivel e responsiva com regioes projetadas de cabecalho, corpo e rodape. O componente centraliza backdrop, limite de viewport, rolagem vertical, espacamento entre titulo e conteudo, organizacao das acoes, fechamento por backdrop ou Escape e restauracao de foco;
- `vc-checkbox`: checkbox visual padronizado com label, descricao, erro e integracao com Angular Forms;
- `vc-select`: listbox customizado, sem depender da aparencia nativa diferente entre navegadores;
- `vc-input`: inclui mascaras de telefone, e-mail, data, CPF e CEP e emite o valor mascarado no desfoque;
- `vc-textarea`: campo multilinha reutilizavel com label, hint, erro, obrigatoriedade, limite de caracteres, estado desabilitado e integracao `ControlValueAccessor`;
- `vc-search`: campo de busca com integracao `ControlValueAccessor`; no modo de acao, inclui o botao lateral acessivel e emite o termo normalizado por clique ou Enter;
- `vc-phone-input`: fixa tipo telefonico, teclado adequado, placeholder nacional e mascara para dez ou onze digitos;
- `vc-email-input`: fixa tipo e teclado de e-mail e normaliza o valor sem espacos e em minusculas;
- `vc-date-time-input`: usa o controle nativo `datetime-local` dentro do mesmo layout, estados e contrato de Angular Forms;
- `vc-postal-code-input`: fixa teclado numerico, placeholder e mascara brasileira `00000-000`, preservando o evento de desfoque para consultas de endereco.

Os MFEs devem preferir os campos semanticos quando o dado corresponder a telefone, e-mail, data/hora ou CEP, deixando validacoes de negocio no formulario consumidor. Eles nao devem recriar mascaras, tipos nativos, textareas, selects, checkboxes ou buscas acionaveis quando existir equivalente compartilhado. Tambem nao devem recriar paineis ou estilos de autocomplete localmente. Empilhamento, sombra, estados interativos, truncamento de textos extensos e responsividade pertencem ao componente compartilhado. O `vyracare-app-dashboard-mfe` usa `vc-input`, `vc-select`, `vc-autocomplete` e os campos semanticos no atendimento e na notificacao, mantendo consultas e regras de dominio no MFE. O `vyracare-app-user-mfe` usa os controles compartilhados no cadastro, edicao e notas de pacientes. O `vyracare-app-profile-mfe` e o `vyracare-app-proceedings-mfe` usam os mesmos controles nos formularios e buscas de catalogo.

Uma verificacao de conformidade dos templates de aplicacao deve confirmar ausencia de `input`, `select`, `textarea` e `button` nativos nos MFEs. Elementos nativos continuam permitidos dentro da implementacao encapsulada do Design System, onde acessibilidade, estados e responsividade sao centralizados. Excecoes de dominio exigem justificativa arquitetural documentada e nao podem duplicar apenas aparencia ou comportamento ja fornecido pelo pacote.

Inputs e selects desabilitados recebem fundo e borda acinzentados, texto atenuado e cursor `not-allowed` diretamente pelo estilo-base do Design System. Assim, o mesmo estado visual e aplicado aos controles base e aos campos semanticos em todos os MFEs.

## Acoes por icone e tooltip

O Design System fornece `vc-icon-button` para acoes compactas e `vc-tooltip` para sua descricao flutuante. O tooltip aceita posicionamento superior, inferior, esquerdo ou direito, abre por hover ou foco de teclado e fecha na saida, perda de foco, tecla Escape ou movimentacao da viewport. Sua superficie usa coordenadas fixas calculadas a partir do gatilho, ficando acima de containers com `overflow` sem aumentar a largura de tabelas ou criar barras de rolagem. O texto visivel do tooltip complementa o `ariaLabel` obrigatorio do botao de icone.

Em viewports de ate `720px`, `vc-input` e o gatilho de `vc-select` usam a mesma altura real de `3.5rem`, calculada com `border-box`, fonte de `1rem` e padding interno. A altura explicita impede que controles nativos, como data, preservem uma dimensao intrinseca maior que o select. As opcoes do select mantêm altura minima de `3.25rem`. Essas dimensoes garantem proporcao entre controles e ampliam a area de toque sem alterar a densidade dos formularios no desktop.

Tabelas com varias acoes por registro devem preferir essa combinacao para reduzir largura e ruido visual, mantendo nomes objetivos para leitores de tela. Os MFEs nao devem depender apenas do atributo nativo `title` nem recriar a superficie flutuante localmente.

## Feedback global por toast

O Design System fornece `VcToastService` e `vc-toast-container` para mensagens transitorias de sucesso, erro, alerta e informacao. O shell mantem uma unica viewport flutuante montada acima das rotas. MFEs publicam feedback pelo servico e nao devem criar banners locais para o mesmo resultado.

O estado usa Angular Signals e um evento de navegador namespaced para sincronizar o shell e remotos mesmo quando mais de uma instancia fisica do pacote tiver sido carregada. Cada mensagem possui identificador, variante semantica, titulo, descricao e duracao; mensagens podem ser fechadas manualmente e sao removidas automaticamente por padrao. Erros usam live region assertiva e os demais estados usam live region educada.

O toast e o contrato obrigatorio de retorno para operacoes assincronas da interface:

- toda criacao, edicao, ativacao, inativacao, exclusao ou inclusao de nota confirmada pela API publica uma mensagem de sucesso;
- toda falha HTTP percebida pelo usuario publica uma mensagem de erro com titulo orientado a acao e descricao segura, sem detalhes internos, stack traces ou conteudo bruto inesperado do backend;
- a navegacao posterior a uma gravacao ocorre somente depois da publicacao do sucesso, permitindo que o container global preserve o feedback na tela de destino;
- mensagens inline podem permanecer quando ajudam a localizar o problema ou preservar o contexto da pagina, mas nao substituem o toast em falhas de requisicao;
- validacoes locais, como campo obrigatorio, formato invalido ou horario inconsistente, continuam proximas ao formulario e nao geram toast antes de existir uma requisicao;
- processos silenciosos recorrentes devem evitar repeticao ilimitada da mesma mensagem e podem aplicar deduplicacao ou retentativa antes de notificar.

Titulos devem identificar o resultado, como `Paciente atualizado` ou `Nao foi possivel cadastrar o atendimento`. A descricao informa a consequencia ou a proxima acao. O frontend deve preferir mensagens conhecidas por status e nunca apresentar diretamente objetos de erro, respostas HTML ou informacoes de infraestrutura.

O pacote `@vyracare/design-system` deve ser compartilhado como singleton na configuracao de Module Federation. Isso evita instancias duplicadas dos componentes, elimina colisoes `NG0912` e garante um contrato visual unico entre shell e MFEs.

Como o shell fornece a instancia singleton em tempo de execucao, sua versao do Design System deve ser igual ou superior a versao exigida por todos os MFEs carregados. Uma entrega de MFE que utilize uma nova exportacao somente pode ser promovida depois que o shell estiver alinhado com essa versao. Em ambiente local, depois da atualizacao do pacote, o processo do shell deve ser reiniciado para descartar a instancia anterior mantida em memoria. A ausencia desse alinhamento pode deixar um componente remoto indefinido e causar erros Angular como acesso a `ɵcmp` durante a ativacao da rota.

Cada remoto local deve estar disponivel na porta configurada pelo shell antes da navegacao: Dashboard `4201`, Pacientes `4202`, Perfil `4203` e Procedimentos `4204`. Se o carregamento remoto falhar, o `loadChildren` retorna uma rota curinga para a pagina de erro, inclusive em subrotas como `/pacientes/cadastro`; assim, indisponibilidade de infraestrutura nao e apresentada incorretamente como `NG04002 Cannot match any routes`.

## Documentacao do codigo TypeScript

Componentes TypeScript devem usar comentarios JSDoc objetivos para explicar a responsabilidade da classe e de seus metodos. A documentacao deve registrar a intencao e a regra coordenada pelo metodo, evitando apenas repetir seu nome ou descrever detalhes obvios de sintaxe. Metodos publicos, protegidos e privados criados ou alterados em uma entrega devem ser revisados como parte da mesma mudanca.

## Ambientes frontend

### Dev

- bucket com sufixo `-dev`
- sem blue/green
- deploy versionado por pasta na raiz do bucket
- CloudFront `dev` aponta para a pasta versionada ativa

### HML

- bucket sem `-dev`
- uso da pasta `blue/<timestamp>`
- CloudFront `hml` aponta para `blue`

### Prod

- bucket sem `-dev`
- uso da pasta `green/<timestamp>`
- CloudFront `prod` aponta para `green`

## Estrategia de build

### `develop`
Build Angular usa configuracao `dev`, com `fileReplacement` para `environments.dev.ts`.

### `release/*`
Build Angular usa configuracao `hml`, com `fileReplacement` para `environments.hml.ts`.

### `main`
Build Angular usa configuracao `production`, com `fileReplacement` para `environments.prod.ts`.

## Integracao com backends

As URLs de API podem ser atualizadas automaticamente pelas esteiras `.NET`.

A regra atual e:

- branch `develop` -> atualiza `environments.dev.ts`
- branch `release/*` -> atualiza `environments.hml.ts`
- branch `main` -> atualiza `environments.prod.ts`

Isso vale para:

- shell consumidor de auth
- MFEs consumidores de APIs especificas

### Dashboard

O `vyracare-app-dashboard-mfe` nao mantem valores demonstrativos nos cards integrados. Ele consulta:

- `appointmentsApiUrl` para os indicadores operacionais;
- `financeApiUrl` para os indicadores de saude financeira.

As duas chamadas enviam o JWT do usuario. O ambiente local aponta para as portas `5003` e `5004`; os arquivos de `dev`, `hml` e `prod` devem receber as URLs publicadas pelas esteiras das APIs.

## Risco conhecido

Quando automacoes alteram arquivos de environment, o maior risco e gerar:

- propriedades duplicadas
- virgulas duplicadas
- arquivo JSON/TS invalido

As pipes atuais ja possuem normalizacao para reduzir esse risco, mas isso continua sendo um ponto sensivel da arquitetura de sincronizacao entre repositorios.
