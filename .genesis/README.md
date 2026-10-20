# Genesis

Pacote de prompts que constrói a estrutura de governança de um projeto de
software: diretrizes, requisitos, arquitetura, decisões, documentação,
changelog, licença e gates de qualidade.

Chama-se um prompt; ele ramifica no resto.

---

## Uso

1. Copie a pasta `.genesis/` para a raiz do projeto.
2. Peça ao modelo: **"Execute `.genesis/00_ORQUESTRADOR.md`"**.

O orquestrador diagnostica o que falta, monta o roteiro, mostra para
aprovação e executa. No mesmo roteiro, pergunta quais skills opcionais
instalar (`frontend-design`, `diagnosing-bugs` e outras, via `npx skills add`).

Ao terminar, instala no projeto só o que o ciclo de trabalho continua usando —
gates, ADR, documentação, changelog e ferramentas — em `docs/governanca/`, e
**remove `.genesis/`**. A coluna **Persiste** do `MANIFEST.md` define o que
fica.

Rodar de novo no mesmo projeto é seguro: copie `.genesis/` outra vez; ele lê o
perfil do manifesto instalado, detecta o que já existe e cobre só as lacunas.

---

## Mapa

```text
00_ORQUESTRADOR    ponto de entrada — diagnostica, roteiriza, delega
MANIFEST           tabela ID -> caminho + perfil do projeto

10_DISCOVERY       o que é este projeto            -> VISAO
20_REQUISITOS      o que deve fazer e garantir     -> RF / RNF
30_ARQUITETURA     que forma deve ter              -> entrevista + ADR
40_ADR             como registrar decisões         -> adr/
50_DOCUMENTACAO    como documentar e manter        -> docs/
55_DOCUMENTACAO_USUARIO  guia de quem usa o produto -> docs/usuario/
60_CHANGELOG       como registrar mudanças         -> CHANGELOG
70_LICENCA         sob que termos existe           -> LICENSE, ATRIBUICOES
80_GATE_CODE       o código está bom?              -> PASS | FAIL
81_GATE_SECURITY   o código está seguro?           -> PASS | FAIL
82_GATE_FRONTEND   o frontend está medido?         -> PASS | FAIL
83_GATE_API        a API tem design coerente?      -> PASS | FAIL
84_GATE_INPUT      todo input é tratado no servidor? -> PASS | FAIL
85_RELATORIO_GATES o que os gates decidiram na fase?  -> docs/gates/
90_DIRETRIZES      como se trabalha aqui           -> CLAUDE.md
95_AGENTES         como um agente descobre isto    -> AGENTS.md

tools/             ferramentas avulsas, fora da sequência
  HUMANIZER        texto que não soa como saída de modelo
  MAPA_MODULOS     quem importa quem, quem usa qual biblioteca
```

---

## Como a auto-referência funciona

Os prompts se citam por **identificador simbólico**, nunca por caminho:

```text
Ao final da fase, execute [[GATE_CODE]].   correto
Ao final da fase, execute ../prompts/...   errado
```

O `MANIFEST.md` é a única tabela que traduz ID em caminho. Consequências:

- **Nenhum prompt depende da estrutura de pastas do projeto.** Projeto que usa
  `documentacao/` em vez de `docs/` muda uma linha do manifesto, e todos os
  prompts continuam corretos.
- **Nenhuma referência quebra em silêncio.** ID ausente do manifesto é
  detectável; caminho relativo errado só aparece quando alguém tenta abrir.
- **Adotar é editar um arquivo só.**

---

## Ordem, e por que ela é essa

Cada etapa consome a saída da anterior:

```text
DISCOVERY     define o perfil
    v
REQUISITOS    precisa saber o que é o projeto
    v
ARQUITETURA   precisa dos requisitos não funcionais para decidir a forma
    v
ADR           registra a decisão da arquitetura
    v
DOCUMENTACAO  documenta o que as etapas acima definiram
    v
GATES         medem contra a régua que os requisitos e a arquitetura criaram
```

Arquitetura decidida antes dos requisitos é preferência estética com aparência
de engenharia. Gate rodado antes das diretrizes mede contra régua inventada.

O orquestrador impõe a ordem, mas pula o que não se aplica: biblioteca sem
implantação não ganha runbook; projeto sem API não ganha `api/`.

---

## Outros agentes

O pacote é Markdown puro: funciona em Claude Code, Codex, Cursor, Gemini,
Copilot ou modelo local, sem instalação.

`95_AGENTES` gera um `AGENTS.md` na raiz — a convenção que Codex, Cursor,
Gemini e Claude Code leem sozinhos. Agente que não segue convenção nenhuma
recebe o caminho direto:

```text
Leia .genesis/MANIFEST.md e depois execute .genesis/00_ORQUESTRADOR.md
```

Atalhos específicos de ferramenta — skill do Claude, comando TOML do Gemini,
regra do Cursor — são opcionais e ficam documentados em `95_AGENTES`. Todos são
ponteiros de poucas linhas; o conteúdo fica em `docs/governanca/`.

---

## Ferramentas

`tools/` guarda prompts que não pertencem à sequência de montagem. Chamam-se
direto, a qualquer momento:

```text
Execute .genesis/tools/HUMANIZER.md neste texto: ...
```

`[[HUMANIZER]]` também é chamado por `[[DOCUMENTACAO]]`, sobre o texto corrido
dos documentos gerados — cabeçalho, tabela e identificador ficam intactos.

`[[MAPA_MODULOS]]` produz o grafo concreto de imports, para a LLM achar arquivo
sem varrer a base. `[[GATE_CODE]]` § 12.1 avisa quando ele fica velho.

---

## O que o pacote não faz

- Não escreve o código do projeto.
- Não decide licença nem arquitetura sozinho — recomenda e espera o dono.
- Não substitui advogado em questão de licenciamento.
- Não gera documento para o qual não haja conteúdo real. Seção vazia é
  removida, não preenchida com "a definir".
