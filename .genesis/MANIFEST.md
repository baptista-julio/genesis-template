# Manifesto do Genesis

Tabela única de resolução de identificadores. Todo prompt deste pacote
referencia outro por **ID** entre colchetes duplos — `[[GATE_CODE]]` — e nunca
por caminho relativo. Este arquivo é o que traduz ID em caminho.

Ao adotar o pacote em um projeto novo, **este é o único arquivo que se edita**.

---

## Como resolver uma referência

Ao encontrar `[[ID]]` em qualquer prompt:

1. Procure o ID na tabela **Prompts** ou **Artefatos** abaixo.
2. Use o caminho da coluna `Caminho`.
3. Se o ID não estiver na tabela, ele não existe neste projeto — trate a
   referência como opcional e siga adiante, registrando a ausência.
4. Nunca invente caminho. Nunca navegue por `../` para achar o arquivo.

Caminhos são relativos à **raiz do projeto**, não à pasta deste pacote.

---

## Prompts

Os prompts são o pacote em si. Mudam de lugar apenas se você mover a pasta.

A coluna **Persiste** diz o que sobrevive à montagem. Ao final, o
`[[ORQUESTRADOR]]` copia os prompts marcados para `[[DIR_GOVERNANCA]]` e remove
`.genesis/` — ver **Depois da montagem**, abaixo. Prompt com `não` só serve
para montar a estrutura.

| ID | Caminho | Responde a pergunta | Persiste |
| --- | --- | --- | --- |
| `[[ORQUESTRADOR]]` | `.genesis/00_ORQUESTRADOR.md` | O que este projeto precisa, e em que ordem? | não |
| `[[DISCOVERY]]` | `.genesis/10_DISCOVERY.md` | O que é este projeto, afinal? | não |
| `[[REQUISITOS]]` | `.genesis/20_REQUISITOS.md` | O que o sistema precisa fazer e garantir? | não |
| `[[ARQUITETURA]]` | `.genesis/30_ARQUITETURA.md` | Que forma o sistema deve ter? | não |
| `[[ADR]]` | `.genesis/40_ADR.md` | Como registrar uma decisão? | sim |
| `[[DOCUMENTACAO]]` | `.genesis/50_DOCUMENTACAO.md` | Como criar e manter a documentação? | sim |
| `[[DOCUMENTACAO_USUARIO]]` | `.genesis/55_DOCUMENTACAO_USUARIO.md` | Como explicar o produto a quem o usa? | sim |
| `[[CHANGELOG]]` | `.genesis/60_CHANGELOG.md` | Como registrar o que mudou? | sim |
| `[[LICENCA]]` | `.genesis/70_LICENCA.md` | Sob que termos isto é distribuído? | não |
| `[[GATE_CODE]]` | `.genesis/80_GATE_CODE.md` | O código está bom o bastante? | sim |
| `[[GATE_SECURITY]]` | `.genesis/81_GATE_SECURITY.md` | O código está seguro o bastante? | sim |
| `[[GATE_FRONTEND]]` | `.genesis/82_GATE_FRONTEND.md` | O frontend está conforme, rápido e mensurável? | se `interface_web: sim` |
| `[[GATE_API]]` | `.genesis/83_GATE_API.md` | A API tem design consistente, previsível e sustentável? | se `tem_api: sim` |
| `[[GATE_INPUT]]` | `.genesis/84_GATE_INPUT.md` | Todo dado externo é validado e tratado no servidor? | sim |
| `[[RELATORIO_GATES]]` | `.genesis/85_RELATORIO_GATES.md` | O que os gates decidiram nesta fase? | sim |
| `[[DIRETRIZES]]` | `.genesis/90_DIRETRIZES.md` | Como se trabalha neste repositório? | não |
| `[[AGENTES]]` | `.genesis/95_AGENTES.md` | Como um agente descobre tudo isto? | não |

---

## Ferramentas

Prompts avulsos. Não fazem parte da sequência do `[[ORQUESTRADOR]]` — são
chamados quando a tarefa pede.

| ID | Caminho | O que faz | Persiste |
| --- | --- | --- | --- |
| `[[HUMANIZER]]` | `.genesis/tools/HUMANIZER.md` | Reescreve texto para não soar como saída de modelo | sim |
| `[[MAPA_MODULOS]]` | `.genesis/tools/MAPA_MODULOS.md` | Gera o grafo de quem importa quem | sim |

---

## Artefatos

Os documentos que os prompts produzem e consultam. **Esta é a seção que muda
de projeto para projeto.** Os caminhos abaixo são o padrão sugerido; ajuste-os
à estrutura real e os prompts acompanham sem serem editados.

