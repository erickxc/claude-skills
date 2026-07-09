# Claude Code Skills

Skills (slash commands) para o Claude Code com ativação automática por contexto.

## Instalação

```bash
git clone https://github.com/erickxc/claude-skills.git temp-skills
# Linux/macOS
cp temp-skills/*.md ~/.claude/commands/
cp temp-skills/CLAUDE.md ~/.claude/CLAUDE.md

# Windows (PowerShell)
Copy-Item temp-skills\*.md -Destination "$env:USERPROFILE\.claude\commands\"
Copy-Item temp-skills\CLAUDE.md -Destination "$env:USERPROFILE\.claude\CLAUDE.md"

rm -rf temp-skills
```

> O `CLAUDE.md` define quando cada skill dispara automaticamente. Sem ele, as skills funcionam apenas via `/nome-da-skill` no prompt.

---

## Skills Disponíveis

### `brainstorming`
Planejamento colaborativo antes de qualquer implementação. Força o fluxo design-first com aprovação antes de codar.

**Ativar automaticamente quando:** "quero criar", "planejar", "como implementar", "ideia para", "pensar em"

**Invocar manualmente:** `/brainstorming`

**Exemplos de prompt:**
```
Quero criar um sistema de matrícula de alunos
Vamos planejar o módulo de relatórios antes de codar
Me ajuda a pensar na arquitetura do financeiro
```

---

### `ui-ux-designer`
Crítica de design e UX com embasamento em pesquisa (Nielsen Norman Group). Evita estética genérica, cita fontes para cada recomendação.

**Ativar automaticamente quando:** "o que acha", "ficou bom", "melhorar o design", "revisar interface", "acessibilidade", "como ficou"

**Invocar manualmente:** `/ui-ux-designer`

**Exemplos de prompt:**
```
O que acha do layout desta página de cadastro?
Ficou bom esse componente de tabela?
Como posso melhorar a acessibilidade deste formulário?
```

---

### `web-design-guidelines`
Revisão de código de UI contra diretrizes de boas práticas web. Valida acessibilidade, HTML semântico e padrões de usabilidade.

**Ativar automaticamente quando:** "revisar meu UI", "checar acessibilidade", "auditar design", "verificar boas práticas"

**Invocar manualmente:** `/web-design-guidelines`

**Exemplos de prompt:**
```
Revisa meu componente de formulário contra boas práticas
Checa acessibilidade nessa página de listagem
Audita meu layout para HTML semântico
```

---

### `database-schema-designer`
Design de schemas SQL/NoSQL com normalização, indexação, constraints e estratégias de migração. Focado em performance e escalabilidade.

**Ativar automaticamente quando:** "criar tabela", "modelar banco", "schema para", "estrutura do banco", "preciso de um banco"

**Invocar manualmente:** `/database-schema-designer`

**Exemplos de prompt:**
```
Cria um schema para rastrear matrículas de alunos
Preciso de um banco para o módulo financeiro
Normaliza esse modelo de dados de alunos
```

---

### `nestjs-patterns`
Padrões NestJS 11 + TypeORM. Cobre estrutura de módulos, JWT auth, hierarquia de roles, soft deletes, DTOs, migrations e anti-patterns.

**Ativar automaticamente quando:** "criar módulo", "novo endpoint", "nova entidade", "criar service", "criar guard"

**Invocar manualmente:** `/nestjs-patterns`

**Exemplos de prompt:**
```
Cria um módulo para gestão de cursos
Qual o padrão para adicionar @Roles() em endpoint protegido?
Como estruturo a entity com soft delete?
```

---

### `typeorm-migration`
Workflow completo de migrations TypeORM. Cobre comandos, templates manuais, estratégias zero-downtime e tratamento de erros comuns.

**Ativar automaticamente quando:** "migration", "alterar tabela", "adicionar coluna", "nova coluna", "mudar schema"

**Invocar manualmente:** `/typeorm-migration`

**Exemplos de prompt:**
```
Gera uma migration para adicionar coluna de status do aluno
Como renomeio uma coluna sem downtime?
Cria migration para um relacionamento de chave estrangeira
```

---

### `api-design`
Padrões REST: URL conventions, filtros/paginação, DTOs com validação, response standards, auth/roles, error handling e uploads.

