# Gestao de funcionarios e procedimentos

Este documento define o padrao de navegacao e apresentacao aplicado aos dominios de funcionarios e procedimentos.

## Padrao visual

As telas principais dos dois dominios seguem a mesma hierarquia visual da consulta de pacientes:

- breadcrumb com retorno ao Dashboard;
- hero em gradiente com selo de contexto, titulo e descricao;
- card branco de gestao com titulo, texto auxiliar e acao primaria;
- busca antes da tabela;
- estados explicitos de carregamento, lista vazia, sucesso e falha;
- tabela responsiva com rolagem horizontal em larguras reduzidas.

As acoes `Cadastrar funcionario` e `Cadastrar procedimento` navegam para paginas dedicadas, seguindo o mesmo modelo do cadastro de pacientes. Essas paginas possuem breadcrumb com o nivel `Cadastrar`, hero em gradiente e card branco contendo o formulario reutilizavel. Depois de uma gravacao bem-sucedida, o usuario retorna para a listagem correspondente; em caso de falha, permanece na pagina com os dados preenchidos e feedback de erro.

## Funcionarios

O `vyracare-app-profile-mfe` usa uma projecao administrativa protegida por role `Administrador`:

- `GET /api/auth/employees/manage?search={termo}&limit=100`: lista ativos e inativos;
- `POST /api/auth/employees`: cadastra um funcionario com cargo, nivel e status definidos pelo administrador;
- `GET /api/auth/employees/{id}`: carrega os dados editaveis;
- `PUT /api/auth/employees/{id}`: atualiza nome, e-mail, telefone, cargo, departamento, nivel de acesso e status;
- `PATCH /api/auth/employees/{id}/status`: ativa ou inativa rapidamente.
- `DELETE /api/auth/employees/{id}`: exclui definitivamente um funcionario, somente por administrador.

A busca aceita nome, e-mail ou telefone. O endpoint operacional `GET /api/auth/employees`, consumido por autocompletes de atendimento, permanece separado e continua retornando somente funcionarios ativos.

O cadastro publico `POST /api/auth/register` nao aceita elevacao de privilegio: cargo, departamento, telefone e status enviados diretamente sao ignorados, e o novo usuario recebe somente o nivel `Leitura`. O cadastro completo de funcionario e exclusivo da rota administrativa autenticada.

A tabela apresenta somente dados operacionais retornados pela projecao segura da API:

- identificador, usado apenas para rastreamento da linha;
- nome completo;
- telefone;
- e-mail;
- cargo;
- indicacao de ativo ou inativo;
- acoes por icones do Design System para editar e alterar rapidamente o status, com tooltip flutuante e `ariaLabel` equivalente.

A edicao usa a rota `/cadastro/funcionarios/editar/{id}` e reutiliza o formulario compartilhado. O formulario e preenchido pela API e solicita confirmacao antes de salvar. Senha, hash e demais dados de credencial nao fazem parte do contrato, nao sao exibidos e sao preservados pelo backend. Alteracoes de nivel de acesso passam a valer quando o funcionario entrar novamente.

Apos a API confirmar a criacao ou a edicao de um funcionario, o MFE publica o toast global de sucesso e retorna para a grade em `/cadastro/funcionarios`. Alteracoes rapidas de status e exclusoes confirmadas tambem usam o mesmo retorno. Em caso de erro, a navegacao nao ocorre, os dados permanecem disponiveis para correcao e o MFE publica um toast de erro com mensagem segura. Falhas ao carregar a grade ou a ficha podem manter o aviso contextual na pagina, mas tambem devem usar o toast global.

O cadastro usa a rota `/cadastro/funcionarios/novo`. A listagem continua em `/cadastro/funcionarios`, e sua acao primaria apenas navega para a pagina de cadastro, sem manter estado de modal.

A acao rapida de status tambem exige confirmacao. O backend impede que o administrador inative o proprio usuario, evitando bloqueio acidental. Um funcionario inativo nao aparece nos autocompletes e nao consegue realizar novos logins. Como a autenticacao usa JWT sem lista de revogacao, uma sessao emitida antes da inativacao permanece valida somente ate a expiracao normal do token.

A exclusao e uma acao permanente apresentada por icone e tooltip na tabela. Ela exige um modal de confirmacao que informa a remocao do cadastro e da credencial. A API aplica novamente a autorizacao `Administrador`, valida a existencia do funcionario e impede a autoexclusao; a interface remove a linha somente depois de receber sucesso do `DELETE`. Tokens previamente emitidos para um usuario excluido permanecem validos ate a expiracao enquanto nao houver revogacao central, portanto a inativacao deve ser preferida quando for necessario bloquear o acesso antes da remocao definitiva.

Os modais de confirmacao de status e exclusao usam superficie branca, cantos arredondados, sombra e largura limitada a viewport. O conteudo nao pode ficar transparente sobre a tabela nem gerar rolagem horizontal; quando a altura disponivel for insuficiente, somente o eixo vertical do card pode rolar, e as acoes sao empilhadas em telas estreitas.

