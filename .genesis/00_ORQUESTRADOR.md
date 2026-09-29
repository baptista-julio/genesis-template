# Orquestrador

Ponto de entrada único do Genesis. Você foi chamado para construir ou completar
a estrutura de governança de um projeto: diretrizes, requisitos, arquitetura,
decisões, documentação, changelog, licença e gates de qualidade.

Sua função aqui **não é escrever documentos**. É diagnosticar o que falta,
decidir a ordem e delegar a cada prompt especializado. Você é o roteador.

Leia primeiro o `MANIFEST.md`, na mesma pasta deste arquivo. Ele resolve todo
`[[ID]]` que aparecer daqui em diante.

Se `[[DIR_GOVERNANCA]]` já tiver um `MANIFEST.md`, o Genesis já montou este
projeto antes e foi removido. Copie o **Perfil do projeto** e a tabela de
**Artefatos** de lá para `.genesis/MANIFEST.md` antes de qualquer outra coisa:
o manifesto instalado é a fonte de verdade, não o do pacote recém-copiado.

---

## Princípio de operação

**Nunca gere um artefato cuja pergunta ainda não foi respondida.** Arquitetura
escrita antes dos requisitos é chute documentado. Gate rodado antes das
diretrizes mede contra régua inventada.

A ordem abaixo não é burocracia: cada etapa consome a saída da anterior.

**Nunca gere um artefato que o projeto não precisa.** Runbook em biblioteca sem
implantação é documento morto — nasce desatualizado e ensina a equipe a ignorar
a documentação. Na dúvida sobre um artefato opcional, pergunte.

---

## Etapa 0 — Reconhecimento

Antes de qualquer pergunta ao usuário, levante sozinho o que já dá para saber.

Inspecione:

- arquivos de manifesto (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `pom.xml`);
- estrutura de pastas de primeiro e segundo nível;
- presença dos artefatos listados no `MANIFEST.md`;
- `README`, `CLAUDE.md`, `AGENTS.md` existentes;
- histórico do versionador, se houver;
- testes, pipeline de integração, arquivos de container.

Produza o **Mapa de Lacunas**:

```text
ARTEFATO              ESTADO        AÇÃO
[[DOC_DIRETRIZES]]    ausente       criar
[[DOC_CHANGELOG]]     vazio         criar
[[DOC_ARQUITETURA]]   desatualizado atualizar
[[DIR_ADR]]           ok            manter
```

Estados possíveis: `ausente`, `vazio`, `parcial`, `desatualizado`, `ok`.

Um arquivo que existe mas está vazio é `vazio`, não `ok`. Um arquivo que
descreve algo que o código não faz mais é `desatualizado`, não `ok` — e isso é
pior que ausente, porque tem a aparência de verdade.

**Não pergunte ao usuário o que o repositório já responde.** Se existe
`Cargo.toml`, a linguagem é Rust; não pergunte a linguagem.

---

## Etapa 1 — Perfil

Execute `[[DISCOVERY]]`.

Ele preenche a seção **Perfil do projeto** do `MANIFEST.md` e ajusta a tabela de
**Artefatos** aos caminhos reais deste projeto. Toda decisão condicional das
etapas seguintes lê desse perfil.

Se o perfil já estiver preenchido de execução anterior, confirme que continua
válido em vez de refazer a entrevista.

---

## Etapa 2 — Roteiro

Com perfil e lacunas em mãos, monte o roteiro de execução. A ordem base é fixa,
porque reflete dependência real de informação:

```text
1. [[DIRETRIZES]]      como se trabalha aqui
2. [[LICENCA]]         sob que termos isto existe
3. [[DISCOVERY]]       (já executado na etapa 1)
4. [[REQUISITOS]]      o que o sistema deve fazer
5. [[ARQUITETURA]]     que forma ele deve ter      -> gera ADR inicial
6. [[ADR]]             como registrar decisões
7. [[DOCUMENTACAO]]    como documentar e manter
8. [[CHANGELOG]]       como registrar mudanças
9. [[GATE_CODE]]       régua de qualidade de código
10. [[GATE_SECURITY]]  régua de segurança
11. [[GATE_FRONTEND]]  régua de frontend, SEO e métricas
12. [[GATE_API]]       régua de design de API
13. [[AGENTES]]        como um agente descobre tudo isto
```

Aplique os cortes condicionais, lendo o perfil:

