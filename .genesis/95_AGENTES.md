# Camada de Agentes

Produz `[[DOC_AGENTES]]` — o arquivo de convenção que torna o pacote
descobrível por qualquer agente de código que trabalhe no projeto.

O pacote é Markdown puro e funciona em qualquer agente sem instalação. Este
prompt resolve um problema menor mas real: **o agente precisa saber que o
pacote existe** antes de poder usá-lo.

---

## O problema

Cada ferramenta lê um arquivo diferente ao abrir um projeto:

| Agente | Lê automaticamente |
| --- | --- |
| Claude Code | `CLAUDE.md`, `AGENTS.md` |
| Codex | `AGENTS.md` |
| Cursor | `AGENTS.md`, `.cursor/rules/` |
| Gemini CLI | `AGENTS.md`, `GEMINI.md` |
| Copilot | `.github/copilot-instructions.md` |
| Demais e modelos locais | nada padronizado |

`AGENTS.md` é o maior denominador comum. **É a saída padrão deste prompt.**

Agente que não lê nada continua funcionando — basta apontar o caminho:
`leia .genesis/00_ORQUESTRADOR.md e execute`.

---

## Princípio

**Uma fonte, muitos ponteiros.**

O conteúdo vive em `.genesis/` e em `[[DOC_DIRETRIZES]]`. Todo arquivo gerado
aqui é ponteiro curto — nunca cópia. Conteúdo duplicado em cinco lugares
diverge em três semanas, e a partir daí cada agente segue uma versão diferente
das regras.

Se algum ponteiro precisar crescer além de uma tela, o conteúdo está no lugar
errado.

---

## Parte 1 — AGENTS.md

Sempre gerado. Na raiz do projeto.

```markdown
# Agentes

Instruções para qualquer agente de código que trabalhe neste repositório.

## Antes de alterar código

Leia `<caminho de [[DOC_DIRETRIZES]]>` — idioma, formato de commit,
regras de teste e de dependência.

Leia as trinta primeiras linhas de `<caminho de [[DOC_CHANGELOG]]>`
para saber em que estado o projeto está.

Procurando onde algo está implementado: consulte
`<caminho de [[DOC_MAPA]]>` antes de varrer a base. Confira o carimbo
de validade no topo; se estiver velho, varra.

## Governança

Este projeto usa o pacote Genesis, em `.genesis/`. Cada arquivo é um
prompt autocontido; execute o que a tarefa pedir.

| Tarefa | Execute |
| --- | --- |
| Montar ou completar a estrutura do projeto | `.genesis/00_ORQUESTRADOR.md` |
| Revisar qualidade ao fim de uma fase | `.genesis/80_GATE_CODE.md` |
| Revisar segurança, após o gate de código | `.genesis/81_GATE_SECURITY.md` |
| Registrar uma decisão arquitetural | `.genesis/40_ADR.md` |
| Criar ou atualizar documentação | `.genesis/50_DOCUMENTACAO.md` |
| Tirar cara de IA de um texto ou de uma interface | `.genesis/tools/HUMANIZER.md` |
| Gerar ou atualizar o mapa de módulos | `.genesis/tools/MAPA_MODULOS.md` |
| Registrar o que mudou | `.genesis/60_CHANGELOG.md` |
| Levantar ou revisar requisitos | `.genesis/20_REQUISITOS.md` |
| Decidir ou revisar a arquitetura | `.genesis/30_ARQUITETURA.md` |

Os prompts se referenciam por identificador `[[ID]]`. A tabela que
traduz identificador em caminho está em `.genesis/MANIFEST.md` —
leia-o antes de executar qualquer prompt do pacote.

## Ao terminar uma fase

1. `.genesis/80_GATE_CODE.md` — precisa retornar PASS
2. Se a seção 21.2 acusar documentação desatualizada:
   `.genesis/50_DOCUMENTACAO.md` em modo incremental, depois volte ao passo 1
3. `.genesis/81_GATE_SECURITY.md` — precisa retornar PASS
4. Entrada em `<caminho de [[DOC_CHANGELOG]]>`

O PASS dos dois gates é o que autoriza a entrada no changelog.

Documentação desatualizada reprova a fase como qualquer outro BLOCKER — a
seção 21.2 do gate compara o diff contra o campo `Implementação:` dos
documentos.

## Comandos

build:     <comando real>
teste:     <comando real>
lint:      <comando real>
tipos:     <comando real>
```

