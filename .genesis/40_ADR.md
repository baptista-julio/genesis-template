# Registro de Decisões (ADR)

Cria e mantém `[[DIR_ADR]]`. Usado por `[[ARQUITETURA]]` para o ADR fundador e
por qualquer trabalho posterior que tome decisão relevante.

Um ADR registra **uma decisão** e, principalmente, **o contexto que a
justificou**. O contexto é o que tem valor: a decisão sozinha vira regra que
ninguém entende e por isso ninguém ousa mudar.

---

## Quando escrever

Escreva quando a decisão for **cara de reverter** ou **não óbvia para quem
chega**:

- forma da arquitetura, fronteira entre módulos;
- escolha de banco, fila, protocolo de comunicação;
- dependência central, ou dependência com licença que exigiu decisão;
- estratégia de autenticação, autorização, criptografia;
- modelo de dados que muitos módulos compartilham;
- padrão que o código inteiro passa a seguir;
- abandonar deliberadamente uma prática comum;
- aceitar dívida técnica conscientemente.

Não escreva para: escolha de nome, formatação, detalhe de implementação local,
ou o que o código já diz sozinho. ADR de decisão trivial faz o índice virar
ruído e treina a equipe a não ler nenhum.

**Teste:** se daqui a um ano alguém puder perguntar "por que diabos isto é
assim?", escreva.

---

## Formato

Arquivo em `[[DIR_ADR]]`, nomeado `NNNN-titulo-em-kebab-case.md`. Numeração
sequencial com quatro dígitos, **nunca reaproveitada**.

```markdown
# ADR-0004 — Armazenamento local em SQLite

**Status:** Aceito
**Data:** 2026-09-12
**Decisor:** <quem decidiu>
**Relacionados:** ADR-0001, RNF-003, [[DOC_ARQUITETURA]]

## Contexto

O que era verdade quando a decisão foi tomada. Restrições, requisitos,
o que se sabia e — importante — o que não se sabia.

Escreva para quem não estava presente. Sem contexto, o ADR não permite
reavaliação honesta: quem lê não distingue o que foi princípio do que
foi circunstância.

## Problema

A pergunta que precisava de resposta, em uma frase.

## Alternativas

### A — SQLite embarcado
Como funcionaria. A favor. Contra.

### B — PostgreSQL local
Como funcionaria. A favor. Contra.

### C — Arquivos JSON
Como funcionaria. A favor. Contra.

Descreva cada alternativa com honestidade, **inclusive as rejeitadas**.
Alternativa apresentada como espantalho torna o ADR inútil: quem
reavaliar não saberá se ela foi mesmo considerada.

Registre também a opção de não fazer nada, quando for viável.

## Decisão

O que foi escolhido, e o porquê ancorado no contexto acima.

Aponte o critério que decidiu. Normalmente um trade-off dominou.

## Consequências

### Positivas
O que melhora.

### Negativas
O que piora. **Seção obrigatória.** Decisão sem consequência negativa
é sinal de que a análise não foi feita — toda escolha real fecha
portas.

### Neutras
O que muda sem melhorar nem piorar.

## Conformidade

Como se verifica que o código segue esta decisão. Teste de
arquitetura, regra de lint, item de revisão, verificação manual.

Decisão sem verificação é erodida em silêncio. Quando houver
verificação automática, ela entra no [[GATE_CODE]].

## Reavaliação

O que faria esta decisão ser revista. Um gatilho concreto:
"se a base passar de 5 GB", "se surgir necessidade de acesso
concorrente por múltiplos processos".

Não escreva "reavaliar quando necessário" — não é gatilho.
```

---

## Estados

| Estado | Significado |
| --- | --- |
| `Proposto` | Em discussão, ainda não vale |
| `Aceito` | Em vigor |
| `Rejeitado` | Considerado e recusado. Mantido: evita repropor |
| `Substituído` | Trocado por outro. Aponta o sucessor |
| `Obsoleto` | Não vale mais, sem sucessor. O problema deixou de existir |

**Nunca apague nem reescreva um ADR aceito.** ADR é registro histórico, não
documentação viva. Mudou de ideia? Escreva um novo e marque o antigo como
`Substituído`:

```markdown
**Status:** Substituído por [ADR-0012](0012-migracao-para-postgres.md)
```

E no novo:

```markdown
**Substitui:** [ADR-0004](0004-armazenamento-local-sqlite.md)
```

O par permite ler a evolução do raciocínio, que costuma ser mais instrutivo
que qualquer decisão isolada.

---

## Índice

`[[DIR_ADR]]` contém um `README.md` com a tabela completa, em ordem
decrescente:

```markdown
# Decisões arquiteturais

| ADR | Título | Status | Data |
| --- | --- | --- | --- |
| [0012](0012-migracao-para-postgres.md) | Migração para PostgreSQL | Aceito | 2026-09-01 |
| [0004](0004-armazenamento-local-sqlite.md) | Armazenamento local em SQLite | Substituído | 2026-03-14 |
| [0001](0001-monolito-modular.md) | Monólito modular | Aceito | 2026-01-20 |
```

Atualize o índice na mesma alteração que cria o ADR. Índice desatualizado é a
falha mais comum aqui, e a que faz o registro inteiro perder utilidade.

---

## Regras

1. **Um ADR, uma decisão.** Texto com "e também decidimos" são dois ADRs.
2. **ADR não é documentação técnica.** Não explique como o sistema funciona —
   isso é `[[DOCUMENTACAO]]`. Explique por que ele é assim.
3. **Tempo passado, fatos do momento.** O ADR congela o que era verdade na
   data. Não o atualize quando o mundo mudar; escreva outro.
4. **Consequência negativa é obrigatória.**
5. **Data real.** Nunca invente data. Sem certeza, use a do versionador.

---

## Saída

```text
ADR
Criados: lista com id e título
Atualizados de status: lista com motivo
Índice: atualizado
Sem verificação de conformidade: lista  <- risco de erosão silenciosa
```
