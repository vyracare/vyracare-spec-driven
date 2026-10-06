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

Os formularios de cadastro sao abertos em modal a partir das acoes `Cadastrar funcionario` e `Cadastrar procedimento`. Depois de uma gravacao bem-sucedida, o modal fecha e a listagem e recarregada. Enquanto uma gravacao estiver em andamento, o modal nao pode ser fechado pelo backdrop.

## Funcionarios

O `vyracare-app-profile-mfe` usa uma projecao administrativa protegida por role `Administrador`:

- `GET /api/auth/employees/manage?search={termo}&limit=100`: lista ativos e inativos;
- `POST /api/auth/employees`: cadastra um funcionario com cargo, nivel e status definidos pelo administrador;
- `GET /api/auth/employees/{id}`: carrega os dados editaveis;
- `PUT /api/auth/employees/{id}`: atualiza nome, e-mail, telefone, cargo, departamento, nivel de acesso e status;
- `PATCH /api/auth/employees/{id}/status`: ativa ou inativa rapidamente.

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

A acao rapida de status tambem exige confirmacao. O backend impede que o administrador inative o proprio usuario, evitando bloqueio acidental. Um funcionario inativo nao aparece nos autocompletes e nao consegue realizar novos logins. Como a autenticacao usa JWT sem lista de revogacao, uma sessao emitida antes da inativacao permanece valida somente ate a expiracao normal do token.

A resposta administrativa inclui nivel de acesso apenas porque a rota e exclusiva de administradores e esse campo precisa ser editado. Ela nao inclui senha, hash, token, segredo ou qualquer outro dado de autenticacao. Novas colunas nao devem ser adicionadas sem revisar o principio de minimizacao de dados e o contrato do backend.

## Procedimentos

O `vyracare-app-proceedings-mfe` carrega o catalogo pelo servico de procedimentos. A busca local filtra, sem diferenciar maiusculas e minusculas, por:

- nome;
- codigo;
- categoria.

A tabela apresenta nome, codigo, categoria, duracao, valor por sessao e status. O cadastro reutiliza o formulario existente e, ao concluir, recarrega o catalogo para refletir o novo procedimento.

## Seguranca de entrega

Antes de publicar alteracoes nesses MFEs, a revisao deve confirmar que:

- nenhum arquivo de ambiente recebeu credenciais, tokens ou chaves privadas;
- fixtures e testes usam apenas dados ficticios;
- somente a projecao operacional de funcionarios e renderizada;
- artefatos temporarios, respostas de API e arquivos locais nao estao no conjunto de arquivos versionados;
- o diff staged foi inspecionado antes do commit e do push.
