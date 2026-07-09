# Instruções Globais — Claude Code

## Skills Sempre Ativas (Plantão)

Estas regras estão **sempre em vigor**, em toda resposta, sem exceção:

### Modo Econômico de Tokens (caveman/full)

Responda de forma compacta. Estas regras se aplicam a **todas** as respostas:

- Eliminar: artigos desnecessários, palavras de preenchimento (basicamente, simplesmente, realmente), saudações, afirmações ("Claro!", "Ótimo!", "Com certeza!")
- Fragmentos são permitidos. Sinônimos curtos preferidos.
- Código: nunca abreviar — escrever normal.
- Padrão: `[coisa] [ação] [motivo]. [próximo passo].`
- **Desligar apenas se:** usuário pedir explicitamente "modo normal" ou "resposta completa"
- **Exceções automáticas:** avisos de segurança, ações irreversíveis, sequências com risco de ambiguidade

---

## Skills Automáticas por Contexto

Leia as skills em `~/.claude/commands/` e aplique-as automaticamente baseado no contexto, sem esperar invocação explícita. As regras abaixo definem quando cada skill deve ser ativada:

---

### `/ui-ux-designer`
**Ativar quando:**
- Usuário pede feedback, revisão ou opinião sobre UI, UX, design, interface, layout, cores, tipografia
- Usuário compartilha screenshot, código de componente visual ou página e pede análise
- Palavras-chave: "o que acha", "ficou bom", "melhorar o design", "revisar interface", "acessibilidade", "como ficou"

---

### `/web-design-guidelines`
**Ativar quando:**
- Usuário pede para revisar código de UI contra boas práticas
- Palavras-chave: "revisar meu UI", "checar acessibilidade", "auditar design", "verificar boas práticas"

---

### `/database-schema-designer`
**Ativar quando:**
- Usuário pede para criar, modelar ou revisar schema de banco de dados
- Palavras-chave: "criar tabela", "modelar banco", "schema para", "estrutura do banco", "preciso de um banco"

---

### `/nestjs-patterns`
**Ativar quando:**
- Usuário vai criar ou está criando módulo, controller, service, entity, guard, decorator no NestJS
- Palavras-chave: "criar módulo", "novo endpoint", "nova entidade", "criar service", "criar guard"

---

### `/typeorm-migration`
**Ativar quando:**
- Usuário vai criar, alterar ou discutir migrations no TypeORM
- Palavras-chave: "migration", "alterar tabela", "adicionar coluna", "nova coluna", "mudar schema"

---

### `/api-design`
**Ativar quando:**
- Usuário vai criar novos endpoints REST, DTOs ou discutir estrutura de API
- Palavras-chave: "criar endpoint", "nova rota", "DTO", "validação da API", "estrutura da API"

---

### `/component-review`
**Ativar quando:**
- Usuário vai criar ou revisar componente React/Next.js
- Palavras-chave: "criar componente", "revisar componente", "nova página", "novo componente"

---

### `/form-builder`
**Ativar quando:**
- Usuário vai criar formulário com inputs, validação ou submissão de dados
- Palavras-chave: "criar formulário", "novo form", "campo de input", "validação de formulário"

---

### `/matricula-flow`
**Ativar quando:**
- Usuário trabalha com matrícula, inscrição, LGPD, termo, número de matrícula
- Palavras-chave: "matrícula", "inscrição", "LGPD", "termo", "efetivar matrícula"

---

### `/financeiro-rules`
**Ativar quando:**
- Usuário trabalha com boletos, plano de contas, movimentações financeiras, desconto, isenção
- Palavras-chave: "boleto", "financeiro", "plano de contas", "movimentação", "desconto", "isenção"

---

### `/relatorios`
**Ativar quando:**
- Usuário vai criar ou trabalhar com exportação de PDF, Excel ou relatórios
- Palavras-chave: "relatório", "exportar PDF", "exportar Excel", "gerar PDF", "imprimir"

---

### `/code-review`
**Ativar quando:**
- Usuário pede revisão de código, PR review ou quer checar qualidade antes de commitar
- Palavras-chave: "revisar código", "code review", "antes de commitar", "checar qualidade", "revisar PR"

---

### `/debug-nestjs`
**Ativar quando:**
- Usuário reporta erro no backend NestJS, TypeORM, auth ou banco de dados
- Palavras-chave: "erro no backend", "não está funcionando", "bug no NestJS", "erro TypeORM", "401", "403", "query falhando"

---

### `/performance-check`
**Ativar quando:**
- Usuário reporta lentidão, quer otimizar queries, reduzir re-renders ou melhorar performance
- Palavras-chave: "lento", "performance", "otimizar", "N+1", "query pesada", "carregando devagar"

---

### `/git-commit`
**Ativar quando:**
- Usuário pede para criar mensagem de commit ou está prestes a commitar
- Palavras-chave: "commitar", "mensagem de commit", "git commit", "escrever commit"

---

### `/security`
**Ativar quando:**
- Usuário pede revisão de segurança, menciona Snyk, vulnerabilidades, JWT, LGPD ou dados sensíveis
- Palavras-chave: "segurança", "Snyk", "vulnerabilidade", "JWT", "LGPD", "dados sensíveis", "brecha"

---

### `/cost-reducer`
**Ativar quando:**
- Usuário quer reduzir custos de infraestrutura, otimizar queries caras, revisar uso do Vercel/banco
- Palavras-chave: "custo", "reduzir gasto", "otimizar infra", "caro", "billing", "Vercel spend"

---

### `/brainstorming`
**Ativar quando:**
- Usuário quer planejar uma nova feature, componente ou funcionalidade antes de implementar
- Palavras-chave: "quero criar", "pensar em", "planejar", "como implementar", "ideia para"

---

### `/caveman`
**Sempre ativo** (regras já embutidas na seção "Skills Sempre Ativas" acima).
Usar `/caveman lite`, `/caveman ultra` para mudar intensidade.

@RTK.md
