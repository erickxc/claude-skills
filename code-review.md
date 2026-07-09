---
name: code-review
description: Checklist de code review específico para o ERP ITP. Use ao revisar PRs ou antes de commitar. Verifica segurança JWT, soft delete, UUID, TypeORM, patterns NestJS e qualidade geral.
---

# Code Review — ERP ITP

Execute este checklist ao revisar código do projeto.

## Segurança

- [ ] Todos os endpoints têm `@Roles()` definido (nenhum endpoint sem guard)
- [ ] Rotas públicas usam `@Public()` explicitamente — não ausência de guard
- [ ] Sem JWT secret ou credenciais hardcoded no código
- [ ] Sem `console.log` com dados sensíveis (CPF, senha, token)
- [ ] Inputs validados com DTOs + class-validator antes de chegar ao service
- [ ] Uploads: extensão e tamanho validados no backend
- [ ] SQL raw usa parâmetros — nunca concatenação de string com input do usuário

## Banco de Dados

- [ ] PK é UUID (`@PrimaryGeneratedColumn('uuid')`)
- [ ] Soft delete: usa `ativo: boolean`, não DELETE físico
- [ ] Tem `createdAt` e `updatedAt` em todas as entidades
- [ ] `synchronize: false` — mudanças de schema via migration, não automático
- [ ] DECIMAL(10,2) para valores monetários — nunca FLOAT
- [ ] FK com estratégia `ON DELETE` definida
- [ ] Índice criado para FKs e colunas de filtro frequente

## TypeORM / NestJS

- [ ] `findOne` verifica se resultado é null e lança `NotFoundException`
- [ ] Sem `find()` sem `where: { ativo: true }` (não listar inativos)
- [ ] Módulo registrado em `AppModule`
- [ ] Entidade registrada em `TypeOrmModule.forFeature([...])`
- [ ] Sem lógica de negócio no controller

## Frontend

- [ ] Chamadas API via `/backend-api/` (proxy) — não direto ao backend
- [ ] Cores usando variáveis Tailwind (`bg-background`) — não hardcoded
- [ ] Loading state em operações assíncronas
- [ ] Erros da API exibidos via toast (sonner)
- [ ] Ações destrutivas confirmadas via AlertDialog
- [ ] Permissões verificadas com `usePermissions()`
- [ ] Sem `any` desnecessário no TypeScript

## Qualidade Geral

- [ ] Sem `TODO` ou `FIXME` esquecido
- [ ] Sem `console.log` de debug em produção
- [ ] Sem código comentado sem explicação
- [ ] Funções com responsabilidade única (< ~50 linhas)
- [ ] Nomes de variáveis/funções descritivos em português ou inglês consistente
- [ ] Sem dependência circular entre módulos

## Performance

- [ ] Sem N+1 queries (usar JOIN ou eager loading quando necessário)
- [ ] Paginação em listagens grandes
- [ ] Sem `SELECT *` em queries críticas

## Formato de Avaliação

Para cada problema encontrado:

```
**[Arquivo:Linha]**
- Problema: [o que está errado]
- Risco: [impacto se não corrigir]
- Correção: [como resolver]
- Prioridade: Blocker / Alta / Média / Baixa
```

## Prioridades

| Nível | Significado |
|-------|-------------|
| **Blocker** | Não pode mergear — segurança ou dado em risco |
| **Alta** | Deve corrigir antes do próximo deploy |
| **Média** | Corrigir em breve, não bloqueia |
| **Baixa** | Melhoria, pode ficar para depois |
