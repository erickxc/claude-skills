---
name: security
description: Revisão de segurança para o ERP ITP com Snyk. Cobre análise de vulnerabilidades de dependências, OWASP Top 10, segurança JWT, proteção de dados LGPD e configurações de produção no Vercel/NestJS.
---

# Security Review — ERP ITP

## Snyk — Análise de Dependências

```bash
# Instalar CLI (se necessário)
npm install -g snyk

# Autenticar
snyk auth

# Analisar vulnerabilidades do backend
cd apps/backend && snyk test

# Analisar frontend
cd apps/frontend && snyk test

# Monitorar continuamente (registra no dashboard Snyk)
snyk monitor

# Ver apenas vulnerabilidades críticas/high
snyk test --severity-threshold=high

# Fix automático (quando disponível)
snyk fix

# Analisar código (SAST — não só dependências)
snyk code test
```

### Interpretar Resultados Snyk

| Severity | Ação |
|----------|------|
| **Critical** | Corrigir imediatamente — não deployar |
| **High** | Corrigir antes do próximo deploy |
| **Medium** | Planejar correção em dias |
| **Low** | Monitorar, corrigir na próxima janela |

```bash
# Atualizar dependência vulnerável
npm audit fix                    # Automático (patch-level)
npm install pacote@versao-segura # Manual para major

# Ver detalhes de uma vulnerabilidade específica
snyk test --json | jq '.vulnerabilities[] | select(.severity == "critical")'
```

## JWT / Auth

```typescript
// Verificar configurações no auth module

// ✅ Correto
JwtModule.register({
  secret: process.env.JWT_SECRET,  // Nunca hardcoded
  signOptions: { expiresIn: '8h' },
})

// ❌ Problemático
secret: 'minha-senha-123'  // Hardcoded = crítico

// Cookie httpOnly — verificar configuração
res.cookie('itp_token', token, {
  httpOnly: true,    // ✅ JS não consegue ler
  secure: true,      // ✅ Apenas HTTPS (verificar em prod)
  sameSite: 'strict', // ✅ Proteção CSRF
  maxAge: 8 * 60 * 60 * 1000,
})
```

## OWASP Top 10 — Verificações

### A01: Broken Access Control

```typescript
// Verificar: todo endpoint tem @Roles()?
// Buscar endpoints sem guard:

// ❌ Sem proteção
@Get('alunos')
findAll() {}

// ✅ Protegido
@Get('alunos')
@Roles('assist')
findAll() {}

// ❌ Acesso a recurso de outro usuário sem validação
async getAluno(id: string) {
  return this.repo.findOne({ where: { id } });  // Qualquer aluno!
}

// ✅ Validar ownership quando necessário
async getAluno(id: string, userId: string) {
  return this.repo.findOne({ where: { id, usuarioId: userId } });
}
```

### A02: Cryptographic Failures

```bash
# Verificar variáveis sensíveis
# JWT_SECRET deve ter >= 32 caracteres e ser randômico
# Nunca logar tokens ou senhas

# Checar se .env está no .gitignore
cat .gitignore | grep -E "\.env"

# Verificar se há secrets no git history
git log --all --full-history -- "**/.env*"
git grep -i "secret\|password\|token" -- "*.ts" "*.js"
```

### A03: Injection

```typescript
// SQL Injection via TypeORM

// ✅ Parâmetros seguros
this.repo.createQueryBuilder('aluno')
  .where('aluno.nome = :nome', { nome: userInput })  // Parametrizado
  .getMany();

// ❌ Concatenação direta — NUNCA
.where(`aluno.nome = '${userInput}'`)  // SQL Injection!

// Raw SQL — usar parâmetros
await queryRunner.query(
  'SELECT * FROM alunos WHERE nome = $1',
  [userInput]  // ✅ Parametrizado
);
```

### A05: Security Misconfiguration

```typescript
// Verificar CORS em produção
// apps/backend/src/main.ts

const allowedOrigins = process.env.NODE_ENV === 'production'
  ? ['https://itp.institutotiapretinha.org']  // ✅ Restrito em prod
  : ['http://localhost:3000', /^http:\/\/192\.168\./];  // Dev

app.enableCors({ origin: allowedOrigins, credentials: true });
```

### A07: Identification & Authentication Failures

```typescript
// Verificar: rate limiting em rotas de auth
// Verificar: senha com hash (bcrypt) — nunca plaintext
// Verificar: reset de senha com token de uso único

// Buscar uso incorreto de senhas
// grep -r "password" apps/backend/src --include="*.ts" | grep -v "hash\|bcrypt\|dto"
```

### A09: Security Logging Failures

```typescript
// ✅ Logar eventos de segurança
// Login bem-sucedido/falho
// Tentativas de acesso não autorizado
// Mudanças de role

// ❌ Nunca logar
console.log('Token:', jwtToken);
console.log('Senha:', password);
console.log('CPF:', cpf);
```

## LGPD — Dados Pessoais

```typescript
// Campos que são dados pessoais — exigem consentimento:
// CPF, RG, data de nascimento, endereço, email, telefone, foto

// Verificar:
// 1. Termo LGPD enviado e aceito antes de armazenar
// 2. Dados pessoais não expostos em logs
// 3. Relatórios com dados pessoais exigem role >= drt
// 4. Export de dados inclui apenas dados necessários
```

## Tokens Especiais (vercel.json)

```
⚠️ Atenção: tokens expostos no vercel.json
COLETOR_TOKEN = itp-coletor-2026
CHAMADA_TOKEN = itp-chamada-2026

Risco: qualquer pessoa com acesso ao repositório tem esses tokens.
Ação recomendada: mover para variáveis de ambiente privadas no Vercel dashboard.
```

## Checklist de Security Review

### Crítico (Blocker)
- [ ] Sem JWT_SECRET hardcoded no código
- [ ] Sem senhas em plaintext no banco
- [ ] Sem SQL concatenado com input do usuário
- [ ] Todos endpoints protegidos com `@Roles()` ou `@Public()` explícito

### Alto
- [ ] `snyk test` sem vulnerabilidades Critical/High
- [ ] Cookie JWT com `httpOnly: true` e `secure: true` em prod
- [ ] CORS restrito a domínios conhecidos em produção
- [ ] Dados pessoais (CPF, RG) não aparecem em logs

### Médio
- [ ] Tokens de COLETOR/CHAMADA movidos para env vars privadas no Vercel
- [ ] Rate limiting em endpoints de auth
- [ ] `snyk test` sem vulnerabilidades Medium
- [ ] Relatórios com dados pessoais exigem role >= drt

### Baixo
- [ ] Headers de segurança configurados (CSP, HSTS)
- [ ] Monitoramento Snyk ativo (`snyk monitor`)
- [ ] Auditoria de acessos logada

## Comandos Rápidos de Auditoria

```bash
# Vulnerabilidades em dependências
cd apps/backend && snyk test --severity-threshold=high
cd apps/frontend && snyk test --severity-threshold=high

# Buscar secrets no código
git grep -i "secret\|password\|apikey\|token" -- "*.ts" "*.tsx" | grep -v "\.env\|process\.env\|test\|spec"

# Endpoints sem @Roles (possíveis brechas)
grep -r "@Get\|@Post\|@Patch\|@Delete" apps/backend/src --include="*.ts" -A2 | grep -v "@Roles\|@Public"

# Verificar .env no gitignore
git check-ignore -v .env apps/backend/.env apps/frontend/.env.local
```
