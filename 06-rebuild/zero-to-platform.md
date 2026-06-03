# Visao Zero to Platform

Este documento existe para responder uma pergunta direta:

"Se a organizacao Vyracare precisasse ser recriada do zero, qual e a sequencia minima para voltar ao estado atual?"

## Resultado alvo

Ao final da reconstrucao, o ecossistema esperado deve conter:

- perfil da organizacao publicado em `.github`
- repositorio `vyracare-spec-driven` como fonte central de especificacao
- reusable workflows Angular e .NET
- templates Angular e .NET prontos para scaffolding
- design system publicado e consumivel
- shell publicado em `dev`, `hml` e `prod`
- MFEs publicados em `dev`, `hml` e `prod`
- APIs .NET publicadas em `dev`, `hml` e `prod`
- Mongo segregado por ambiente
- secrets segregados por ambiente
- Swagger exposto nas APIs publicadas

## Pre-requisitos minimos

Antes de criar qualquer repositorio, a organizacao precisa ter:

- conta AWS operacional
- conta MongoDB Atlas operacional
- GitHub organization com administracao disponivel
- CodeArtifact funcional para o design system
- usuario ou role com permissao para:
  - S3
  - CloudFront
  - Lambda
  - API Gateway
  - IAM
  - Cognito
  - Secrets Manager

## Ordem recomendada

### Fase 1. Fundacao da organizacao

1. criar `.github`
2. publicar o `README` do perfil da organizacao
3. criar `vyracare-spec-driven`
4. registrar neste repositorio a estrategia de ambientes, naming e pipelines

### Fase 2. Infra de automacao

5. criar `vyracare-infra-pipes-angular`
6. criar `vyracare-infra-pipes-dot-net`
7. validar reusable workflows com `PAT_TOKEN`

### Fase 3. Templates

8. criar `templates-angular`
9. criar `template-dot-net-api`
10. validar os scripts de rename e os workflows gerados

### Fase 4. Base compartilhada de frontend

11. criar `vyracare-design-system`
12. publicar playground e Storybook
13. publicar o pacote no CodeArtifact

### Fase 5. Aplicacoes frontend

14. criar `vyracare-app-shell`
15. criar os MFEs:
  - `vyracare-app-user-mfe`
  - `vyracare-app-profile-mfe`
  - `vyracare-app-dashboard-mfe`
  - `vyracare-app-proceedings-mfe`
16. validar `remoteEntry.js` e integracao com o shell

### Fase 6. Aplicacoes backend

17. criar `vyracare-api-authentication`
18. criar `vyracare-api-client`
19. criar `vyracare-api-proceedings`
20. validar pipelines e provisionamento `dev`, `hml` e `prod`

### Fase 7. Dados e seguranca

21. criar usuario de banco e databases:
  - `vyracare_db_dev`
  - `vyracare_db_hml`
  - `vyracare_db`
22. criar secrets por ambiente
23. aplicar rotacao quando houver secrets legados

### Fase 8. Validacao de plataforma

24. validar shell publicado
25. validar remote entries dos MFEs
26. validar APIs e Swagger
27. validar login, cadastro e leitura de dados

## Regra de ouro

Nao comece pelos repositorios de produto.

Se a base nao estiver pronta:

- os templates geram drift
- as esteiras falham por falta de contrato
- os ambientes nao se alinham

O caminho correto e sempre:

- fundacao
- reusable workflows
- templates
- base compartilhada
- produto
