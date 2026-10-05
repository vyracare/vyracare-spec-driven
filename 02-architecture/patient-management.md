# Gestao de Pacientes

Este documento define a navegacao, os contratos e as regras de acesso da gestao de pacientes.

## Rotas do frontend

O `vyracare-app-shell` monta o `vyracare-app-user-mfe` em `/pacientes`.

| Rota | Responsabilidade |
| --- | --- |
| `/pacientes` | consulta e busca de pacientes |
| `/pacientes/cadastro` | cadastro da ficha inicial |
| `/pacientes/editar/{id-paciente}` | ficha preenchida em modo de edicao ou leitura |

A rota anterior `/cadastro/pacientes` redireciona para `/pacientes/cadastro` para preservar links existentes. O menu lateral aponta para `/pacientes`.

As tres telas do dominio usam o cabecalho visual comum das telas internas: breadcrumb, hero em gradiente, selo de contexto, titulo e descricao. O antigo card branco lateral de contexto foi removido de todos os cabecalhos padronizados. Na consulta, o botao `Cadastrar paciente` fica na barra da tabela. Na ficha, as acoes `Historico` e `Adicionar nota` ficam logo abaixo do cabecalho.

## Formulario cadastral

O contrato da ficha nao possui mais os campos `rg` e `whatsapp`. O telefone principal tambem e o contato usado para WhatsApp. Documentos antigos do MongoDB podem continuar contendo esses atributos durante a transicao, mas eles sao ignorados na leitura e nao voltam a ser persistidos nas atualizacoes.

Regras de interface:

- CPF usa a mascara `000.000.000-00` e validacao do formato;
- e-mail remove espacos, normaliza para minusculas e usa validacao de e-mail;
- telefone usa a mascara nacional de dez ou onze digitos;
- CEP e o primeiro campo do endereco e usa a mascara `00000-000`;
- rua, numero, complemento, bairro, cidade e estado iniciam desabilitados em um novo cadastro;
- uma consulta de CEP bem-sucedida preenche rua, bairro, cidade e estado e libera os campos do endereco para revisao e complemento manual;
- o numero permanece vazio e deve ser informado pelo usuario, pois nao faz parte do retorno de um CEP;
- CEP incompleto ou com formato invalido mantem os campos do endereco bloqueados ate que os oito digitos sejam informados;
- CEP inexistente ou falha da integracao libera rua, numero, complemento, bairro, cidade e estado para preenchimento manual, preservando uma mensagem que explica a indisponibilidade;
- ao editar uma ficha que ja possui endereco valido, os campos iniciam liberados para usuarios com permissao de edicao;
- condicoes medicas, alergias, medicamentos em uso, cirurgias anteriores e procedimentos esteticos anteriores sao campos multilinha;
- genero, estado, tipo de pele e exposicao solar usam o select customizado do Design System;
- habitos e consentimento usam o checkbox do Design System.

Ao desfocar um CEP completo, o MFE consulta somente a API interna:

- `GET /api/client/addresses/postal-code/{cep}`

A API normaliza o CEP para oito digitos, consulta prioritariamente a API Busca CEP oficial dos Correios e devolve `postalCode`, `street`, `complement`, `neighborhood`, `city` e `state`. Quando o token dos Correios nao estiver configurado, for rejeitado ou o provedor estiver indisponivel, o backend consulta o ViaCEP como contingencia. Rua, bairro, cidade e estado sao preenchidos automaticamente; o complemento retornado e apresentado quando existir e pode ser alterado pelo usuario. O numero do imovel nao integra o contrato da consulta e sempre depende de preenchimento manual.

Os campos dependentes usam o estado visual desabilitado do Design System enquanto aguardam um CEP completo ou o resultado da consulta. Esse estado deve ter fundo e borda acinzentados, texto atenuado e cursor de indisponibilidade para nao ser confundido com um campo editavel.

A integracao oficial exige contrato com os Correios e token Bearer. A configuracao e feita apenas no backend por `Correios__BaseUrl`, `Correios__AddressPathTemplate` e `Correios__BearerToken`. A contingencia usa `Correios__FallbackBaseUrl` e `Correios__FallbackAddressPathTemplate`, sem armazenar credenciais no frontend. A rota responde `503` somente quando nenhum provedor estiver disponivel; CEP invalido responde `400` e CEP inexistente responde `404`.

Depois que `POST /api/client/patients` responder com sucesso, o MFE publica um toast de confirmacao e navega para `/pacientes`. Falhas permanecem na ficha, preservam os dados preenchidos e usam toast de erro; conflito de CPF recebe mensagem especifica.

## Consulta

A tela principal exibe nome, CPF, telefone, e-mail, ultima atualizacao e acoes. A busca usa um unico termo, sem diferenciar maiusculas e minusculas, sobre:

- nome completo;
- telefone;
- e-mail.

Contrato:

- `GET /api/client/patients?search={termo}`

Sem `search`, a API devolve todos os pacientes ordenados por nome.

## Ficha e autorizacao

Contrato de leitura:

- `GET /api/client/patients/{id}`

Contrato de atualizacao integral:

- `PUT /api/client/patients/{id}`

Somente tokens com role `Administrador` podem executar o `PUT`. A interface desabilita os campos para os demais niveis, mas a API e a autoridade final e responde `403` para tentativa nao autorizada.

O `vyracare-api-authentication` inclui no JWT:

- claim de role derivada de `AccessLevel`;
- `access_level` com o mesmo valor para consumo do frontend;
- `job_role` com o cargo funcional.

Depois dessa alteracao, usuarios ja autenticados precisam entrar novamente para receber as novas claims.

## Notas profissionais

Qualquer funcionario autenticado pode registrar notas, inclusive quando nao puder alterar a ficha cadastral.

Contratos:

- `POST /api/client/patients/{id}/notes`;
- `GET /api/client/patients/{id}/notes`.

Entrada de criacao:

```json
{
  "content": "Evolucao observada apos o procedimento.",
  "procedureName": "Peeling"
}
```

A API acrescenta `id`, `authorId`, `authorName` e `createdAt` em UTC a partir do usuario autenticado e do relogio do backend. O historico e devolvido do mais recente para o mais antigo. Notas sao anexadas e nao substituem registros anteriores.

## Responsabilidades por repositorio

- `vyracare-app-user-mfe`: tabela, busca, cadastro, ficha, modal de nota e historico;
- `vyracare-api-client`: persistencia, busca, atualizacao autorizada e notas;
- `vyracare-api-authentication`: claims de autorizacao;
- `vyracare-app-shell`: rotas e navegacao lateral.
