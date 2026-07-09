---
name: nestjs-patterns
description: Padrões e convenções NestJS específicos do projeto ERP ITP. Use ao criar módulos, guards, decorators, pipes, interceptors ou qualquer estrutura NestJS. Garante consistência com a arquitetura existente.
---

# NestJS Patterns — ERP ITP

Referência de padrões para o backend NestJS deste projeto.

## Stack

- NestJS 11 + TypeORM 0.3.20 + PostgreSQL (Neon em prod)
- Porta local: 3001
- Auth: JWT via Passport, cookie `itp_token` (httpOnly), expiração 8h

## Estrutura de Módulo

Todo módulo segue:

```
src/[modulo]/
  [modulo].module.ts
  [modulo].controller.ts
  [modulo].service.ts
  [modulo].entity.ts (ou entities/)
  dto/
    create-[modulo].dto.ts
    update-[modulo].dto.ts
  [modulo].controller.spec.ts
```

## Hierarquia de Roles

```
user(0) < cozinha(1) < assist(2) < monitor(3) < prof(4)
< adjunto(5) < drt(8) < vp(9) < prt/admin(10)
```

Guard verifica `roleLevel >= nivelMínimo`. Usar `@Roles('drt')` para proteger.

## Padrões de Controller

```typescript
@Controller('api/[recurso]')
@UseGuards(JwtAuthGuard, RolesGuard)
export class RecursoController {
  constructor(private readonly service: RecursoService) {}

  @Get()
  @Roles('assist')
  findAll() {
    return this.service.findAll();
  }

  @Post()
  @Roles('drt')
  create(@Body() dto: CreateRecursoDto) {
    return this.service.create(dto);
  }
}
```

## Rota Pública

```typescript
@Get('publica')
@Public()  // Decorator @Public() — ignora JwtAuthGuard
metodoPublico() {}
```

## Entidade TypeORM

```typescript
@Entity('nome_tabela')
export class Recurso {
  @PrimaryGeneratedColumn('uuid')
  id: string;  // UUID via gen_random_uuid()

  @Column()
  nome: string;

  @Column({ default: true })
  ativo: boolean;  // Soft delete padrão do projeto

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

## Soft Delete

**Nunca deletar fisicamente.** Sempre usar `ativo: boolean`.

```typescript
// Service — desativar
async remove(id: string) {
  await this.repo.update(id, { ativo: false });
}

// Query — sempre filtrar ativos
findAll() {
  return this.repo.find({ where: { ativo: true } });
}
```

## DTOs com Validação

```typescript
import { IsString, IsNotEmpty, IsOptional, IsUUID } from 'class-validator';

export class CreateRecursoDto {
  @IsString()
  @IsNotEmpty()
  nome: string;

  @IsOptional()
  @IsString()
  descricao?: string;
}
```

## Service com TypeORM

```typescript
@Injectable()
export class RecursoService {
  constructor(
    @InjectRepository(Recurso)
    private repo: Repository<Recurso>,
  ) {}

  findAll() {
    return this.repo.find({ where: { ativo: true } });
  }

  async findOne(id: string) {
    const item = await this.repo.findOne({ where: { id, ativo: true } });
    if (!item) throw new NotFoundException('Recurso não encontrado');
    return item;
  }

  async create(dto: CreateRecursoDto) {
    const item = this.repo.create(dto);
    return this.repo.save(item);
  }

  async update(id: string, dto: UpdateRecursoDto) {
    await this.findOne(id);  // Valida existência
    await this.repo.update(id, dto);
    return this.findOne(id);
  }
}
```

## Migrations (não usar synchronize)

```bash
# Gerar migration após alterar entidade
npm run typeorm:migration:generate -- -n NomeDaMigration

# Executar
npm run typeorm:migration:run

# Reverter
npm run typeorm:migration:revert
```

`synchronize` está **desabilitado** em prod. Sempre usar migrations.

## Módulo — Registro

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([Recurso])],
  controllers: [RecursoController],
  providers: [RecursoService],
  exports: [RecursoService],  // Se outros módulos precisarem
})
export class RecursoModule {}
```

Registrar em `AppModule`.

## Anti-Patterns

| Evitar | Por quê | Em vez disso |
|--------|---------|--------------|
| `synchronize: true` em prod | Destrói dados | Migrations |
| DELETE físico | Perde histórico | `ativo: false` |
| Lógica no controller | Difícil testar | Mover pro service |
| Sem `@Roles()` em endpoints sensíveis | Brecha de segurança | Sempre definir role mínima |
| UUID hardcoded | Frágil | Usar parâmetro de rota |

## Checklist ao Criar Módulo

- [ ] Entidade com `id` UUID, `ativo`, `createdAt`, `updatedAt`
- [ ] DTOs com class-validator
- [ ] Service com findOne que lança NotFoundException
- [ ] Controller com `@Roles()` em todos os endpoints
- [ ] Módulo registrado no AppModule
- [ ] Migration gerada (se nova tabela/coluna)
