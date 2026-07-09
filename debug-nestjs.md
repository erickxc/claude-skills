---
name: debug-nestjs
description: Diagnóstico de erros comuns em NestJS e TypeORM no ERP ITP. Use quando encontrar erros de runtime, problemas de banco, falhas de auth ou comportamento inesperado no backend.
---

# Debug NestJS — ERP ITP

## Erros de Auth / JWT

### 401 Unauthorized

```
Causas:
1. Token expirado (expiração: 8h)
2. Cookie `itp_token` não enviado
3. Bearer token ausente no header
4. JWT_SECRET diferente entre envs

Diagnóstico:
- Verificar cookie nas devtools (Application > Cookies)
- Verificar header Authorization: Bearer <token>
- Conferir JWT_SECRET no .env vs Vercel
```

### 403 Forbidden

```
Causa: usuário autenticado mas role insuficiente

Diagnóstico:
- Verificar @Roles('X') no endpoint
- Verificar role do usuário no banco
- Hierarquia: user(0) < assist(2) < prof(4) < drt(8) < prt(10)
```

## Erros TypeORM

### `QueryFailedError: null value in column violates not-null constraint`

```
Causa: novo campo NOT NULL sem valor default e sem backfill

Solução:
1. Adicionar como nullable primeiro
2. Fazer backfill dos dados existentes
3. Depois tornar NOT NULL em migration separada
```

### `EntityMetadataNotFound` ou `Repository not found`

```
Causa: entidade não registrada no módulo

Solução:
@Module({
  imports: [TypeOrmModule.forFeature([MinhaEntidade])],  // ← verificar
})
```

### `QueryRunner is already released`

```
Causa: usar queryRunner após commit/rollback

Solução: sempre usar try/finally para liberar
try {
  await queryRunner.startTransaction();
  // operações
  await queryRunner.commitTransaction();
} catch {
  await queryRunner.rollbackTransaction();
} finally {
  await queryRunner.release();  // ← sempre liberar
}
```

### N+1 Queries

```
Sintoma: muitas queries idênticas no log para cada item de uma lista

Diagnóstico: ativar log de queries
// ormconfig / dataSource
logging: true,

Solução: usar JOIN explícito
this.repo.find({
  relations: ['turma', 'turma.curso'],
  where: { ativo: true }
})

// Ou QueryBuilder
this.repo.createQueryBuilder('aluno')
  .leftJoinAndSelect('aluno.turma', 'turma')
  .where('aluno.ativo = :ativo', { ativo: true })
  .getMany();
```

## Erros de Migration

### Migration não encontrada / não roda

```bash
# Verificar migrations pendentes
npm run typeorm:migration:show

# Forçar rerun (cuidado em prod!)
npm run typeorm:migration:revert
npm run typeorm:migration:run
```

### `relation already exists`

```
Causa: migration executou parcialmente e ficou em estado inconsistente

Solução:
1. Verificar tabela __migrations no banco
2. Remover entrada da migration com problema
3. Corrigir migration e rodar novamente
```

## Erros de Módulo / DI

### `Nest can't resolve dependencies`

```
Causa: provider não injetado ou módulo não importado

Diagnóstico: ler a mensagem completa — ela mostra qual provider está faltando

Solução:
1. Verificar se o serviço está em providers[] do módulo
2. Se vem de outro módulo, verificar exports[] lá e imports[] aqui
```

### `Cannot read properties of undefined (reading 'X')` em service

```
Causa 1: @InjectRepository() sem TypeOrmModule.forFeature([Entidade])
Causa 2: Circular dependency entre módulos

Solução circular:
@Inject(forwardRef(() => OutroServico))
private outroServico: OutroServico
```

## Erros de Validação

### `ValidationPipe` não está funcionando

```
Verificar se main.ts tem:
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  transform: true,
}));

E se o DTO tem os decorators do class-validator.
```

## Erros de CORS (local)

```
Causa: frontend na 3000 chamando backend 3001 sem proxy

Solução: usar sempre /backend-api/ no frontend
→ Next.js proxy redireciona para o backend

Em dev, o CORS aceita localhost:3000 e IPs 192.168.x.x
```

## Ferramentas de Debug

```bash
# Log detalhado do TypeORM (ver queries geradas)
# No .env local:
TYPEORM_LOGGING=true

# Testar endpoint sem frontend
curl -X GET http://localhost:3001/api/alunos \
  -H "Authorization: Bearer <token>"

# Ver estrutura da tabela no Neon
# Dashboard: console.neon.tech
```

## Checklist de Debug

1. Ler a mensagem de erro **completa** (stack trace)
2. Verificar se é erro de auth, validação, banco ou lógica
3. Reproduzir em dev local antes de investigar em prod
4. Verificar variáveis de ambiente (.env vs Vercel dashboard)
5. Verificar migrations rodaram (`typeorm:migration:show`)
