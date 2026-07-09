---
name: component-review
description: Revisão de componentes Next.js/React para o ERP ITP. Use ao criar ou revisar componentes, páginas, layouts. Verifica consistência com Shadcn/UI, Tailwind, padrões do projeto e acessibilidade.
---

# Component Review — ERP ITP

## Stack Frontend

- Next.js 15.5 (App Router) + React 19
- Shadcn/UI + Tailwind CSS
- Porta: 3000
- Auth via `useAuth()` hook

## Estrutura de Componente

```tsx
// Componente de página (app/[rota]/page.tsx)
export default function MinhaPage() {
  return <div>...</div>;
}

// Componente reutilizável (components/[nome].tsx)
interface Props {
  titulo: string;
  onSalvar: () => void;
}

export function MeuComponente({ titulo, onSalvar }: Props) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{titulo}</CardTitle>
      </CardHeader>
      <CardContent>...</CardContent>
    </Card>
  );
}
```

## Auth e Permissões

```tsx
import { useAuth } from '@/context/auth-context';
import { usePermissions } from '@/hooks/use-permissions';

export function MinhaPage() {
  const { user } = useAuth();
  const { canWrite, canRead } = usePermissions(user?.role);

  if (!canRead) return <p>Sem permissão</p>;

  return (
    <div>
      {canWrite && <Button>Novo</Button>}
    </div>
  );
}
```

## Integração com API

```tsx
// Usar proxy Next.js — NUNCA chamar backend direto
// /backend-api/* → /api/* no backend

import axios from '@/lib/axios';  // Interceptor JWT já configurado

// Buscar dados
const { data } = await axios.get('/backend-api/alunos');

// Mutations com estado de loading
const [loading, setLoading] = useState(false);

async function handleSalvar() {
  setLoading(true);
  try {
    await axios.post('/backend-api/alunos', formData);
    toast.success('Salvo com sucesso');
  } catch (err) {
    toast.error('Erro ao salvar');
  } finally {
    setLoading(false);
  }
}
```

## Toasts

```tsx
import { toast } from 'sonner';

toast.success('Operação realizada');
toast.error('Erro ao processar');
toast.warning('Atenção: ...');
toast.loading('Processando...');
```

## Padrões Shadcn/UI

```tsx
// Tabela
<Table>
  <TableHeader>
    <TableRow>
      <TableHead>Nome</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    {items.map(item => (
      <TableRow key={item.id}>
        <TableCell>{item.nome}</TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>

// Dialog de confirmação
<AlertDialog>
  <AlertDialogTrigger asChild>
    <Button variant="destructive">Excluir</Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Confirmar exclusão?</AlertDialogTitle>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancelar</AlertDialogCancel>
      <AlertDialogAction onClick={handleDelete}>Confirmar</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>

// Badge de status
<Badge variant={ativo ? 'default' : 'secondary'}>
  {ativo ? 'Ativo' : 'Inativo'}
</Badge>
```

## Tema Escuro

```tsx
// Usar variáveis CSS do Tailwind — não hardcodar cores
className="bg-background text-foreground"
className="bg-card text-card-foreground"
className="border-border"

// Nunca:
className="bg-white text-black"  // Quebra no dark mode
```

## Loading States

```tsx
// Skeleton enquanto carrega
if (loading) return (
  <div className="space-y-3">
    <Skeleton className="h-10 w-full" />
    <Skeleton className="h-10 w-full" />
    <Skeleton className="h-10 w-3/4" />
  </div>
);

// Botão com loading
<Button disabled={loading} onClick={handleSalvar}>
  {loading ? (
    <><Loader2 className="mr-2 h-4 w-4 animate-spin" /> Salvando...</>
  ) : 'Salvar'}
</Button>
```

## Checklist de Revisão

- [ ] Props tipadas com interface TypeScript
- [ ] Estados de loading e erro tratados
- [ ] Toasts para feedback de ação (sucesso/erro)
- [ ] Usa variáveis CSS do Tailwind (compatível com dark mode)
- [ ] Permissões verificadas via `usePermissions()`
- [ ] Chamadas API via proxy `/backend-api/` (não direto ao backend)
- [ ] Confirmação antes de ações destrutivas (AlertDialog)
- [ ] Sem `console.log` esquecido em produção

## Anti-Patterns

| Evitar | Por quê | Em vez disso |
|--------|---------|--------------|
| `fetch()` direto para o backend | CORS em prod | Usar proxy `/backend-api/` |
| Cores hardcoded (`bg-white`) | Quebra dark mode | Variáveis Tailwind (`bg-background`) |
| Sem loading state | UX ruim | Skeleton ou spinner |
| Deletar sem confirmar | Acidente irreversível | AlertDialog |
| `any` no TypeScript | Perde type safety | Tipar corretamente |
