---
name: api-design
description: Padrões de design de API REST para o ERP ITP. Use ao criar novos endpoints, DTOs, validações e respostas. Garante consistência com a API existente em NestJS.
---

# API Design — ERP ITP

## Convenções de URL

```
GET    /api/[recurso]           — listar (com filtros via query params)
GET    /api/[recurso]/:id       — buscar um
POST   /api/[recurso]           — criar
PATCH  /api/[recurso]/:id       — atualizar parcialmente
DELETE /api/[recurso]/:id       — desativar (soft delete)

# Sub-recursos
GET    /api/turmas/:id/alunos
POST   /api/turmas/:id/chamada
```

**Plural sempre.** Kebab-case: `/api/grade-horaria`.

## Filtros e Paginação

```typescript
// Query params padrão
GET /api/alunos?search=João&ativo=true&page=1&limit=20

// DTO de query
export class ListAlunosDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @Transform(({ value }) => value === 'true')
  @IsBoolean()
  ativo?: boolean;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}
```

## DTOs

```typescript
// Create DTO — todos os campos obrigatórios explícitos
export class CreateAlunoDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  nome: string;

  @IsEmail()
  email: string;

  @IsDateString()
  dataNascimento: string;

  @IsOptional()
  @IsString()
  telefone?: string;
}

// Update DTO — tudo opcional via PartialType
export class UpdateAlunoDto extends PartialType(CreateAlunoDto) {}
```

## Respostas Padrão

```typescript
// Lista com paginação
{
  data: [...],
  total: 150,
  page: 1,
  limit: 20
}

// Item único — retornar objeto direto
{
  id: "uuid",
  nome: "...",
  ...
}

// Criação — retornar objeto criado com status 201
@Post()
@HttpCode(HttpStatus.CREATED)
create(@Body() dto: CreateDto) { ... }

// Erro — NestJS trata automaticamente
throw new NotFoundException('Aluno não encontrado');
throw new BadRequestException('CPF já cadastrado');
throw new ForbiddenException();
```

## Validação

```typescript
// main.ts — pipe global (já configurado)
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,       // Remove campos não declarados no DTO
  forbidNonWhitelisted: true,
  transform: true,       // Transforma tipos automaticamente
}));
```

Decorators úteis do `class-validator`:

```typescript
@IsString()           // string
@IsNumber()           // número
@IsInt()              // inteiro
@IsBoolean()          // boolean
@IsEmail()            // email válido
@IsUUID()             // UUID v4
@IsDateString()       // ISO date string
@IsEnum(MeuEnum)      // valor do enum
@IsNotEmpty()         // não vazio
@IsOptional()         // campo opcional
@MaxLength(100)       // tamanho máximo
@Min(0) @Max(100)     // range numérico
@Type(() => Number)   // conversão de tipo (query params)
```

## Autenticação nos Endpoints

```typescript
// Protegido — requer JWT + role mínima
@Get()
@Roles('assist')        // roleLevel >= 2
findAll() {}

@Post()
@Roles('drt')           // roleLevel >= 8
create() {}

// Público — sem auth
@Get('publico')
@Public()
metodoPublico() {}

// Acesso ao usuário logado
@Get('perfil')
@Roles('user')
getPerfil(@Request() req) {
  return req.user;  // { sub, email, role, nome, grupo }
}
```

## Upload de Arquivos

```typescript
@Post('upload')
@Roles('assist')
@UseInterceptors(FileInterceptor('arquivo'))
upload(@UploadedFile() file: Express.Multer.File) {
  // file.filename, file.path, file.mimetype, file.size
}
```

## Checklist de Endpoint

- [ ] URL segue convenção REST (plural, kebab-case)
- [ ] DTO com class-validator para body/query
- [ ] `@Roles()` definido (nunca sem guard em endpoint sensível)
- [ ] Erros lançam HttpException correto (Not/Bad/Forbidden)
- [ ] Soft delete em vez de DELETE físico
- [ ] Resposta consistente com o padrão do módulo

## Anti-Patterns

| Evitar | Por quê | Em vez disso |
|--------|---------|--------------|
| Lógica de negócio no controller | Difícil testar | Mover pro service |
| Retornar senha/token em response | Segurança | Excluir campo no DTO de resposta |
| Sem validação no body | Crash ou injeção | Sempre usar DTO + ValidationPipe |
| Endpoint sem `@Roles()` | Acesso não autorizado | Mínimo `@Roles('user')` |
| Deletar fisicamente | Perde histórico | `ativo: false` |
