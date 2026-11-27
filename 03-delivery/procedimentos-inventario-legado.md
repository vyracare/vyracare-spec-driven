# Inventario de procedimentos legados

O isolamento de procedimentos exige `tenantId` em cada documento. Dados anteriores sem um identificador valido nao aparecem apos a promocao. Nao e permitido atribuir todos os registros a uma empresa por suposicao.

O repositorio `vyracare-api-proceedings` disponibiliza `scripts/inventory-legacy-proceedings.js`, executado pelo `mongosh` contra o banco de procedimentos. Ele apenas le a colecao `proceedings`. O relatorio identifica tenant ausente, nulo, vazio, composto so por espacos ou de tipo diferente de string, exibindo identificador, nome, codigo e tenant atual. Strings nao vazias tambem precisam ser conferidas independentemente: o script nao confirma que uma empresa existe nem detecta vinculacoes incorretas ja preenchidas.

1. Em ambiente controlado, confirme a base de destino e use uma credencial Mongo com permissao somente de leitura. Guarde o relatorio em local de acesso restrito.
2. Execute `mongosh <conexao-do-banco-de-procedimentos> --quiet --file scripts/inventory-legacy-proceedings.js`. Nao registre credenciais em repositorios ou logs.
3. Confirme a empresa proprietaria de cada item usando evidencia confiavel. O mapeamento opcional e um arquivo JSON externo ao repositorio, por exemplo `[{"id":"aaaaaaaaaaaaaaaaaaaaaaaa","tenantId":"identificador-confirmado"}]`. Cada `id` deve ser um ObjectId de 24 caracteres hexadecimais; o `tenantId` deve corresponder exatamente ao identificador emitido pela API de tenancy.
4. Para conferir uma proposta no PowerShell, defina `$env:MAPPING_FILE` para o caminho do arquivo, execute novamente o mesmo comando e remova a variavel apos o uso. O relatorio apresenta os mapeados e os nao mapeados. IDs repetidos, invalidos, inexistentes ou referentes a documentos nao identificados como legados interrompem a execucao. A ferramenta nao verifica a existencia dos tenants nem a evidencia da associacao.
5. Revise o resultado, inclusive IDs e campos malformados, e prepare separadamente backup, migracao por registro, verificacao posterior e rollback. O inventario nao aplica nem gera uma operacao de escrita. Nao promova a mudanca de isolamento ate que essa etapa esteja concluida e validada no ambiente de destino.

Os dados de exemplo usam identificadores ficticios. Nenhuma consulta a base real foi realizada na criacao deste procedimento.
