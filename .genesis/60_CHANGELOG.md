# Changelog

Cria e mantém `[[DOC_CHANGELOG]]`.

O changelog é escrito **para a equipe e para quem integra com o software**, não
para quem o escreve. Quem quer saber o que mudou linha a linha lê o versionador;
quem lê o changelog quer saber o que o software passou a fazer e por quê, com
requisito, decisão e verificação ao alcance.

Quem usa o produto sem conhecer a parte técnica lê outro registro:
`[[DOC_NOVIDADES]]`, em linguagem simples. Ver **Novidades para quem usa**,
abaixo.

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
changelog é escrita **depois** de `[[GATE_SECURITY]]` e `[[GATE_CODE]]`
retornarem `PASS`, e de `[[RELATORIO_GATES]]` gravar o relatório em
`[[DIR_GATES]]` — e de `[[GATE_INPUT]]`, `[[GATE_FRONTEND]]` ou `[[GATE_API]]`,
quando a fase tocou fronteira de entrada, a interface web ou a API. O gate
aprovado é o que autoriza a entrada.

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
  Verificação: relatório dos gates 2026-09-12-importacao (aprovado),
  cobertura 94%.
  Pendente: formato `.ods` não suportado; fora do escopo desta fase.
```

Objetivo e direto. O changelog não é relatório de gate — se a sessão precisar
de narrativa longa, ela pertence ao ADR.

---

## Novidades para quem usa

*(Quando `[[DOC_NOVIDADES]]` consta no manifesto.)*

Depois de escrever a entrada da sessão, decida para cada item se a pessoa que
usa o produto percebe a mudança. Percebe: execute `[[DOCUMENTACAO_USUARIO]]`,
seção **Novidades**, que escreve a entrada correspondente em
`[[DOC_NOVIDADES]]`. Não percebe: o item fica só aqui.

```text
entrada do changelog                     [[DOC_NOVIDADES]]
Importação de .xlsx (RF-001)       ──►   Novo: importar planilhas
Leitura em fluxo (ADR-0007)        ──►   Melhorou: arquivos grandes mais rápidos
Cobertura 94%                      ──►   (nada: a pessoa não percebe)
Refatorado o módulo de importação  ──►   (nada)
```

Na publicação de versão, renomeie `Próxima atualização` em `[[DOC_NOVIDADES]]`
junto com `[Não publicado]` aqui: os dois mudam na mesma alteração.

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

### Arquivamento

Um arquivo só é o padrão. Não crie changelog novo por quantidade de tarefas:
quem lê procura "o que mudou desde a versão X", e arquivo fatiado quebra essa
busca.

Quando `[[DOC_CHANGELOG]]` passar de cerca de 500 linhas, mova as versões
antigas para um arquivo por versão MAIOR (ou por ano, se o projeto não
versiona):

```text
docs/CHANGELOG.md          atual, versão mais recente no topo
docs/changelog/1.x.md      arquivado
docs/changelog/2025.md     arquivado (projeto sem versão)
```

- Mova o texto sem alterá-lo: a regra de nunca reescrever versão publicada
  vale também aqui.
- Deixe no fim de `[[DOC_CHANGELOG]]` uma seção **Versões anteriores** com o
  link para cada arquivo.
- `[Não publicado]` e a versão corrente nunca são arquivados.
- Atualize as citações ao changelog em outros documentos que apontem para as
  versões movidas.

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
Arquivado: <versões movidas e destino>, ou nenhum
Entradas com efeito visível sem novidade em [[DOC_NOVIDADES]]: N   <- precisa ser 0
```
