# Changelog

Cria e mantém `[[DOC_CHANGELOG]]`.

O changelog é escrito **para quem usa o software**, não para quem o escreve.
Quem quer saber o que mudou linha a linha lê o versionador; quem lê o changelog
quer saber o que o software passou a fazer.

---

## Formato

Baseado em *Keep a Changelog*, com versionamento semântico quando o projeto
distribui versões.

```markdown
# Changelog

Todas as mudanças relevantes deste projeto.
Formato baseado em Keep a Changelog; versionamento semântico.

## [Não publicado]

### Adicionado
- Importação de planilhas `.xlsx` com mapeamento de colunas (RF-001)

### Corrigido
- Datas em fuso diferente do local deixam de ser gravadas com um dia
  de diferença

## [1.2.0] — 2026-09-12

### Adicionado
- Exportação de relatório em PDF (RF-014)

### Alterado
- Armazenamento migrado de SQLite para PostgreSQL (ADR-0012).
  Bases existentes são migradas na primeira execução.

### Removido
- Suporte a importação via arquivo `.xls` antigo. Use `.xlsx`.

### Segurança
- Corrigida exposição de detalhe interno em mensagem de erro da
  importação
```

Categorias: `Adicionado`, `Alterado`, `Descontinuado`, `Removido`,
`Corrigido`, `Segurança`. Categoria vazia não aparece.

---

## Regras

1. **Escreva o efeito, não a tarefa.** "Importa planilhas com mais de 50 mil
   linhas sem travar", não "refatorado o módulo de importação". A etapa interna
   do roteiro — "Fase 2", "parte 1 de 3" — nunca aparece.

2. **Uma entrada por mudança percebida.** Três commits que entregam uma
   funcionalidade são uma entrada.

3. **Referencie.** Requisito (`RF-001`), decisão (`ADR-0012`), incidente. É o
   que liga o changelog ao resto da documentação e permite ir do efeito à
   razão.

4. **Mudança incompatível é destacada.** Diga o que quebra e como migrar. Quem
   lê precisa saber o que fazer antes de atualizar.

5. **Segurança tem categoria própria.** Sempre. Mesmo quando a correção veio
   junto de outra coisa.

6. **Sem entrada vazia.** "Diversas melhorias" não informa nada e ocupa o lugar
   do que informaria.

7. **Ordem decrescente.** Versão mais recente no topo, `[Não publicado]` acima
   de tudo.

8. **Data real, formato `AAAA-MM-DD`.** Nunca invente data.

---

## Versionamento semântico

Quando o projeto publica versões:

| Parte | Quando incrementa |
| --- | --- |
| MAIOR | Mudança incompatível com a versão anterior |
| MENOR | Funcionalidade nova, mantendo compatibilidade |
| CORREÇÃO | Correção de defeito, mantendo compatibilidade |

Antes de `1.0.0`, a promessa de compatibilidade não vale — diga isso no topo do
arquivo se for o caso.

Projeto que não distribui versões (aplicação interna, site) usa data como
identificador de seção, ou só `[Não publicado]` com entradas datadas.

---

## Entrada de sessão de trabalho

Quando o projeto usa gates (`[[DIRETRIZES]]` define isso), a entrada do
changelog é escrita **depois** de `[[GATE_CODE]]` e `[[GATE_SECURITY]]`
retornarem `PASS` — e de `[[GATE_FRONTEND]]` ou `[[GATE_API]]`, quando a fase
tocou a interface web ou a API. O gate aprovado é o que autoriza a entrada.

Fase que mexeu no frontend registra, junto com o que mudou, o que a medição
mostrou: a métrica que se moveu, de quanto para quanto. Changelog que diz
"melhorou o desempenho" sem número não registra nada.

Formato enxuto:

```markdown
## [Não publicado]

### Adicionado
- Importação de planilhas `.xlsx` (RF-001)

  Decisões: leitura em fluxo em vez de carregar tudo em memória —
  arquivos de 50 mil linhas estouravam o limite (ADR-0007).
  Verificação: gate de código PASS, gate de segurança PASS,
  cobertura 94%.
  Pendente: formato `.ods` não suportado; fora do escopo desta fase.
```

Objetivo e direto. O changelog não é relatório de gate — se a sessão precisar
de narrativa longa, ela pertence ao ADR.

---

## Dependências

Dependência nova é justificada na entrada correspondente: o que resolve e por
que não foi escrita. É o que `[[REQUISITOS]]` exige no RNF de licenciamento, e
o changelog é onde essa justificativa fica localizável no tempo.

---

## Manutenção

- Atualizado na mesma alteração que muda o comportamento, nunca depois.
- Publicar versão = renomear `[Não publicado]` para a versão com data, e abrir
  um `[Não publicado]` vazio acima.
- **Nunca reescreva entrada de versão publicada.** Errou? Corrija na versão
  seguinte, dizendo o que a anterior informou errado.

Se `[[DIRETRIZES]]` mandar ler as primeiras linhas do changelog no início de
cada sessão, este arquivo é contexto de trabalho além de registro — mais uma
razão para o topo ser legível e atual.

---

## Saída

```text
CHANGELOG
Criado | atualizado
Entradas adicionadas: N
Versão publicada: <se houve>
Entradas sem referência a requisito ou ADR: N
```