Regras de geração:

1. **Resolva os IDs antes de escrever.** `AGENTS.md` é lido por agentes que não
   conhecem a sintaxe `[[ID]]`; escreva o caminho real. A única menção ao
   mecanismo é a linha que manda ler o `MANIFEST.md`.
2. **Omita a linha do prompt que não se aplica.** Projeto sem API não precisa
   da linha de API na tabela.
3. **Verifique os comandos.** Comando que não roda é pior que comando ausente:
   o agente tenta, falha, e gasta a sessão descobrindo o certo.
4. **Uma tela.** Se passar disso, está copiando conteúdo em vez de apontar.

---

## Parte 2 — Ponteiros adicionais

**Não gere por padrão.** Só quando o usuário pedir, ou quando houver evidência
de que o projeto usa aquela ferramenta — um `.cursor/` existente, um fluxo de
Copilot configurado.

Cada camada extra é mais uma coisa a atualizar quando o pacote mudar. A
pergunta antes de criar qualquer uma: *isto economiza mais do que custa
manter?*

### CLAUDE.md

Se o projeto já tem `CLAUDE.md`, ou se o usuário usa Claude Code como
ferramenta principal:

```markdown
# CLAUDE.md

Veja [AGENTS.md](AGENTS.md).
```

Uma linha. Não duplique o conteúdo — o Claude Code lê os dois, e conteúdo
repetido gasta contexto sem adicionar informação.

Alternativa: se `[[DOC_DIRETRIZES]]` já for o `CLAUDE.md` do projeto, acrescente
a ele a seção de governança e deixe o `AGENTS.md` apontando de volta. Um dos
dois é a fonte; o outro aponta. Nunca os dois com conteúdo.

### Comandos de barra — Claude Code

Em `.claude/commands/<nome>.md`. Cada arquivo vira `/<nome>`:

```markdown
---
description: Gate de qualidade de código ao fim de uma fase
---

Execute `.genesis/80_GATE_CODE.md` sobre as alterações atuais.
```

Vale para os três ou quatro fluxos de uso diário — tipicamente `/gate-code`,
`/gate-security`, `/adr`. Criar um para os onze prompts é ruído: o resto se
chama pelo caminho, e é chamado raramente.

### Comandos — Gemini CLI

Em `.gemini/commands/<nome>.toml`:

```toml
description = "Gate de qualidade de código"
prompt = "Execute `.genesis/80_GATE_CODE.md` sobre as alterações atuais."
```

### Cursor

Em `.cursor/rules/genesis.mdc`:

```markdown
---
description: Governança do projeto
alwaysApply: true
---

Este projeto usa o pacote Genesis em `.genesis/`.
Antes de alterar código, leia `AGENTS.md`.
```

### Copilot

Em `.github/copilot-instructions.md`, ponteiro equivalente ao `CLAUDE.md`.

---

## Parte 3 — Agentes sem convenção

Nenhum arquivo a gerar. Registre em `AGENTS.md`, ou diga ao usuário, que
qualquer agente é instruído assim:

```text
Leia .genesis/MANIFEST.md e depois execute .genesis/00_ORQUESTRADOR.md
```

O pacote não depende de ferramenta, formato proprietário ou carregamento
automático. Essa é a propriedade que o torna portátil, e ela não deve ser
sacrificada por conveniência de invocação em nenhuma ferramenta específica.

---

## Manutenção

Quando o pacote mudar — prompt novo, prompt renomeado — os ponteiros
desatualizam em silêncio: apontam para arquivo que não existe mais, e o agente
descobre isso no meio da tarefa.

Ao final deste prompt, verifique que **todo caminho citado em todo ponteiro
gerado existe em disco**. Caminho morto aqui é falha, não detalhe.

---

## Saída

```text
CAMADA DE AGENTES
[[DOC_AGENTES]]: criado | atualizado em <caminho>
Ponteiros adicionais: lista, ou nenhum
Comandos verificados: N de M   <- precisa ser N = M
Caminhos citados que não existem: lista  <- precisa ser vazia
Agentes cobertos automaticamente: lista
Agentes que exigem caminho explícito: lista
```