| ID | Caminho | Produzido por | Obrigatório |
| --- | --- | --- | --- |
| `[[DIR_GOVERNANCA]]` | `docs/governanca/` | `[[ORQUESTRADOR]]` | sim |
| `[[DOC_DIRETRIZES]]` | `CLAUDE.md` | `[[DIRETRIZES]]` | sim |
| `[[DOC_AGENTES]]` | `AGENTS.md` | `[[AGENTES]]` | sim |
| `[[DOC_CHANGELOG]]` | `docs/CHANGELOG.md` | `[[CHANGELOG]]` | sim |
| `[[DOC_LICENCA]]` | `LICENSE` | `[[LICENCA]]` | sim |
| `[[DOC_ATRIBUICOES]]` | `ATRIBUICOES.md` | `[[LICENCA]]` | se houver dependência |
| `[[DOC_VISAO]]` | `docs/VISAO.md` | `[[DISCOVERY]]` | sim |
| `[[DOC_REQUISITOS_F]]` | `docs/requisitos/funcionais.md` | `[[REQUISITOS]]` | sim |
| `[[DOC_REQUISITOS_NF]]` | `docs/requisitos/nao-funcionais.md` | `[[REQUISITOS]]` | sim |
| `[[DOC_ARQUITETURA]]` | `docs/architecture/README.md` | `[[ARQUITETURA]]` | sim |
| `[[DIR_ADR]]` | `docs/adr/` | `[[ADR]]` | sim |
| `[[DOC_INDICE]]` | `docs/README.md` | `[[DOCUMENTACAO]]` | sim |
| `[[DOC_MAPA]]` | `docs/MAPA_MODULOS.md` | `[[MAPA_MODULOS]]` | se houver código |
| `[[DOC_METRICAS_FRONT]]` | `docs/metricas/frontend.md` | `[[GATE_FRONTEND]]` | se houver interface web |
| `[[DIR_GATES]]` | `docs/gates/` | `[[RELATORIO_GATES]]` | quando houver fase concluída |
| `[[DIR_SPEC]]` | `docs/spec/` | `[[DOCUMENTACAO]]` | quando houver domínio |
| `[[DIR_API]]` | `docs/api/` | `[[DOCUMENTACAO]]` | se expuser API |
| `[[DIR_RUNBOOK]]` | `docs/runbook/` | `[[DOCUMENTACAO]]` | se houver implantação |
| `[[DIR_USUARIO]]` | `docs/usuario/` | `[[DOCUMENTACAO_USUARIO]]` | quando houver RF com efeito visível |
| `[[DOC_NOVIDADES]]` | `docs/usuario/novidades.md` | `[[DOCUMENTACAO_USUARIO]]` | se existir `[[DIR_USUARIO]]` |
| `[[DOC_GLOSSARIO]]` | `docs/GLOSSARIO.md` | `[[DOCUMENTACAO]]` | recomendado |

---

## Perfil do projeto

Preenchido por `[[DISCOVERY]]` na primeira execução. Os demais prompts leem
daqui para decidir o que se aplica — é o que evita rodar revisão de API em
projeto que não tem API.

```yaml
nome:
tipo:              # biblioteca | servico | aplicacao-web | desktop | cli | monorepo
estagio:           # bootstrap | em-desenvolvimento | maduro | legado
distribuicao:      # aberto | fechado | interno
arquitetura:       # definida por [[ARQUITETURA]]
linguagens:
tem_api:           # sim | nao
tem_banco:         # sim | nao
tem_interface:     # sim | nao
interface_web:     # sim | nao — se sim, [[GATE_FRONTEND]] se aplica
indexavel:         # sim | nao — a interface deve ser encontrada por busca?
tem_dependencias:  # sim | nao
equipe:            # solo | pequena | multipla
```

---

## Adotando em um projeto novo

1. Copie a pasta `.genesis/` para a raiz do projeto.
2. Execute `[[ORQUESTRADOR]]`.
3. Ele preenche o **Perfil do projeto** acima e ajusta a tabela de
   **Artefatos** aos caminhos reais — antes de gerar qualquer documento.

Nada além deste arquivo precisa ser alterado.

---

## Depois da montagem

`.genesis/` é andaime: sai do projeto quando a estrutura fica pronta. O que o
ciclo de trabalho continua usando fica em `[[DIR_GOVERNANCA]]`:

- os prompts com **Persiste** `sim`, ou cuja condição o perfil satisfaz,
  com o mesmo nome de arquivo — `tools/` achatado na raiz da pasta;
- uma cópia deste manifesto, com as colunas `Caminho` apontando para
  `[[DIR_GOVERNANCA]]`, as linhas dos prompts que não persistem removidas e
  esta seção substituída por uma linha dizendo como montar de novo.

A partir daí, o manifesto instalado é a fonte de verdade de perfil e caminhos.
ID de prompt que não persistiu cai na regra 3 de **Como resolver uma
referência**: opcional, siga adiante.

Para rodar o Genesis de novo — artefato novo, mudança de estágio, arquitetura
revista —, copie `.genesis/` outra vez para a raiz e execute
`[[ORQUESTRADOR]]`. Ele lê o manifesto instalado antes de perguntar qualquer
coisa.