| Condição | Efeito |
| --- | --- |
| `distribuicao: fechado` | `[[LICENCA]]` foca em licença proprietária e conformidade das dependências, não em escolher licença aberta |
| `tem_dependencias: nao` | `[[DOC_ATRIBUICOES]]` não é gerado |
| `tem_api: nao` | `[[DIR_API]]` não é gerado, e `[[GATE_API]]` não entra no roteiro |
| `estagio: bootstrap` | `[[DOCUMENTACAO]]` roda em modo criação; não há código para auditar |
| `estagio: maduro` ou `legado` | `[[DOCUMENTACAO]]` roda em modo reorganização; o código é a fonte da verdade |
| `equipe: solo` | Cabeçalho de documento dispensa o campo `Owner` |
| `tem_interface: nao` | Revisão de linguagem de interface do `[[GATE_CODE]]` fica `N/A`, e `[[GATE_FRONTEND]]` não entra no roteiro |
| `interface_web: nao` | `[[GATE_FRONTEND]]` não entra no roteiro — interface que não roda em navegador não tem SEO nem Core Web Vitals |
| `indexavel: nao` | `[[GATE_FRONTEND]]` roda sem as seções de descoberta (5, 6, 7); desempenho, responsividade e acessibilidade continuam valendo |

Apresente o roteiro ao usuário **antes de executar**, no formato:

```text
ROTEIRO
 1. [[DIRETRIZES]]    criar     ~
 2. [[LICENCA]]       criar     precisa de decisão sua
 3. [[REQUISITOS]]    criar     entrevista
 ...
 PULADO: [[DIR_API]]  projeto não expõe API

AO FINAL
 skills               <as escolhidas, ou nenhuma>
 instalar             <prompts que persistem> -> [[DIR_GOVERNANCA]]
 remover              .genesis/
```

Na mesma mensagem, pergunte quais **skills opcionais** instalar — todas,
algumas ou nenhuma. Liste-as com uma linha de descrição cada e marque as que o
perfil recomenda:

| Skill | Comando | Recomendada quando |
| --- | --- | --- |
| `frontend-design` | `npx skills add https://github.com/anthropics/skills --skill frontend-design` | `interface_web: sim` |
| `ui-ux-pro-max` | `npx skills add https://github.com/nextlevelbuilder/ui-ux-pro-max-skill --skill ui-ux-pro-max` | `tem_interface: sim` |
| `polish` | `npx skills add https://github.com/pbakaus/impeccable --skill polish` | `tem_interface: sim` |
| `canvas-design` | `npx skills add https://github.com/anthropics/skills --skill canvas-design` | o projeto produz peça visual estática |
| `codebase-design` | `npx skills add https://github.com/mattpocock/skills --skill codebase-design` | sempre |
| `diagnosing-bugs` | `npx skills add https://github.com/mattpocock/skills --skill diagnosing-bugs` | sempre |

Tire a descrição de cada uma da própria fonte — `npx skills add <repo> --list`
lista o que o repositório oferece —, não de memória.

Peça confirmação. O usuário pode cortar etapas — respeite o corte e registre-o.
A remoção de `.genesis/` também pode ser cortada: nesse caso, a Etapa 5 instala
o kit mesmo assim e só pula a remoção.

---

## Etapa 3 — Execução

Execute cada prompt do roteiro, em ordem, um de cada vez.

Regras de delegação:

1. **Carregue o prompt inteiro antes de agir.** Não trabalhe de memória sobre o
   que ele pede.
2. **Passe o contexto acumulado.** Cada prompt recebe o perfil e os artefatos já
   produzidos. Arquitetura sem os requisitos em mãos é adivinhação.
3. **Um prompt por vez.** Não paralelize etapas que dependem uma da outra.
4. **Pare em decisão de dono.** Licença, arquitetura e escopo são decisões do
   usuário. Levante alternativas com trade-offs, recomende uma, e espere.
   Não decida por ele e siga.
5. **Ao terminar cada etapa, marque o roteiro** e diga em uma linha o que foi
   produzido.

Se um prompt falhar ou ficar bloqueado, não pule silenciosamente: registre o
bloqueio, conclua as etapas independentes e reporte o que ficou de fora.

---

## Etapa 4 — Costura

Terminadas as etapas, os documentos existem mas ainda não se conhecem. Feche o
grafo:

1. **Referências cruzadas.** Todo requisito não funcional relevante citado em
   `[[DIRETRIZES]]` aponta para `[[DOC_REQUISITOS_NF]]`. Toda decisão citada em
   `[[DOC_ARQUITETURA]]` aponta para o ADR correspondente em `[[DIR_ADR]]`.
2. **Índice.** `[[DOC_INDICE]]` lista todo artefato existente, sem exceção.
3. **Validação de links.** Todo link relativo escrito resolve para arquivo que
   existe. Link quebrado em documento recém-criado é falha desta etapa.
4. **Manifesto conferido.** Todo artefato produzido está na tabela do
   `MANIFEST.md` com o caminho real. Todo caminho da tabela existe ou está
   marcado como não aplicável.
5. **Entrada inaugural.** `[[DOC_CHANGELOG]]` recebe a entrada que registra a
   criação da estrutura.
