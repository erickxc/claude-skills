---
name: financeiro-rules
description: Regras de negócio do módulo financeiro do ERP ITP. Use ao trabalhar com plano de contas, movimentações, boletos, formas de pagamento, desconto de faltas ou qualquer lógica financeira.
---

# Financeiro Rules — ERP ITP

## Módulo

`financeiro` — plano de contas, movimentações, formas de pagamento, boletos

## Entidades Principais

| Entidade | Responsabilidade |
|----------|-----------------|
| `PlanoContas` | Categorias de receita/despesa |
| `Movimentacao` | Lançamento financeiro (entrada/saída) |
| `FormaPagamento` | Dinheiro, PIX, cartão, boleto, etc. |
| `Boleto` | Cobrança gerada para aluno/responsável |
| `ContaBancaria` | Contas da instituição (em `cadastro`) |

## Regras de Desconto por Faltas

```typescript
// Desconto calculado APÓS verificar isenção
// Ordem obrigatória:
// 1. Verificar se aluno é isento de pagamento
// 2. Se isento → valor = 0, ignorar faltas
// 3. Se não isento → calcular desconto por faltas

function calcularValorFinal(aluno, faltas, valorBase) {
  if (aluno.isentoPagemento) return 0;

  const percentualDesconto = calcularDescontoFaltas(faltas);
  return valorBase * (1 - percentualDesconto);
}
```

**Nunca aplicar desconto de faltas antes de verificar isenção** — gera valores negativos.

## Boletos

```typescript
// Campos obrigatórios de boleto
{
  descricao: string,    // Título do boleto (exibido ao aluno)
  valor: decimal,       // DECIMAL(10,2) — nunca FLOAT
  vencimento: date,
  alunoId: uuid,
  status: 'pendente' | 'pago' | 'vencido' | 'cancelado'
}
```

Alertas de vencimento:
- **3 dias antes**: notificação amarela
- **No dia**: notificação laranja
- **Vencido**: notificação vermelha

## Plano de Contas

Estrutura hierárquica:
```
Receitas
  └── Mensalidades
  └── Matrículas
  └── Doações
Despesas
  └── Folha de pagamento
  └── Insumos
  └── Manutenção
```

- Nunca deletar fisicamente uma conta com movimentações
- Usar `ativo: false` para inativar

## Movimentações

```typescript
// Tipo: entrada (receita) ou saída (despesa)
{
  tipo: 'entrada' | 'saida',
  valor: decimal,           // DECIMAL(10,2)
  data: date,
  planoContasId: uuid,
  formaPagamentoId: uuid,
  contaBancariaId: uuid,
  descricao: string,
  comprovante?: string,     // path do arquivo
  ativo: boolean
}
```

## Tipos de Dados Financeiros

```sql
-- SEMPRE DECIMAL para dinheiro
valor DECIMAL(10,2)     -- Máximo R$ 99.999.999,99
saldo DECIMAL(12,2)     -- Para saldos maiores

-- NUNCA
valor FLOAT             -- Erros de arredondamento
valor INTEGER           -- Perde centavos
```

## Roles de Acesso

| Ação | Role Mínima |
|------|-------------|
| Ver extrato/movimentações | `assist` (2) |
| Lançar movimentação | `adjunto` (5) |
| Editar/cancelar lançamento | `drt` (8) |
| Relatório financeiro | `drt` (8) |
| Configurar plano de contas | `vp` (9) |
| Emitir boletos | `adjunto` (5) |

## Relatórios Financeiros

Exportação disponível via:
- **PDF**: jsPDF + html2canvas
- **Excel**: XLSX (via CDN)

Dados sensíveis nos relatórios exigem role `drt` ou superior.

## Checklist ao Alterar Módulo Financeiro

- [ ] Valores monetários usam `DECIMAL(10,2)` (nunca `FLOAT`)
- [ ] Cálculo de desconto verifica isenção ANTES das faltas
- [ ] Movimentação tem plano de contas associado
- [ ] Soft delete em movimentações (nunca deletar histórico)
- [ ] Roles verificadas para operações de escrita
- [ ] Alertas de vencimento de boleto funcionando

## Anti-Patterns

| Evitar | Por quê |
|--------|---------|
| `FLOAT` para valor monetário | Erros de ponto flutuante |
| Deletar movimentação | Perde auditoria |
| Calcular desconto antes de isenção | Valores negativos |
| Exibir relatório financeiro sem verificar role | Vazamento de dados |
