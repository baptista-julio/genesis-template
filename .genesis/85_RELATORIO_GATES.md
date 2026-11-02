# Relatório dos Gates

Produz o relatório final da fase e o grava em `[[DIR_GATES]]`.

Roda **uma vez**, depois que todos os gates aplicáveis terminaram — não ao fim
de cada gate. Consolida o que eles já mediram, num formato único, e deixa um
registro que o `[[DOC_CHANGELOG]]` pode citar.

---

## Princípios

1. **Este prompt não mede nada.** Todo valor vem do veredito de um gate. Valor
   que nenhum gate mediu aparece como `n/m` (não medido), nunca estimado.
2. **O percentual informa, não decide.** A decisão é a regra de cada gate:
   um BLOCKER reprova, mesmo com 99% dos itens aprovados.
3. **Roda com qualquer resultado.** Fase reprovada também gera relatório: é
   nele que ficam as falhas, o que foi tentado e a ação seguinte.
4. **Um formato, sempre.** Mesmas seções, mesma ordem, em todas as fases.

---

## Quando roda

```text
[[GATE_API]] -> [[GATE_FRONTEND]] -> [[GATE_INPUT]] -> [[GATE_SECURITY]]
  -> [[GATE_CODE]] (§ 21.2 limpa, ou limite de iterações atingido)
  -> [[RELATORIO_GATES]]        <- grava em [[DIR_GATES]]
  -> [[DOC_CHANGELOG]]          <- cita o relatório
```

Gate que não se aplica à fase (sem API, sem interface web) entra como
`N/A` com o motivo, não some da tabela.

---

## Arquivo

```text
docs/gates/
  AAAA-MM-DD-<fase>.md
```

- Um arquivo por execução. Reexecução no mesmo dia: sufixo `-2`, `-3`.
- **Nunca sobrescreva** relatório antigo. Ele é a evidência do que foi
  aprovado naquele momento.
- `<fase>` em minúsculas e hífens, igual ao nome usado no changelog.

---

## Formato

Largura fixa de 60 colunas. **Só borda esquerda**: sem borda direita, o
alinhamento não depende de contar caracteres. Barras sempre com 20 posições.

Unicode (`█ ░ ✓ ✗ ☐ ─ ═`) por padrão. Se o ambiente não renderiza, ASCII:
`[OK] [X] #### ---- [ ]`. Sem emoji.

Cálculo da barra: `preenchidos = round(percentual / 5)`.

```text
════════════════════════════════════════════════════════════
 RELATÓRIO DOS GATES · <fase>
════════════════════════════════════════════════════════════
 Data:       AAAA-MM-DD
 Decisão:    ✗ REPROVADO            (ou ✓ APROVADO)
 Gates:      4 executados · 1 N/A

── 1. RESUMO ───────────────────────────────────────────────
 ✓ Aprovados    8 / 11
 ✗ Reprovados   3 / 11
 Sucesso        ██████████████░░░░░░  73%   (informativo)
 BLOCKERs       3        <- é isto que decide, não o percentual
 Avisos         2

── 2. GATES ────────────────────────────────────────────────
 Gate        Status    BLOCKER  Aviso  Iterações
 ──────────  ────────  ───────  ─────  ─────────
 API         ✓ PASS    0        1      1/3
 Frontend    ✓ PASS    0        0      1/3
 Input       ✗ FAIL    1        0      3/3
 Security    ✗ FAIL    2        1      2/3
 Code        — não executado (gate anterior reprovou)

── 3. MÉTRICAS ─────────────────────────────────────────────
 Só o que um gate mediu. Meta e limite vêm do próprio gate.

 ✓ Cobertura (linhas)    ████████████████░░░░  82%  ≥ 80%
 ✓ LCP                   ████████░░░░░░░░░░░░  1,9 s ≤ 2,5 s
 ✗ Complexidade máx.     ████████████████████  14   ≤ 10   +4
 · Latência p95          n/m

── 4. BLOQUEANTES ──────────────────────────────────────────
 ✗ [B1] Input · fronteira POST /importar
     Obtido: sem validação de tamanho no servidor
     Exigido: limite validado no backend
     → Validar tamanho antes de ler o corpo.
     Evidência: entrada de 2 GB aceita (gate Input, passo 4)

 ✗ [B2] Security · ...

── 5. AVISOS ───────────────────────────────────────────────
 ! [A1] API · campo `data` sem formato documentado

── 6. PLANO DE AÇÃO ────────────────────────────────────────
 ☐ Corrigir B1
 ☐ Corrigir B2
 ☐ Reexecutar os gates afetados
 ☐ Reexecutar este relatório

════════════════════════════════════════════════════════════
 DECISÃO FINAL
════════════════════════════════════════════════════════════
 ✗ FASE REPROVADA
 Aprovações 8/11 · Reprovações 3/11 · Sucesso 73%
 [BLOQUEADO] Changelog não autorizado.
 Corrija os bloqueantes e execute os gates de novo.
════════════════════════════════════════════════════════════
```

Fase aprovada: seções 4 e 6 viram uma linha (`Nenhum bloqueante.`), e a
decisão final diz que o changelog está autorizado.

---

## Passos

1. **Colete** o veredito de cada gate aplicável: status, BLOCKERs, avisos,
   iterações, evidências e métricas com seus limites.
2. **Marque `N/A`** nos gates que o perfil dispensa, com o motivo.
3. **Registre `não executado`** quando um gate anterior reprovou e o seguinte
   não rodou. Nunca preencha por dedução.
4. **Monte** o relatório no formato acima. Contagens saem da soma dos gates; se
   não fecham, há erro de coleta — corrija antes de gravar.
5. **Decida**: aprovado só se todo gate aplicável retornou `PASS` e a § 21.2 do
   `[[GATE_CODE]]` está limpa. Qualquer outro caso é reprovado.
6. **Grave** em `[[DIR_GATES]]` e exiba o mesmo texto ao usuário.

---

## Ligação com o changelog

A entrada de `[[DOC_CHANGELOG]]` cita o relatório no campo `Verificação:`, em
vez de repetir números:

```markdown
  Verificação: relatório dos gates 2026-10-19-importacao (aprovado).
```

`PASS` sem relatório gravado não autoriza a entrada.

---

## Manutenção

- Relatório é registro, não documento vivo: não se edita depois de gravado.
- Passa de meia centena de arquivos: agrupe por ano em subpasta
  (`docs/gates/2026/`). Os links do changelog acompanham a mudança.

---

## Saída

```text
RELATÓRIO DOS GATES
Arquivo: <caminho em [[DIR_GATES]]>
Decisão: aprovado | reprovado
Gates: N executados, N N/A, N não executados
BLOCKERs: N   Avisos: N
Métricas não medidas (n/m): N
Contagens conferidas contra os gates: sim | não   <- precisa ser sim
```
