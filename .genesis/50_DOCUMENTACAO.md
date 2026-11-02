# Documentação

Cria, mantém e reorganiza a documentação do projeto. Produz `[[DOC_INDICE]]`,
`[[DIR_SPEC]]`, `[[DIR_API]]`, `[[DIR_RUNBOOK]]`, `[[DOC_GLOSSARIO]]`.

Define também o **cabeçalho de metadados** e o **ciclo de vida** que todos os
demais prompts do pacote referenciam.

---

## Modo de operação

Leia `estagio` no perfil do `MANIFEST.md`:

| Estágio | Modo | O que fazer |
| --- | --- | --- |
| `bootstrap` | **Criação** | Não há código para auditar. Documente o que foi decidido em `[[DOC_VISAO]]`, `[[DOC_REQUISITOS_F]]` e `[[DOC_ARQUITETURA]]`, tudo como `Proposto`. Crie o esqueleto e o índice; não invente conteúdo |
| `em-desenvolvimento` | **Misto** | Parte do código existe. Documente o implementado como `Implementado` e o planejado como `Proposto`. A distinção é o valor principal |
| `maduro` / `legado` | **Reorganização** | O código é a fonte da verdade. Execute as três etapas de auditoria abaixo |

Há ainda um quarto modo, independente do estágio:

| Modo | Quando | O que fazer |
| --- | --- | --- |
| **Incremental** | Ao fim de uma fase de implementação, ou quando `[[GATE_CODE]]` § 21.2 acusar divergência | Atualiza apenas os documentos afetados pelo diff. Ver seção própria abaixo |

No modo Criação, **as etapas 1 a 3 não se aplicam** — pule direto para
Estrutura. Auditar código que não existe produz documento ficcional.

---

## Princípios

### Fonte única da verdade

Uma informação existe em um lugar. Outro documento que precise dela usa link.

Duplicar é criar divergência futura: as duas cópias começam iguais e uma delas
é esquecida na próxima alteração. A partir daí, quem lê não sabe qual vale.

### Responsabilidade única

Cada documento responde **uma** pergunta. "Como funciona a autenticação?"
"Como fazer a implantação?" "Que decisões foram tomadas?"

Documento que responde a três perguntas é atualizado por três motivos
diferentes, e por isso desatualiza três vezes mais rápido.

### Organização por domínio

Organize pelo domínio do sistema, não pela tecnologia:

```text
spec/autenticacao/     em vez de   backend.md
spec/importacao/                   frontend.md
spec/relatorios/                   banco.md
```

A documentação deve acompanhar a forma como quem desenvolve pensa o sistema.
Ninguém depura "o backend" — depura a autenticação.

### Escalabilidade

Nenhum documento cresce indefinidamente. Quando um documento começa a tratar
de assuntos distintos, divida:

```text
importacao/
  README.md       visão e índice
  pipeline.md
  formatos.md
  erros.md
  desempenho.md
```

### Clareza e manutenibilidade

Documentação reflete o comportamento atual. Havendo divergência, **o código é a
verdade** e a documentação é corrigida.

Depois de alterar um módulo, quem desenvolve deve precisar consultar dois ou
três documentos. Se a alteração exige navegar por dezenas de arquivos, a
organização está errada.

---

## Cabeçalho de metadados

Todo documento crítico começa com este bloco. É a referência que os demais
prompts do pacote citam.

```yaml
Status: Implementado
Versão: 2.1
Criado em: 2026-07-19
Atualizado em: 2026-09-12
Domínio: Importação
Owner: <omitir se equipe: solo>

Implementação:
  - src/importacao/

Relacionados:
  - ../architecture/README.md
  - ../adr/0004-armazenamento-local-sqlite.md

Depende de: Autenticação, Armazenamento
Usado por: Relatórios
```

Preencha o que fizer sentido para o documento. **Nunca mantenha campo
desatualizado** — campo errado é pior que campo ausente, porque tem aparência
de informação.

---

## Ciclo de vida

Nenhum documento pode deixar dúvida se descreve proposta, implementação ou
história.

