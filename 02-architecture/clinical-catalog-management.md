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

O `vyracare-app-profile-mfe` consulta `GET /api/auth/employees`, enviando `search` quando houver texto e limitando a resposta a 100 registros. A busca aceita nome, e-mail ou telefone.

A tabela apresenta somente dados operacionais retornados pela projecao segura da API:

- identificador, usado apenas para rastreamento da linha;
- nome completo;
- telefone;
- e-mail;
- cargo;
- indicacao de ativo, pois o endpoint retorna exclusivamente funcionarios ativos.

A resposta nao inclui senha, hash, token, segredo, nivel interno de credencial ou qualquer outro dado de autenticacao. Novas colunas nao devem ser adicionadas sem revisar o principio de minimizacao de dados e o contrato do backend.

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