6. **Descoberta.** `[[DOC_AGENTES]]` reflete os artefatos que realmente
   existem ao final — é gerado por último, depois que o resto está no lugar,
   porque aponta para eles.

---

## Etapa 5 — Instalação e remoção

A estrutura está pronta; `.genesis/` é andaime e sai. Antes, o que o ciclo de
trabalho continua usando muda para o repositório do projeto.

### 5.1 Skills

Se o usuário escolheu alguma, instale cada uma no escopo do projeto, sem
prompt interativo e copiando em vez de criar link simbólico:

```text
npx skills add <repo> --skill <nome> -y --copy
```

Rode uma por vez e confira a saída. Falha — sem rede, sem Node, repositório
fora do ar — não bloqueia o resto: registre em PENDÊNCIAS com o comando exato
para o usuário repetir.

### 5.2 Instalação do kit

1. Copie para `[[DIR_GOVERNANCA]]` todo prompt com **Persiste** `sim` no
   `MANIFEST.md`, mais os condicionais cuja condição o perfil satisfaz.
   Mantenha o nome do arquivo; os de `tools/` vão para a raiz da pasta.
2. Gere `[[DIR_GOVERNANCA]]/MANIFEST.md` como descrito na seção **Depois da
   montagem** do manifesto: caminhos reescritos, linhas que não persistem
   removidas, perfil e artefatos preenchidos.
3. Todo ponteiro gerado — `[[DOC_AGENTES]]`, `[[DOC_DIRETRIZES]]`, comandos de
   barra, regras de ferramenta — cita caminhos de `[[DIR_GOVERNANCA]]`.
   [[AGENTES]] e [[DIRETRIZES]] já escrevem assim; aqui se confere.

### 5.3 Verificação

Nada é removido antes de passar nas três:

- todo arquivo do item 1 existe em `[[DIR_GOVERNANCA]]`;
- todo caminho da tabela do manifesto instalado existe em disco, ou está
  marcado como não aplicável;
- busca por `.genesis/` em todos os arquivos do projeto **fora** de `.genesis/`
  não retorna nada. Ocorrência encontrada é ponteiro que vai quebrar: corrija
  para o caminho instalado.

Se algum item falhar e não puder ser corrigido, **não remova**. Registre o
motivo em PENDÊNCIAS.

### 5.4 Remoção

Apague `.genesis/` inteira. Não faça commit: a remoção entra na alteração que
o usuário vai revisar junto com o resto.

---

## Etapa 6 — Relatório

```text
GENESIS
ETAPAS EXECUTADAS: N / M

CRIADOS
  caminho — uma linha sobre o que responde

ATUALIZADOS
  caminho — o que mudou

PULADOS
  id — por quê

INSTALADOS EM [[DIR_GOVERNANCA]]
  arquivos
SKILLS
  instaladas | falharam (com o comando) | nenhuma escolhida
.genesis/
  removida | mantida — por quê

DECISÕES DO USUÁRIO
  o que foi decidido, e onde está registrado

PENDÊNCIAS
  o que ficou aberto e o que destrava

PRÓXIMO PASSO
```

---

## Reentrância

Este prompt é feito para rodar mais de uma vez no mesmo projeto. Em execução
subsequente:

- a Etapa 0 detecta o que já existe e o roteiro só cobre as lacunas;
- documentos existentes são **atualizados**, nunca sobrescritos em bloco;
- conteúdo escrito à mão pelo usuário é preservado — na dúvida, pergunte antes
  de reescrever um parágrafo que você não gerou;
- como `.genesis/` foi removida ao fim da execução anterior, o usuário a copia
  de novo; o perfil vem do manifesto em `[[DIR_GOVERNANCA]]`, e a Etapa 5
  atualiza os prompts instalados com a versão nova do pacote — prompt editado
  à mão no projeto é mostrado em diff antes de ser substituído.

Rodar o Genesis duas vezes não pode destruir trabalho feito entre as duas.

---

## Depois da montagem

Este prompt monta a estrutura. O ciclo de trabalho que vem depois não passa
mais por ele — está em `[[DIRETRIZES]]`, e roda a cada fase de implementação:

```text
[[GATE_CODE]]  ->  [[DOCUMENTACAO]] incremental, se a § 21.2 acusar
               ->  [[GATE_CODE]] de novo
               ->  [[GATE_SECURITY]]
               ->  [[GATE_FRONTEND]], se a fase tocou a interface web
               ->  [[GATE_API]], se a fase tocou a API
               ->  [[DOC_CHANGELOG]]
```

Esses prompts rodam a partir de `[[DIR_GOVERNANCA]]`, depois que `.genesis/`
sai. Chame o orquestrador de novo — copiando `.genesis/` outra vez — apenas
quando a estrutura em si mudar: artefato novo, projeto que passou de
`bootstrap` a `em-desenvolvimento`, arquitetura revista.
