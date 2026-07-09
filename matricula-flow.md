---
name: matricula-flow
description: Regras de negócio do fluxo de matrícula do ERP ITP. Use ao trabalhar com inscrições, matrícula direta, LGPD, geração de número de matrícula ou qualquer lógica do módulo de matrículas.
---

# Matrícula Flow — ERP ITP

## Módulos Envolvidos

- `matriculas` — inscrições, LGPD, matrícula direta
- `academico` — cursos e turmas de destino
- `auth` — roles que podem efetivar matrícula
- `EmailService` — envio de termos LGPD

## Numeração de Matrícula

Formato: `ITP-ROLE-YYYYMM-###`

```
ITP-ALUNO-202503-001
ITP-ALUNO-202503-002
ITP-FUNCIONARIO-202503-001
```

- Sequencial por mês + role
- Gerado automaticamente no backend
- Nunca editável pelo usuário

## Fluxo de Inscrição (Google Forms)

```
Google Forms
  → Google Apps Script (google-apps-script/)
    → POST /api/matriculas/inscricao
      → Salva inscrição
      → Envia termo LGPD por email
```

## Fluxo de Matrícula Direta

```
Operador (role >= drt)
  → POST /api/matriculas/direta
    → Valida dados obrigatórios por faixa etária
    → Gera número de matrícula (ITP-ROLE-YYYYMM-###)
    → Associa a turma
    → Envia termo LGPD
    → Retorna matrícula criada
```

## Documentos Obrigatórios por Faixa Etária

| Faixa | Documentos |
|-------|-----------|
| < 18 anos | RG ou Certidão de Nascimento + documento do responsável |
| >= 18 anos | RG ou CPF |
| Todas | Comprovante de residência + foto 3x4 |

Lógica de validação baseada em `dataNascimento` calculando idade atual.

## LGPD

```typescript
// Aluno >= 18: envia para o próprio aluno
EmailService.enviarTermoLGPD(email, token)

// Aluno < 18: envia para o responsável
EmailService.enviarTermoLGPDResponsavel(emailResponsavel, token, nomeAluno)
```

- Token único por matrícula
- Rota de aceite: `/lgpd/[token]` (pública)
- Rota de assinatura de documento: `/documentos/[token]` (pública)
- Status: `pendente` → `aceito` após clique no link

## Validações de Negócio

```typescript
// Verificar antes de efetivar matrícula:
// 1. Turma existe e está ativa
// 2. Turma não está lotada (vagas disponíveis)
// 3. Aluno não está já matriculado na mesma turma
// 4. Documentos obrigatórios presentes (por faixa etária)
// 5. CPF/RG único no sistema (sem duplicata)
```

## Roles com Acesso

| Ação | Role Mínima |
|------|-------------|
| Ver inscrições | `assist` (2) |
| Efetivar matrícula | `drt` (8) |
| Cancelar matrícula | `drt` (8) |
| Editar dados do aluno | `adjunto` (5) |
| Relatório de matrículas | `drt` (8) |

## Status de Matrícula

```
inscrito → em_analise → matriculado → cancelado
                     ↘ reprovado_documentos
```

## Campos Sensíveis (LGPD)

Campos que exigem consentimento explícito antes de exibir/exportar:
- CPF
- RG
- Endereço completo
- Dados do responsável

## Checklist ao Alterar Módulo de Matrículas

- [ ] Numeração automática não foi alterada
- [ ] Validação de documentos por faixa etária mantida
- [ ] Envio de termo LGPD não foi quebrado
- [ ] Roles de acesso verificadas
- [ ] Turma valida vagas disponíveis antes de matricular
- [ ] Token LGPD gerado e único