| Status | Significado |
| --- | --- |
| `Proposto` | Ainda não implementado. Sujeito a mudança |
| `Em desenvolvimento` | Implementação em andamento |
| `Implementado` | Reflete exatamente o comportamento atual. Estado esperado da maioria |
| `Obsoleto` | Não faz mais parte do sistema. Mantido por história. Aponte o substituto |
| `Arquivado` | Preservado por motivo legal ou de auditoria. Não use como referência |

Documento `Implementado` que não corresponde ao código é defeito, não dívida:
corrija na hora em que for detectado.

---

## Política de evolução

A documentação é parte do software.

Toda alteração que mude comportamento, arquitetura, API, dados, regra de
negócio ou operação **atualiza a documentação correspondente na mesma
alteração**.

Nenhuma funcionalidade está concluída enquanto a documentação relacionada não
estiver consistente com a implementação. Documentação não é atividade
posterior — atividade posterior é atividade que não acontece.

---

## Etapa 1 — Análise

*(modo Reorganização)*

Antes de tocar em qualquer documento, entenda o sistema. Examine: código-fonte,
documentação existente, `[[DOC_CHANGELOG]]`, configuração, estrutura de pastas,
dependências, APIs, modelo de dados, testes, diagramas.

Só então comece. Reorganizar documentação sem entender o sistema reorganiza a
ficção existente.

## Etapa 2 — Atualização

Corrija o que está desatualizado: documentos, diagramas, exemplos, APIs,
referências cruzadas, cabeçalhos, status, versões.

**Nunca remova informação relevante** nesta etapa. Consolidar vem depois.

## Etapa 3 — Consolidação

Só depois de tudo atualizado: elimine duplicação, funda documentos
equivalentes, mova conteúdo para o lugar certo, reorganize a estrutura.

O objetivo não é resumir. É consolidar preservando o que importa.

---

## Estrutura

Ajuste ao perfil — não crie pasta que o projeto não precisa:

```text
docs/
  README.md          portal e mapa
  VISAO.md
  CHANGELOG.md
  GLOSSARIO.md

  requisitos/
    funcionais.md
    nao-funcionais.md

  architecture/
    README.md

  spec/              um diretório por domínio
    <dominio>/
      README.md

  adr/
    README.md

  api/               só se tem_api
  runbook/           só se houver implantação
  usuario/           só se houver RF com efeito visível; ver [[DOCUMENTACAO_USUARIO]]
  gates/             relatório final de cada fase; ver [[RELATORIO_GATES]]
  test/
```

Pasta vazia não é criada. Diretório com README dizendo "a definir" é ruído.

---

## Portal

`[[DOC_INDICE]]` é a porta de entrada. Contém: visão geral, arquitetura
resumida, **mapa completo da documentação**, links para todas as áreas, e por
onde começa quem chega hoje.

---

## Documentos de especificação

Cada domínio tem sua pasta, com README contendo objetivo, visão geral, índice
dos documentos e relação com outros domínios.

Estrutura sugerida para cada documento — **use só as seções necessárias**:

```markdown
# Título
## Índice
## Visão geral
## Objetivo
## Fluxograma
## Fluxo detalhado
## Arquitetura
## Dados
## APIs
## Segurança
## Desempenho
## Casos de erro
## Referências
```

Nunca crie seção vazia.

Todo documento de especificação tem índice navegável. Havendo diagrama, o
índice aponta separadamente para o texto e para o diagrama — permite consultar
só o desenho quando é isso que se quer.

---

## Diagramas

Use Mermaid quando houver fluxo. Prefira: fluxograma, sequência, estado,
classes, entidade-relacionamento, C4 simplificado.

Ordem obrigatória: **visão geral em texto → diagrama → explicação detalhada**.
Diagrama complementa o texto, nunca o substitui — diagrama sozinho é
interpretável de várias formas.

---

## API

*(só se `tem_api: sim`)*

Organizada por domínio em `[[DIR_API]]`. Cada documento traz endpoints,
autenticação, exemplos de requisição e resposta, códigos de erro e contratos.

---

## Runbook

*(só se houver implantação)*

Ambientes, configuração, variáveis, contêineres, backup, restauração,
retorno de versão, monitoramento, logs e diagnóstico de problemas.

