---
name: git-commit
description: Padrão de mensagens de commit para o ERP ITP. Use ao criar commits ou revisar mensagens. Segue Conventional Commits com contexto do projeto.
---

# Git Commit — ERP ITP

## Formato

```
<tipo>(<escopo>): <descrição curta em português>

[corpo opcional — o "por quê", não o "o quê"]

[rodapé opcional — breaking changes, issues]
```

## Tipos

| Tipo | Quando usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `refactor` | Refatoração sem mudar comportamento |
| `style` | Formatação, sem mudança de lógica |
| `test` | Adicionar ou corrigir testes |
| `docs` | Documentação |
| `chore` | Tarefas de manutenção (deps, config) |
| `perf` | Melhoria de performance |
| `migration` | Migration de banco de dados |

## Escopos do Projeto

| Escopo | Módulo |
|--------|--------|
| `auth` | Autenticação/JWT |
| `matriculas` | Inscrições e matrículas |
| `academico` | Cursos, turmas, chamada, notas |
| `financeiro` | Plano de contas, boletos, movimentações |
| `estoque` | Produtos, movimentações, coletor |
| `funcionarios` | Gestão de funcionários |
| `usuarios` | Gestão de usuários |
| `relatorios` | Geração de relatórios |
| `notificacoes` | Sistema de notificações |
| `cadastro` | Dados mestre |
| `grupos` | Grupos de permissão |
| `frontend` | Mudanças gerais no frontend |
| `backend` | Mudanças gerais no backend |
| `deploy` | Vercel, configs de deploy |
| `deps` | Dependências |

## Exemplos

```bash
# Feature
feat(academico): adiciona controle de frequência mínima por turma

# Bug fix com contexto
fix(financeiro): corrige cálculo de desconto de faltas
O desconto era aplicado antes de considerar o isento de pagamento,
gerando valores negativos em alunos bolsistas.

# Migration
migration(matriculas): adiciona coluna responsavel_financeiro

# Refactor
refactor(auth): extrai lógica de validação JWT para helper

# Chore
chore(deps): atualiza TypeORM para 0.3.20

# Breaking change
feat(api)!: altera formato de resposta da listagem de alunos

BREAKING CHANGE: campo `alunos` renomeado para `data` no response
```

## Regras

- Descrição em **minúsculas** e **português**
- Máximo 72 caracteres na primeira linha
- Sem ponto final na descrição
- Corpo explica o **por quê**, não o **o quê** (o diff já mostra o quê)
- Commits atômicos: uma mudança lógica por commit

## Anti-Patterns

| Evitar | Motivo |
|--------|--------|
| `fix: corrigindo coisas` | Vago demais |
| `update` sem escopo | Sem contexto |
| `WIP` em main | Não committar trabalho incompleto |
| Commit gigante com tudo | Dificulta revisão e rollback |
| Mensagem em inglês misturada | Inconsistente com o projeto |

## Fluxo Rápido

```bash
git add apps/backend/src/financeiro/
git commit -m "fix(financeiro): corrige cálculo de juros em boletos vencidos"
```
