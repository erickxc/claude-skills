---
name: typeorm-migration
description: Fluxo completo de migrations TypeORM para o ERP ITP. Use ao criar tabelas, adicionar colunas, alterar tipos, criar índices ou qualquer mudança de schema. Inclui comandos, templates e estratégias zero-downtime.
---

# TypeORM Migration — ERP ITP

## Regra Principal

`synchronize` está **desabilitado**. Toda mudança de schema exige migration.

## Comandos

```bash
# Sempre rodar dentro de apps/backend/
cd apps/backend

# Gerar migration baseada na diferença entre entidade e banco
npm run typeorm:migration:generate -- -n DescricaoDaMudanca

# Executar migrations pendentes
npm run typeorm:migration:run

# Reverter última migration
npm run typeorm:migration:revert

# Ver status das migrations
npm run typeorm:migration:show
```

## Template de Migration Manual

```typescript
import { MigrationInterface, QueryRunner, Table, TableIndex } from 'typeorm';

export class NomeDaMigration1234567890123 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Mudanças aqui
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // Reverter mudanças aqui (SEMPRE implementar)
  }
}
```

## Cenários Comuns

### Nova tabela

```typescript
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.createTable(new Table({
    name: 'nome_tabela',
    columns: [
      {
        name: 'id',
        type: 'uuid',
        isPrimary: true,
        default: 'gen_random_uuid()',
      },
      {
        name: 'nome',
        type: 'varchar',
        length: '255',
        isNullable: false,
      },
      {
        name: 'ativo',
        type: 'boolean',
        default: true,
      },
      {
        name: 'created_at',
        type: 'timestamp',
        default: 'CURRENT_TIMESTAMP',
      },
      {
        name: 'updated_at',
        type: 'timestamp',
        default: 'CURRENT_TIMESTAMP',
        onUpdate: 'CURRENT_TIMESTAMP',
      },
    ],
  }), true);
}

async down(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.dropTable('nome_tabela');
}
```

### Adicionar coluna (zero-downtime)

```typescript
// Passo 1: adicionar como nullable primeiro
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.addColumn('tabela', new TableColumn({
    name: 'nova_coluna',
    type: 'varchar',
    length: '100',
    isNullable: true,  // Nullable primeiro!
  }));
}

// Depois de backfill, segunda migration para tornar NOT NULL:
await queryRunner.changeColumn('tabela', 'nova_coluna', new TableColumn({
  name: 'nova_coluna',
  type: 'varchar',
  length: '100',
  isNullable: false,
}));
```

### Adicionar FK

```typescript
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.createForeignKey('tabela_filho', new TableForeignKey({
    columnNames: ['pai_id'],
    referencedTableName: 'tabela_pai',
    referencedColumnNames: ['id'],
    onDelete: 'RESTRICT',  // ou CASCADE / SET NULL
  }));
}

async down(queryRunner: QueryRunner): Promise<void> {
  const table = await queryRunner.getTable('tabela_filho');
  const fk = table.foreignKeys.find(fk => fk.columnNames.includes('pai_id'));
  await queryRunner.dropForeignKey('tabela_filho', fk);
}
```

### Adicionar índice

```typescript
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.createIndex('tabela', new TableIndex({
    name: 'IDX_tabela_coluna',
    columnNames: ['coluna'],
  }));
}

async down(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.dropIndex('tabela', 'IDX_tabela_coluna');
}
```

### Renomear coluna (zero-downtime)

```typescript
// Migration 1: adicionar nova coluna + copiar dados
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.addColumn('tabela', new TableColumn({
    name: 'novo_nome',
    type: 'varchar',
    isNullable: true,
  }));
  await queryRunner.query(`UPDATE tabela SET novo_nome = nome_antigo`);
}

// Migration 2 (após deploy): remover coluna antiga
async up(queryRunner: QueryRunner): Promise<void> {
  await queryRunner.dropColumn('tabela', 'nome_antigo');
}
```

## Estratégia ON DELETE

| Cenário | Estratégia |
|---------|-----------|
| Itens dependem do pai (ex: order_items → orders) | `CASCADE` |
| Referência importante, não deletar pai | `RESTRICT` |
| Relacionamento opcional | `SET NULL` |

## Checklist de Migration

- [ ] `down()` implementado e testado
- [ ] Novas colunas obrigatórias adicionadas como nullable primeiro
- [ ] FKs têm estratégia ON DELETE definida
- [ ] Índices criados em FKs e colunas de filtro
- [ ] Migration testada em ambiente local antes de subir

## Erros Comuns

| Erro | Causa | Solução |
|------|-------|---------|
| `column cannot be null` | Adicionou NOT NULL sem default/backfill | Adicionar nullable, backfill, depois constrainar |
| `relation already exists` | Migration rodou parcialmente | Implementar `down()` e reverter |
| `foreign key violation` | Dados órfãos existentes | Limpar dados antes ou usar SET NULL |
