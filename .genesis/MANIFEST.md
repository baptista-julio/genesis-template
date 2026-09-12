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

| ID | Caminho | Responde a pergunta |
| --- | --- | --- |
| `[[ORQUESTRADOR]]` | `.genesis/00_ORQUESTRADOR.md` | O que este projeto precisa, e em que ordem? |
| `[[DISCOVERY]]` | `.genesis/10_DISCOVERY.md` | O que é este projeto, afinal? |
| `[[REQUISITOS]]` | `.genesis/20_REQUISITOS.md` | O que o sistema precisa fazer e garantir? |
| `[[ARQUITETURA]]` | `.genesis/30_ARQUITETURA.md` | Que forma o sistema deve ter? |
| `[[ADR]]` | `.genesis/40_ADR.md` | Como registrar uma decisão? |
| `[[DOCUMENTACAO]]` | `.genesis/50_DOCUMENTACAO.md` | Como criar e manter a documentação? |
| `[[CHANGELOG]]` | `.genesis/60_CHANGELOG.md` | Como registrar o que mudou? |
| `[[LICENCA]]` | `.genesis/70_LICENCA.md` | Sob que termos isto é distribuído? |
| `[[GATE_CODE]]` | `.genesis/80_GATE_CODE.md` | O código está bom o bastante? |
| `[[GATE_SECURITY]]` | `.genesis/81_GATE_SECURITY.md` | O código está seguro o bastante? |
| `[[DIRETRIZES]]` | `.genesis/90_DIRETRIZES.md` | Como se trabalha neste repositório? |
| `[[AGENTES]]` | `.genesis/95_AGENTES.md` | Como um agente descobre tudo isto? |

---

## Ferramentas

Prompts avulsos. Não fazem parte da sequência do `[[ORQUESTRADOR]]` — são
chamados quando a tarefa pede.

| ID | Caminho | O que faz |
| --- | --- | --- |
| `[[HUMANIZER]]` | `.genesis/tools/HUMANIZER.md` | Reescreve texto para não soar como saída de modelo |
| `[[MAPA_MODULOS]]` | `.genesis/tools/MAPA_MODULOS.md` | Gera o grafo de quem importa quem |

---

## Artefatos

Os documentos que os prompts produzem e consultam. **Esta é a seção que muda
de projeto para projeto.** Os caminhos abaixo são o padrão sugerido; ajuste-os
à estrutura real e os prompts acompanham sem serem editados.

| ID | Caminho | Produzido por | Obrigatório |
| --- | --- | --- | --- |
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
| `[[DIR_SPEC]]` | `docs/spec/` | `[[DOCUMENTACAO]]` | quando houver domínio |
| `[[DIR_API]]` | `docs/api/` | `[[DOCUMENTACAO]]` | se expuser API |
| `[[DIR_RUNBOOK]]` | `docs/runbook/` | `[[DOCUMENTACAO]]` | se houver implantação |
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