Novos modais e migracoes de modais existentes devem usar o `vc-modal` do Design System. As regioes `vcModalHeader`, `vcModalBody` e `vcModalFooter` preservam a separacao visual entre titulo, mensagem e acoes; paginas consumidoras mantem apenas regras realmente especificas do dominio. A confirmacao de edicao de funcionario segue esse padrao e bloqueia o fechamento enquanto a gravacao estiver em andamento.

No formulario compartilhado de funcionarios, o bloco `Status ativo` mantem espacamento vertical proprio em relacao ao grid de dados e a barra de acoes, preservando a separacao visual no cadastro e na edicao.

A resposta administrativa inclui nivel de acesso apenas porque a rota e exclusiva de administradores e esse campo precisa ser editado. Ela nao inclui senha, hash, token, segredo ou qualquer outro dado de autenticacao. Novas colunas nao devem ser adicionadas sem revisar o principio de minimizacao de dados e o contrato do backend.

## Procedimentos

O `vyracare-app-proceedings-mfe` carrega o catalogo pelo servico de procedimentos. A busca local filtra, sem diferenciar maiusculas e minusculas, por:

- nome;
- codigo;
- categoria.

A tabela apresenta nome, codigo, categoria, duracao, valor por sessao e status. O cadastro usa a rota `/cadastro/procedimentos/novo`, reutiliza o formulario existente e retorna ao catalogo em `/cadastro/procedimentos` depois da gravacao.

O retorno para a grade de procedimentos ocorre somente depois da confirmacao de sucesso da API e da publicacao do toast `Procedimento cadastrado`. Falhas de gravacao preservam a pagina e os valores preenchidos para nova tentativa, exibindo o retorno pelo toast global e, quando util, tambem no contexto do formulario.

O formulario de procedimentos usa o padrao de editor de protocolo dentro de uma unica superficie. Em telas amplas, identificacao e apresentacao ocupam a coluna principal, enquanto `Agenda e cobranca` forma um painel lateral de consulta rapida com duracao, numero de sessoes, preco e disponibilidade. Marcadores visuais distinguem os contextos sem criar uma pilha de cards independentes ou impor uma ordem artificial entre as colunas. Em tablets, os dados operacionais formam uma faixa de tres colunas abaixo do conteudo; em telas estreitas, todos os campos e as acoes sao empilhados.

Os campos de identificacao usam uma grade propria do componente, sem depender de classes utilitarias globais: nome ocupa a linha completa, categoria e codigo formam um par, e area principal e recuperacao formam outro. A descricao comercial permanece ampla, mas com altura inicial controlada. Campos obrigatorios sao identificados de forma consistente, o preco explicita a moeda brasileira e o codigo interno apresenta orientacao para facilitar buscas. A disponibilidade para agendamento usa o `vc-checkbox` do Design System, com uma descricao clara do efeito operacional do estado ativo. O rodape reúne a orientacao de revisao e as acoes `Limpar` e `Salvar procedimento`, preservando hierarquia e responsividade.

## Seguranca de entrega

Antes de publicar alteracoes nesses MFEs, a revisao deve confirmar que:

- nenhum arquivo de ambiente recebeu credenciais, tokens ou chaves privadas;
- fixtures e testes usam apenas dados ficticios;
- somente a projecao operacional de funcionarios e renderizada;
- artefatos temporarios, respostas de API e arquivos locais nao estao no conjunto de arquivos versionados;
- o diff staged foi inspecionado antes do commit e do push.

## Criterios de conformidade da gestao de funcionarios

A entrega de gestao de funcionarios somente esta completa quando os seguintes criterios forem atendidos em conjunto:

- o frontend reutiliza os componentes de botao, icone, tooltip, formulario, select, checkbox e toast do Design System quando houver equivalente compartilhado;
- a listagem segue a hierarquia visual de Pacientes, mantem busca, estados de carregamento e vazio, tabela responsiva e acoes acessiveis por teclado;
- toda alteracao de status, edicao definitiva e exclusao exigem confirmacao explicita do administrador;
- componentes e servicos TypeScript documentam as responsabilidades das classes e dos metodos de producao adicionados ou alterados;
- controllers, handlers, contratos e portas do backend mantem documentacao coerente com a responsabilidade publica de cada operacao;
- autorizacao e regras de seguranca sao aplicadas pela API, independentemente da visibilidade dos controles no MFE;
- respostas administrativas usam `EmployeeManagementResponse` e nunca serializam senha, hash ou token;
- cadastro publico nao aceita perfil privilegiado, e a autoinativacao e a autoexclusao administrativas permanecem bloqueadas;
- testes automatizados cobrem carregamento, busca, cadastro, edicao, confirmacao, ativacao, inativacao, falhas HTTP e regras de seguranca;
- build, testes, inspecao do diff e varredura de segredos precisam estar aprovados antes do push.

Arquivos gerados por build, publicacao ou diagnostico local, como `bin`, `obj`, `publish-*` e respostas temporarias, nao fazem parte da especificacao nem devem ser incluidos em commits. Mudancas mecanicas de lockfile sem alteracao intencional de dependencias tambem devem permanecer fora da entrega.