Escrito para ser seguido às três da manhã por quem não escreveu o sistema.
Passo sem comando literal não serve.

---

## Glossário

`[[DOC_GLOSSARIO]]` reúne os termos do projeto, cada um com definição curta.
Os demais documentos referenciam em vez de repetir.

Inclua o vocabulário do negócio, não só o técnico. O termo que o cliente usa e
o código não usa é exatamente o que precisa de entrada.

---

## Instruções de IA

Instruções destinadas a modelos vivem exclusivamente em `[[DOC_DIRETRIZES]]` —
convenções, padrões, regras de documentação, comportamento esperado.

Nenhuma delas é copiada para a documentação do produto. São públicos
diferentes, e misturar os dois faz a documentação envelhecer junto com as
ferramentas.

---

## Documentação do usuário

Quem usa o produto lê outra documentação, em `[[DIR_USUARIO]]`, sem a parte
técnica. Ela nasce desta e é mantida por `[[DOCUMENTACAO_USUARIO]]`, ao fim da
fase, depois do modo incremental. Este prompt não a escreve: só mantém a
coluna **Guia do usuário** coerente quando um RF muda de lugar ou é revogado.

---

## Modo Incremental

O modo usado no dia a dia. Os outros três montam ou refazem a documentação;
este a mantém viva, que é o trabalho que de fato se repete.

Entrada: o diff de uma fase de implementação — normalmente vindo de
`[[GATE_CODE]]` § 21.2, que já montou o mapa `código -> documento`.

**Escopo fechado.** Toque apenas nos documentos que o diff afetou. A tentação
de "já que estou aqui, arrumo aquela outra seção" transforma uma atualização
de cinco minutos em reorganização, e é o motivo pelo qual o passo acaba sendo
pulado nas fases seguintes.

Reorganização é trabalho próprio, agendado, com o modo Reorganização. Não
acontece de carona.

### Passos

1. **Receba ou monte o mapa.** Se `[[GATE_CODE]]` já produziu, use. Senão,
   percorra o campo `Implementação:` dos documentos como descrito lá.

2. **Para cada documento afetado, decida o que mudou de verdade.** Comportamento
   novo, contrato alterado, regra de negócio, caso de erro novo. Refatoração
   sem mudança observável não gera edição — documento que continua verdadeiro
   fica como está.

3. **Edite o conteúdo.** Corrija o texto, os exemplos, os diagramas afetados.
   Preserve o que continua válido: não reescreva o documento inteiro por causa
   de um parágrafo.

4. **Atualize o cabeçalho.** `Atualizado em` com a data real. `Versão`
   incrementada. `Status` de `Proposto` para `Implementado` quando a
   funcionalidade passou a existir. `Implementação:` se o código mudou de
   lugar.

5. **Propague.** Uma alteração costuma tocar mais do que o documento óbvio:

   | Mudou | Verifique também |
   | --- | --- |
   | Comportamento de um domínio | README do domínio, casos de erro |
   | Contrato de API | `[[DIR_API]]`, exemplos, códigos de erro |
   | Modelo de dados | Seção de dados, diagrama ER, runbook de backup |
   | Fronteira entre módulos | `[[DOC_ARQUITETURA]]`, grafo de dependências, ADR |
   | Operação, variável de ambiente | `[[DIR_RUNBOOK]]` |
   | Requisito atendido | Matriz de rastreabilidade em `[[DOC_REQUISITOS_F]]`, inclusive a coluna **Guia do usuário** (preenchida por `[[DOCUMENTACAO_USUARIO]]`) |
   | Termo novo do domínio | `[[DOC_GLOSSARIO]]` |
   | Documento novo criado | `[[DOC_INDICE]]`, e o `MANIFEST.md` se for artefato de primeira ordem |

6. **Decisão vira ADR.** Se a fase tomou decisão que `[[ADR]]` considera
   registrável, escreva o ADR — ele não é substituível por parágrafo em
   documento de especificação.

7. **Revise a escrita.** Passe `[[HUMANIZER]]` sobre o texto corrido que você
   acabou de editar, com as exceções da próxima seção.

8. **Valide os links** dos documentos tocados.

### Ordem no fluxo de fase