**Ativar automaticamente quando:** "criar endpoint", "nova rota", "DTO", "validação da API", "estrutura da API"

**Invocar manualmente:** `/api-design`

**Exemplos de prompt:**
```
Cria endpoint para listar alunos com filtros e paginação
Qual o padrão de validação do DTO para campos de email?
Como tratar upload de arquivo na API?
```

---

### `component-review`
Revisão de componentes React/Next.js. Valida consistência com Shadcn/UI, padrões Tailwind, dark mode, estados de loading, permissões e integração com API.

**Ativar automaticamente quando:** "criar componente", "revisar componente", "nova página", "novo componente"

**Invocar manualmente:** `/component-review`

**Exemplos de prompt:**
```
Revisa meu componente de listagem de alunos
Esse formulário segue o padrão do projeto?
Verifica se está usando as variáveis Tailwind corretas para dark mode
```

---

### `form-builder`
Padrões de formulário com react-hook-form + zod. Cobre templates, tipos de campo, validações customizadas, tratamento de erros e submissão para API.

**Ativar automaticamente quando:** "criar formulário", "novo form", "campo de input", "validação de formulário"

**Invocar manualmente:** `/form-builder`

**Exemplos de prompt:**
```
Cria um formulário de cadastro de aluno com validação de CPF
Como adicionar validação condicional — documentos diferentes para menores?
Configura submissão do form com tratamento de erro e toast
```

---

