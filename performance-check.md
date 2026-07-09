---
name: performance-check
description: Auditoria de performance para o ERP ITP. Use ao identificar lentidão, queries pesadas, renderizações excessivas ou problemas de carregamento. Cobre backend NestJS/TypeORM e frontend Next.js.
---

# Performance Check — ERP ITP

## Backend — Queries TypeORM

### Ativar Log de Queries (dev)

```typescript
// Adicionar no .env local
TYPEORM_LOGGING=true

// Ou no DataSource config
logging: ['query', 'slow'],
maxQueryExecutionTime: 1000,  // Log queries > 1s
```

### Detectar N+1

```typescript
// PROBLEMA: N+1 — 1 query pra lista + N queries pra cada relação
const alunos = await this.alunoRepo.find({ where: { ativo: true } });
for (const aluno of alunos) {
  const turma = await this.turmaRepo.findOne({ where: { id: aluno.turmaId } });
}

// SOLUÇÃO: JOIN em uma query
const alunos = await this.alunoRepo.find({
  where: { ativo: true },
  relations: ['turma'],
});

// Ou QueryBuilder para controle fino
const alunos = await this.alunoRepo
  .createQueryBuilder('aluno')
  .leftJoinAndSelect('aluno.turma', 'turma')
  .leftJoinAndSelect('turma.curso', 'curso')
  .where('aluno.ativo = :ativo', { ativo: true })
  .getMany();
```

### Paginação Obrigatória em Listagens

```typescript
// Sem paginação = carrega tudo (problema com muitos registros)
async findAll(page = 1, limit = 20) {
  const [data, total] = await this.repo.findAndCount({
    where: { ativo: true },
    skip: (page - 1) * limit,
    take: limit,
    order: { createdAt: 'DESC' },
  });

  return { data, total, page, limit };
}
```

### Índices Críticos

```sql
-- Verificar índices existentes no PostgreSQL
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'nome_tabela';

-- Índices que provavelmente faltam:
-- FKs sem índice
CREATE INDEX idx_alunos_turma ON alunos(turma_id);
CREATE INDEX idx_movimentacoes_conta ON movimentacoes(plano_contas_id);

-- Colunas de filtro frequente
CREATE INDEX idx_alunos_ativo ON alunos(ativo);
CREATE INDEX idx_boletos_status ON boletos(status, vencimento);
CREATE INDEX idx_movimentacoes_data ON movimentacoes(data);
```

### SELECT Seletivo (evitar SELECT *)

```typescript
// PROBLEMA: carrega todos os campos
const alunos = await this.repo.find();

// SOLUÇÃO: selecionar apenas o necessário
const alunos = await this.repo.find({
  select: ['id', 'nome', 'matricula', 'turmaId'],
  where: { ativo: true },
});
```

## Frontend — React / Next.js

### Evitar Re-renders Desnecessários

```tsx
// useMemo para cálculos pesados
const alunosFiltrados = useMemo(
  () => alunos.filter(a => a.nome.includes(search)),
  [alunos, search]
);

// useCallback para funções passadas como props
const handleDelete = useCallback(async (id: string) => {
  await axios.delete(`/backend-api/alunos/${id}`);
  refetch();
}, [refetch]);
```

### Debounce em Busca

```tsx
// Sem debounce: requisição a cada tecla digitada
useEffect(() => {
  const timer = setTimeout(() => {
    if (search.length >= 2) fetchAlunos(search);
  }, 300);
  return () => clearTimeout(timer);
}, [search]);
```

### Imagens e Uploads

```tsx
// Next.js Image component para otimização automática
import Image from 'next/image';

<Image
  src={fotoUrl}
  alt="Foto do aluno"
  width={100}
  height={100}
  loading="lazy"
/>
```

### Loading States / Skeleton

```tsx
// Sem loading state = tela em branco
if (loading) return <Skeleton className="h-64 w-full" />;
if (error) return <p>Erro ao carregar dados</p>;
```

## Vercel Serverless — Limites

| Recurso | Limite |
|---------|--------|
| Timeout da função `/api/main.ts` | 30s |
| Memória | 1024 MB |
| Cold start | ~2-3s (primeira requisição) |

```typescript
// Evitar operações longas em endpoints síncronos
// Dividir em jobs ou respostas paginadas se > 10s
```

## Checklist de Performance

### Backend
- [ ] Nenhuma listagem sem paginação
- [ ] Nenhum N+1 (verificar com logging ativo)
- [ ] FKs têm índice correspondente
- [ ] Queries lentas identificadas (> 1s)
- [ ] SELECT seletivo em queries críticas

### Frontend
- [ ] Busca de texto com debounce (≥ 300ms)
- [ ] Loading state em todas as chamadas assíncronas
- [ ] Listas longas com paginação ou virtualização
- [ ] Sem re-renders desnecessários em listas grandes
- [ ] Imagens via Next.js Image component

## Ferramentas

```bash
# Backend: ver queries sendo executadas
TYPEORM_LOGGING=true npm run start:dev

# Frontend: React DevTools Profiler
# Browser > devtools > Profiler tab

# Network: verificar tamanho das respostas
# Browser > devtools > Network > Size column
```
