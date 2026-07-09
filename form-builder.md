---
name: form-builder
description: Padrões de formulário para o ERP ITP com react-hook-form + zod. Use ao criar formulários de cadastro, edição ou qualquer entrada de dados. Inclui validação, máscaras, upload e integração com API.
---

# Form Builder — ERP ITP

## Stack de Formulários

- `react-hook-form` — gerenciamento de estado
- `zod` — schema de validação
- Shadcn/UI — componentes de input
- `sonner` — feedback toast

## Template Completo

```tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import {
  Form, FormControl, FormField, FormItem, FormLabel, FormMessage
} from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { toast } from 'sonner';
import axios from '@/lib/axios';

// 1. Schema de validação
const formSchema = z.object({
  nome: z.string().min(2, 'Mínimo 2 caracteres').max(100),
  email: z.string().email('Email inválido'),
  telefone: z.string().optional(),
  dataNascimento: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, 'Data inválida'),
});

type FormValues = z.infer<typeof formSchema>;

// 2. Componente
export function MeuForm({ onSuccess }: { onSuccess?: () => void }) {
  const form = useForm<FormValues>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      nome: '',
      email: '',
      telefone: '',
    },
  });

  async function onSubmit(values: FormValues) {
    try {
      await axios.post('/backend-api/recurso', values);
      toast.success('Salvo com sucesso!');
      form.reset();
      onSuccess?.();
    } catch (err: any) {
      const msg = err.response?.data?.message || 'Erro ao salvar';
      toast.error(Array.isArray(msg) ? msg[0] : msg);
    }
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="nome"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nome</FormLabel>
              <FormControl>
                <Input placeholder="Nome completo" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={form.formState.isSubmitting}>
          {form.formState.isSubmitting ? 'Salvando...' : 'Salvar'}
        </Button>
      </form>
    </Form>
  );
}
```

## Formulário de Edição (pré-carregar dados)

```tsx
const form = useForm<FormValues>({
  resolver: zodResolver(formSchema),
  defaultValues: async () => {
    const { data } = await axios.get(`/backend-api/recurso/${id}`);
    return data;
  },
});
```

## Tipos de Campo

### Select

```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

<FormField
  control={form.control}
  name="status"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Status</FormLabel>
      <Select onValueChange={field.onChange} defaultValue={field.value}>
        <FormControl>
          <SelectTrigger>
            <SelectValue placeholder="Selecione..." />
          </SelectTrigger>
        </FormControl>
        <SelectContent>
          <SelectItem value="ativo">Ativo</SelectItem>
          <SelectItem value="inativo">Inativo</SelectItem>
        </SelectContent>
      </Select>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Checkbox

```tsx
import { Checkbox } from '@/components/ui/checkbox';

<FormField
  control={form.control}
  name="aceiteTermos"
  render={({ field }) => (
    <FormItem className="flex items-center space-x-2">
      <FormControl>
        <Checkbox checked={field.value} onCheckedChange={field.onChange} />
      </FormControl>
      <FormLabel>Aceito os termos</FormLabel>
      <FormMessage />
    </FormItem>
  )}
/>
```

### Textarea

```tsx
import { Textarea } from '@/components/ui/textarea';

<FormField
  control={form.control}
  name="observacao"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Observação</FormLabel>
      <FormControl>
        <Textarea rows={4} {...field} />
      </FormControl>
      <FormMessage />
    </FormItem>
  )}
/>
```

## Validações Zod Comuns

```typescript
// Obrigatório
z.string().min(1, 'Campo obrigatório')

// CPF (formato)
z.string().regex(/^\d{3}\.\d{3}\.\d{3}-\d{2}$/, 'CPF inválido')

// Telefone
z.string().regex(/^\(\d{2}\)\s\d{4,5}-\d{4}$/, 'Telefone inválido').optional()

// Valor monetário
z.number().min(0, 'Valor deve ser positivo')

// Data ISO
z.string().regex(/^\d{4}-\d{2}-\d{2}$/, 'Data inválida')

// Enum
z.enum(['ativo', 'inativo', 'pendente'])

// Array com mínimo
z.array(z.string()).min(1, 'Selecione ao menos um item')

// Condicional
z.object({
  tipo: z.enum(['pf', 'pj']),
  cpf: z.string().optional(),
  cnpj: z.string().optional(),
}).refine(
  (data) => data.tipo === 'pf' ? !!data.cpf : !!data.cnpj,
  { message: 'Documento obrigatório' }
)
```

## Checklist de Formulário

- [ ] Schema zod com mensagens em português
- [ ] `defaultValues` definidos (evita controlled/uncontrolled warning)
- [ ] Estado de submitting desabilita botão
- [ ] Erro da API exibido via toast
- [ ] `form.reset()` após sucesso (se criar novo)
- [ ] Campos obrigatórios marcados visualmente