### `matricula-flow`
Regras de negócio do módulo de matrícula: numeração (ITP-ROLE-YYYYMM-###), fluxo de inscrição, LGPD, validação de documentos por idade, transições de status e roles de acesso.

**Ativar automaticamente quando:** "matrícula", "inscrição", "LGPD", "termo", "efetivar matrícula"

**Invocar manualmente:** `/matricula-flow`

**Exemplos de prompt:**
```
Como funciona o fluxo de consentimento LGPD para menores?
Quais documentos são obrigatórios para matrícula de aluno com 16 anos?
Implementa a lógica de geração do número de matrícula para março/2026
```

---

### `financeiro-rules`
Regras de negócio do módulo financeiro: plano de contas, transações, boletos, métodos de pagamento, desconto por faltas, isenções, roles e tipos de dados monetários.

**Ativar automaticamente quando:** "boleto", "financeiro", "plano de contas", "movimentação", "desconto", "isenção"

**Invocar manualmente:** `/financeiro-rules`

**Exemplos de prompt:**
```
Calcula pagamento do aluno com desconto por faltas verificando isenção
Qual tipo de dado correto para armazenar valores de boleto?
Cria boleto com alertas de vencimento em 3 dias, no dia e em atraso
```

---

### `relatorios`
Geração de relatórios: jsPDF + html2canvas para PDF, XLSX para Excel, QR codes, templates por módulo e visibilidade por role.

**Ativar automaticamente quando:** "relatório", "exportar PDF", "exportar Excel", "gerar PDF", "imprimir"

**Invocar manualmente:** `/relatorios`

**Exemplos de prompt:**
```
Gera PDF com lista de alunos matriculados e logo da escola
Exporta lista de alunos para Excel com colunas formatadas
Cria relatório financeiro para o diretor com resumo orçamentário
```

---

### `code-review`
Checklist completo de code review. Cobre segurança (@Roles, sem secrets hardcoded, validação), banco (UUID/soft delete/migrations), TypeORM, frontend e performance.

**Ativar automaticamente quando:** "revisar código", "code review", "antes de commitar", "checar qualidade", "revisar PR"

**Invocar manualmente:** `/code-review`

**Exemplos de prompt:**
```
Revisa esse endpoint — tem os guards de role corretos?
Checa meu service contra o checklist do projeto
Esse PR tem mudanças no banco — valida o padrão de migration
```

---

### `debug-nestjs`
Diagnóstico de erros NestJS/TypeORM. Cobre 401/403, falhas TypeORM (null constraint, circular deps, N+1), problemas de migration, validation pipe e CORS.

**Ativar automaticamente quando:** "erro no backend", "não está funcionando", "bug no NestJS", "erro TypeORM", "401", "403", "query falhando"

**Invocar manualmente:** `/debug-nestjs`

**Exemplos de prompt:**
```
Recebendo 401 Unauthorized — como debugo o token?
TypeORM: null value violates not-null constraint. O que aconteceu?
Minha entity não está sendo encontrada — problema de injeção de dependência?
```

---

### `performance-check`
Auditoria de performance backend/frontend. Detecção de N+1, paginação, indexes, query logging, re-renders React, debouncing, skeleton loaders e limites do Vercel serverless.

**Ativar automaticamente quando:** "lento", "performance", "otimizar", "N+1", "query pesada", "carregando devagar"

**Invocar manualmente:** `/performance-check`

**Exemplos de prompt:**
```
Esse endpoint de listagem está lento — provavelmente N+1?
Como detectar problemas de performance nos componentes React?
Adiciona indexes para acelerar busca de alunos e matrículas
```

---

### `security`
Revisão de segurança com Snyk. Cobre OWASP Top 10, JWT, LGPD, dados sensíveis em logs, configuração de tokens e gestão de credenciais.

**Ativar automaticamente quando:** "segurança", "Snyk", "vulnerabilidade", "JWT", "LGPD", "dados sensíveis", "brecha"

**Invocar manualmente:** `/security`

**Exemplos de prompt:**
```
Roda Snyk para checar vulnerabilidades nas dependências
Minha configuração JWT está segura — httpOnly cookie, secure flag?
Revisa o código para conformidade LGPD — CPF/RG aparecem em logs?
```

---

### `cost-reducer`
Otimização de custos de infraestrutura. Cobre bundle, queries caras, pipelines de imagem, cache, N+1, dimensionamento de instâncias e gastos no Vercel.

**Ativar automaticamente quando:** "custo", "reduzir gasto", "otimizar infra", "caro", "billing", "Vercel spend"

**Invocar manualmente:** `/cost-reducer`

**Exemplos de prompt:**
```
Essa query roda 1000x por segundo — como reduzir custo no banco?
Qual a forma mais barata de cachear dados de matrícula?
Revisa meu pipeline de imagens — está desperdiçando banda?
```

---

### `git-commit`
Padrão Conventional Commits. Cobre tipos (feat/fix/refactor/style/test/docs/chore/perf/migration), escopos do projeto, descrições em português e commits atômicos.

**Ativar automaticamente quando:** "commitar", "mensagem de commit", "git commit", "escrever commit"

**Invocar manualmente:** `/git-commit`

**Exemplos de prompt:**
```
Qual o formato do commit para adicionar validação de role?
Escreve a mensagem de commit para esse fix no cálculo financeiro
Revisa: essa mensagem de commit segue o padrão do projeto?
```

---

### `caveman`
Modo de comunicação ultra-comprimido. Reduz ~75% dos tokens mantendo precisão técnica. Suporta intensidades: `lite`, `full` (padrão), `ultra`.

**Ativar automaticamente quando:** "caveman mode", "less tokens", "seja breve", "menos tokens"

**Invocar manualmente:** `/caveman`, `/caveman lite`, `/caveman ultra`

**Exemplos de prompt:**
```
Caveman mode — explica esse erro do NestJS
/caveman lite — qual o padrão de DTO?
Less tokens — corrige esse bug
```

---

## Estrutura dos arquivos

```
~/.claude/
├── CLAUDE.md              # Regras globais + gatilhos automáticos das skills
└── commands/
    ├── api-design.md
    ├── brainstorming.md
    ├── caveman.md
    ├── code-review.md
    ├── component-review.md
    ├── cost-reducer.md
    ├── database-schema-designer.md
    ├── debug-nestjs.md
    ├── financeiro-rules.md
    ├── form-builder.md
    ├── git-commit.md
    ├── matricula-flow.md
    ├── nestjs-patterns.md
    ├── performance-check.md
    ├── relatorios.md
    ├── security.md
    ├── typeorm-migration.md
    ├── ui-ux-designer.md
    └── web-design-guidelines.md
```

## Como funciona

Cada arquivo `.md` em `~/.claude/commands/` vira um slash command disponível no Claude Code.

O `CLAUDE.md` configura gatilhos automáticos — o Claude lê o contexto da mensagem e ativa a skill correspondente sem precisar invocar manualmente.

> Skills com contexto específico do ERP ITP (matricula-flow, financeiro-rules, nestjs-patterns, etc.) podem precisar de adaptação para outros projetos.