```text
implementação
    v
[[GATE_API]]         se a fase tocou a API
    v
[[GATE_FRONTEND]]    se a fase tocou a interface web
    v
[[GATE_INPUT]]       se a fase tocou fronteira de entrada
    v
[[GATE_SECURITY]]
    v
[[GATE_CODE]]        acusa divergência em § 21.2
    v
DOCUMENTACAO         modo incremental  <- aqui
    v
[[DOCUMENTACAO_USUARIO]]  guia de quem usa, derivado desta documentação
    v
[[GATE_CODE]]        re-executa § 21.2, precisa ficar limpa
    v
[[DOC_CHANGELOG]]    o PASS dos gates autoriza a entrada
    v
[[DOC_NOVIDADES]]    se a mudança é percebida por quem usa o produto
```

Documentação vem antes do changelog por um motivo prático: a entrada do
changelog resume o que mudou, e escrevê-la depois de atualizar a documentação
produz resumo melhor — você acabou de reler o que de fato mudou.

### Saída

```text
DOCUMENTAÇÃO — INCREMENTAL
Documentos no mapa: N
Atualizados: lista
Sem necessidade de mudança: lista com motivo
ADR criado: id, ou nenhum necessário
Propagações: lista
Links quebrados: N   <- precisa ser 0
Pendente: o que não deu para atualizar e por quê
```

---

## Revisão de escrita

Documentação gerada por modelo carrega os tiques do modelo: aquecimento antes
de afirmar, tríade forçada, "é importante ressaltar", gerúndio pendurado,
negrito decorativo, fecho que repete o parágrafo.

Isso não é questão de gosto. Documentação inflada custa tempo de leitura a cada
consulta, e treina a equipe a passar os olhos em vez de ler — o que derrota o
propósito de tê-la.

Ao final da criação ou da reorganização, passe `[[HUMANIZER]]` em modo **prosa**
sobre o texto corrido dos documentos produzidos.

**O que não é tocado pelo Humanizer:**

| Elemento | Por quê |
| --- | --- |
| Cabeçalho de metadados | Estrutura obrigatória, lida por pessoas e ferramentas |
| Tabelas de rastreabilidade e de requisitos | Formato é a função |
| Identificadores: `RF-001`, `ADR-0004`, `[[ID]]` | Referência quebra se mudar |
| Blocos de código, comandos, caminhos | Precisam ser literais |
| Diagramas Mermaid | Sintaxe |
| Status e datas | Valores controlados |
| Termos do `[[DOC_GLOSSARIO]]` | Consistência vence variedade |

O alvo é o texto corrido: visão geral, explicação de fluxo, objetivo, contexto
de ADR, descrição de erro.

**Repetição em documentação é virtude.** Quem consulta a seção de erro não leu
a de visão geral. Variar o termo da mesma coisa para "não repetir" é o
conselho errado aqui — vale para ensaio, não para referência.

Runbook merece atenção redobrada: é lido sob estresse, fora de ordem, por quem
não escreveu o sistema. Passo sem comando literal não serve, e adjetivo ali só
atrapalha.

---

## Validação final

Antes de concluir, verifique:

- a documentação reflete o código atual;
- documentos críticos têm cabeçalho completo;
- status, versão e datas consistentes;
- sem duplicação de informação;
- **todos os links internos resolvem** — link quebrado é falha desta etapa;
- diagramas Mermaid válidos;
- índices atualizados, inclusive o de `[[DIR_ADR]]`;
- APIs documentadas correspondem à implementação;
- arquitetura documentada corresponde ao sistema real;
- modelo de dados documentado corresponde ao real;
- nenhuma informação relevante perdida;
- cada documento com responsabilidade única;
- todo artefato existente aparece em `[[DOC_INDICE]]` e no `MANIFEST.md`;
- o texto corrido passou por `[[HUMANIZER]]`.

---

## Saída

```text
DOCUMENTAÇÃO
Modo: criação | misto | reorganização
Criados: N      Atualizados: N      Consolidados: N
Links quebrados: N   <- precisa ser 0
Sem cabeçalho: lista
Divergências código vs documentação: lista
Marcados Obsoleto: lista com motivo
Revisão de escrita: padrões corrigidos, ou não executada
```
