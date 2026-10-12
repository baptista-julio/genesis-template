# Quality Gate — Código

> Prompt do pacote Genesis. Referências `[[ID]]` resolvem pelo `MANIFEST.md`.
> Último gate da fase, executado depois de `[[GATE_API]]`, `[[GATE_FRONTEND]]`,
> `[[GATE_INPUT]]` e `[[GATE_SECURITY]]` — os que se aplicam já retornaram PASS.
> Por vir por último, avalia também o código alterado pelas correções deles.
> A régua deste gate é calibrada por `[[DOC_REQUISITOS_NF]]` (categoria
> Manutenibilidade) e pelas fronteiras declaradas em `[[DOC_ARQUITETURA]]`.
> Quando o projeto definir limiar próprio, **o do projeto prevalece** sobre o
> valor padrão citado aqui — registre a divergência no relatório final.


Você acabou de concluir uma fase de implementação.

Antes de considerar a fase concluída, execute obrigatoriamente este Quality Gate sobre o código alterado.

Seu objetivo NÃO é defender a implementação realizada. Seu objetivo é atuar como um **Senior Software Engineer / Senior Code Quality Analyst independente**, tentando encontrar problemas no código que acabou de ser produzido.

Considere prioritariamente o **diff desta implementação**, mas também analise seus impactos sobre a arquitetura e o restante da codebase.

---

## REGRAS DE INDEPENDÊNCIA DA REVISÃO

Como você é o mesmo agente que implementou o código, aplique obrigatoriamente as seguintes salvaguardas contra viés de autoavaliação:

1. **Releia o diff completo do zero.** Obtenha o diff real via ferramenta (ex.: `git diff`, `git status`, `git log`) antes de qualquer avaliação. NÃO confie na sua memória da sessão sobre o que foi alterado — em sessões longas, alterações feitas no início tendem a ser esquecidas ou lembradas incorretamente.

2. **Toda métrica deve vir de execução real de ferramenta, com a saída citada no relatório.** Nunca estime, deduza ou "avalie mentalmente" uma métrica que possua ferramenta disponível. Isso vale para TODAS as verificações deste gate, não apenas o relatório final.

3. **Justificativas dadas durante a implementação NÃO valem no gate.** Se durante a implementação você decidiu que algo era aceitável ("esse acoplamento é ok porque...", "esse teste não é necessário porque..."), essa decisão não é evidência. Cada exceção precisa ser re-argumentada do zero no relatório, como se um revisor externo a estivesse questionando pela primeira vez.

4. **Procure ativamente por problemas.** A postura padrão é ceticismo: assuma que existem problemas até que as verificações demonstrem o contrário.

---

## PRINCÍPIO FUNDAMENTAL

Uma implementação somente pode ser considerada concluída quando:

1. estiver funcional;
2. estiver testada;
3. não introduzir regressões;
4. não deteriorar a arquitetura;
5. possuir complexidade aceitável;
6. permanecer compreensível e modificável;
7. não criar dívida técnica desnecessária.

Não avalie qualidade apenas por funcionamento.

Código que funciona mas degrada significativamente manutenibilidade, arquitetura ou testabilidade deve ser refatorado.

---

## CLASSIFICAÇÃO DE SEVERIDADE

Cada achado deve ser classificado como **BLOCKER** ou **WARNING**. A classificação determina o veredito:

### BLOCKER — reprova o gate automaticamente (STATUS = FAIL)

* erros de build, compilação, type checking ou lint introduzidos pela alteração;
* qualquer teste falhando;
* código novo com comportamento testável e sem nenhum teste;
* novo ciclo de dependência (sem justificativa arquitetural excepcional e explícita);
* nova violação de boundary arquitetural;
* vulnerabilidade crítica ou alta introduzida;
* secret hardcoded;
* breaking change não solicitada em contrato público;
* teste removido, enfraquecido ou skipado para passar no gate;
* suppression de lint/type sem justificativa;
* dead code ou código comentado introduzido em partes essenciais;
* TODO/FIXME/HACK/TEMP em partes essenciais da implementação;
* exceção mascarada ou catch vazio introduzido.

### WARNING — deve ser reportado e justificado, mas não reprova sozinho

* complexidade ciclomática entre 11 e 20;
* complexidade cognitiva acima de 15;
* aninhamento acima de 3 níveis;
* funções entre 51 e 80 LOC;
* arquivos acima de 400–600 LOC;
* duplicação de código novo acima de 3%;
* cobertura de código novo abaixo dos alvos;
* funções com mais de 5 parâmetros;
* warnings novos de build/lint;
* code smells de design (seção Revisão de Design);
* preocupações de acoplamento sem violação formal de boundary.

**Escalonamento:** um WARNING vira BLOCKER quando: (a) complexidade ciclomática > 20 sem justificativa técnica excepcional; (b) o mesmo tipo de warning se repete de forma sistemática na alteração, indicando problema estrutural; (c) o warning foi apontado em iteração anterior do gate e ignorado sem justificativa.

Todo WARNING não corrigido deve constar no relatório final com justificativa explícita de por que foi aceito.

---

# 1. ENTENDER A ALTERAÇÃO

Antes das métricas, determine:

* qual problema deveria ser resolvido;
* quais arquivos foram alterados;
* quais componentes foram adicionados;
* quais responsabilidades foram modificadas;
* quais contratos públicos foram alterados;
* se a superfície da mudança é proporcional ao problema.

Obtenha o diff completo via ferramenta e analise-o na íntegra (ver Regras de Independência).

Verifique também se a implementação **atende ao requisito funcional da tarefa** — não apenas se "funciona", mas se resolve o problema que deveria resolver.

Procure alterações não relacionadas ao objetivo da implementação.

Não aceite complexidade incidental sem justificativa.

---

# 2. BUILD / COMPILAÇÃO

Execute todos os mecanismos disponíveis na codebase para verificar:

* build;
* compilação;
* type checking;
* lint.

Critérios (BLOCKER se violados):

BUILD_ERRORS = 0
TYPE_ERRORS = 0
LINT_ERRORS = 0

Warnings novos são WARNING: devem ser reportados e, quando triviais de corrigir, corrigidos.

Não ignore erros existentes sem distinguir claramente problemas preexistentes de problemas introduzidos pela alteração. Apenas problemas introduzidos pela alteração contam para o veredito.

---

# 3. TESTES

Execute a suíte de testes aplicável.

Critério (BLOCKER):

FAILED_TESTS = 0

Código novo ou modificado deve possuir testes adequados quando houver comportamento testável. Ausência total de testes para comportamento testável novo é BLOCKER.

Verifique (WARNING se abaixo do alvo):

LINE_COVERAGE_NEW_CODE >= 90%
BRANCH_COVERAGE_NEW_CODE >= 85%

Se a codebase não tiver tooling de cobertura por diff, tente ferramentas leves (ex.: `diff-cover` sobre o relatório de cobertura padrão) antes de declarar NOT_MEASURED.

Não considere cobertura alta como evidência suficiente de bons testes.

Analise também se os testes realmente validam comportamento, edge cases, caminhos alternativos e falhas. Testes que apenas exercitam código sem assertions significativas devem ser reportados como WARNING de design.

É proibido remover, enfraquecer, ignorar ou modificar testes apenas para permitir que a implementação passe pelo gate (BLOCKER).

---

# 4. COMPLEXIDADE CICLOMÁTICA

Analise funções e métodos modificados. Use ferramenta quando disponível (ex.: `lizard`, `radon`, regras do lint da codebase); tente instalar uma ferramenta leve antes de declarar NOT_MEASURED.

Referência:

1–5 = simples
6–10 = aceitável
11–15 = WARNING — requer análise
16–20 = WARNING forte — candidato obrigatório a refatoração
> 20 = BLOCKER, salvo justificativa técnica excepcional e explícita

Quando a complexidade estiver elevada, procure:

* múltiplas responsabilidades;
* branches excessivos;
* regras de negócio misturadas;
* abstrações ausentes;
* lógica duplicada.

Não extraia funções artificialmente apenas para reduzir a métrica.

---

# 5. COMPLEXIDADE COGNITIVA

Avalie quão difícil é compreender cada função modificada.

Referência (WARNING se acima):

COGNITIVE_COMPLEXITY <= 15

Considere particularmente:

* loops aninhados;
* condicionais aninhadas;
* múltiplas bifurcações;
* controle de fluxo difícil de acompanhar.

---

# 6. PROFUNDIDADE DE ANINHAMENTO

Evite profundidade superior a 3 níveis (WARNING se acima).

Quando apropriado, utilize:

* early return;
* guard clauses;
* extração de responsabilidades;
* decomposição da regra;
* estruturas de dados adequadas.

Não utilize early return mecanicamente caso torne o fluxo menos compreensível.

---

# 7. TAMANHO E RESPONSABILIDADE DAS FUNÇÕES

Utilize tamanho apenas como indicador.

Referência:

<=30 LOC = bom
31–50 LOC = aceitável
51–80 LOC = WARNING — revisar
> 80 LOC = WARNING forte — candidato à refatoração

A principal pergunta deve ser:

"Esta função possui uma única responsabilidade conceitual?"

Não fragmente código artificialmente em dezenas de funções pequenas sem ganho de compreensão.

---

# 8. TAMANHO E RESPONSABILIDADE DOS MÓDULOS

Arquivos acima de aproximadamente 400–600 LOC devem receber análise adicional (WARNING).

Não reprovar exclusivamente por tamanho.

Investigue se o módulo:

* possui responsabilidades diferentes;
* conhece componentes demais;
* concentra regras não relacionadas;
* deveria ser separado.

---

# 9. DUPLICAÇÃO

Procure duplicação introduzida pela alteração. Use ferramenta quando disponível (ex.: `jscpd`); tente instalar antes de declarar NOT_MEASURED.

Objetivo (WARNING se acima):

DUPLICATION_NEW_CODE <= 3%

Analise também duplicações semânticas que ferramentas automáticas podem não detectar.

Antes de criar uma abstração, confirme que os trechos realmente representam o mesmo conceito e possuem o mesmo motivo para mudar.

Não aplique DRY de maneira mecânica.

---

# 10. COESÃO

Cada classe, função ou módulo deve possuir responsabilidades relacionadas.

Pergunte:

"Esses elementos mudariam normalmente pelo mesmo motivo?"

Identifique módulos que misturam, por exemplo:

* persistência;
* regra de negócio;
* integração externa;
* transformação;
* apresentação;
* logging;
* validação.

Quando responsabilidades independentes estiverem misturadas, proponha separação (WARNING de design).

---

# 11. ACOPLAMENTO

Avalie dependências introduzidas.

Identifique:

* dependências desnecessárias;
* acesso direto entre camadas que deveriam permanecer isoladas;
* componentes que conhecem detalhes internos de outros componentes;
* excesso de fan-out;
* dependência de implementações concretas quando deveria existir contrato.

Não introduza abstrações sem necessidade concreta.

---

# 12. DEPENDÊNCIAS CIRCULARES

Execute análise de dependências quando houver ferramenta disponível (ex.: `madge`, `dpdm`, `pydeps`, arch tests da codebase); tente instalar uma ferramenta leve antes de declarar NOT_MEASURED.

Critério (BLOCKER):

NEW_DEPENDENCY_CYCLES = 0

Qualquer novo ciclo deve reprovar o gate, salvo justificativa arquitetural excepcional e explícita.

## 12.1 Mapa de módulos

*Aplicável quando `[[DOC_MAPA]]` existir. Caso contrário, `N/A`.*

A passagem acima já percorreu o grafo de dependências. Aproveite-a para
verificar o mapa, em vez de descartá-la.

Se o diff alterou import, adicionou ou removeu arquivo, ou mexeu em dependência
externa, `[[DOC_MAPA]]` precisa ter sido atualizado na mesma alteração.

**WARNING:**

* grafo mudou e o mapa não foi atualizado;
* carimbo de validade do mapa muito anterior ao estado atual do código.

**WARNING forte:**

* mapa contradiz o código — aresta que não existe mais, arquivo listado que foi
  removido, dependência externa que saiu do projeto. Mapa que mente induz ao
  erro e custa mais que mapa ausente.

Não é BLOCKER: o mapa é ferramenta de navegação, não contrato. Reprovar a fase
por causa dele seria desproporcional. WARNING reincidente escala pela regra
geral.

**Correção:** execute `[[MAPA_MODULOS]]` em modo incremental sobre os arquivos
do diff.

---

# 13. BOUNDARIES ARQUITETURAIS

Descubra as regras arquiteturais existentes na codebase antes de avaliar.

Verifique se a implementação respeita:

* módulos;
* layers;
* packages;
* domain boundaries;
* interfaces;
* direction of dependencies.

Exemplos de violações:

controller acessando persistence diretamente quando existe application/service layer;

domain dependendo de infrastructure;

módulo acessando detalhes internos de outro módulo;

feature importando arquivos privados de outra feature.

Critério (BLOCKER):

NEW_ARCHITECTURE_VIOLATIONS = 0

---

# 14. PARÂMETROS E ASSINATURAS

Funções com mais de aproximadamente 5 parâmetros devem ser investigadas (WARNING).

Considere quando apropriado:

* parameter object;
* command;
* DTO;
* value object.

Não crie objetos artificiais apenas para satisfazer a métrica.

---

# 15. DEAD CODE

Detecte código introduzido que não possui uso real.

Critérios (BLOCKER quando em partes essenciais; WARNING em periferia):

NEW_UNUSED_IMPORTS = 0
NEW_DEAD_CODE = 0

Procure também:

* variáveis não utilizadas;
* métodos obsoletos;
* caminhos impossíveis;
* implementação antiga mantida desnecessariamente;
* código comentado.

---

# 16. TODO / FIXME / HACK

Reporte qualquer novo:

TODO
FIXME
HACK
TEMP

Placeholders relacionados a partes essenciais da implementação são BLOCKER. Anotações periféricas legítimas devem ser reportadas como WARNING com justificativa.

---

# 17. TYPE SAFETY

Quando aplicável, execute o type checker oficial da codebase.

Critério (BLOCKER):

TYPE_ERRORS = 0

Investigue especialmente atalhos utilizados para silenciar o sistema de tipos, como:

any
cast desnecessário
type assertion insegura
suppression
ignore

Suppressions introduzidas sem justificativa explícita são BLOCKER. Não utilize esses mecanismos apenas para fazer o código compilar.

---

# 18. TRATAMENTO DE ERROS

Procure:

* exceptions ignoradas;
* catch vazio;
* erros transformados silenciosamente em null;
* falhas externas sem tratamento;
* recursos sem cleanup;
* ausência de timeout quando aplicável;
* mensagens de erro sem contexto suficiente.

Catch vazio e mascaramento de exceção introduzidos são BLOCKER. Os demais itens são WARNING conforme criticidade.

Não esconda erros para permitir que o fluxo continue aparentemente funcionando.

---

# 19. SEGURANÇA

Quando existirem ferramentas disponíveis, execute verificações de segurança (ex.: `npm audit`, `pip-audit`, linters de segurança, semgrep); tente instalar uma ferramenta leve antes de declarar NOT_MEASURED.

Critérios (BLOCKER):

NEW_CRITICAL_VULNERABILITIES = 0
NEW_HIGH_VULNERABILITIES = 0
NEW_HARDCODED_SECRETS = 0

Investigue também:

* injection;
* SQL inseguro;
* command injection;
* path traversal;
* exposição de dados;
* ausência de validação;
* autorização;
* autenticação;
* serialização insegura.

O tratamento sistemático de entrada — validação no servidor, limites, vetores
por tecnologia — é avaliado por `[[GATE_INPUT]]`, que roda antes deste gate.

---

# 20. CONTRATOS E COMPATIBILIDADE

Verifique se foram alterados:

* APIs;
* interfaces;
* DTOs;
* eventos;
* schemas;
* banco;
* métodos públicos;
* formatos de entrada/saída.

Breaking changes não solicitadas são BLOCKER.

Não altere contratos existentes silenciosamente.

Esta seção pergunta **se** o contrato mudou. Se o que mudou foi a API pública,
o **design** dessa mudança — nomenclatura, método, status, paginação,
versionamento, consistência com os demais endpoints — é avaliado por
`[[GATE_API]]`, que roda antes deste gate.

---

# 21. DOCUMENTAÇÃO

## 21.1 Documentação no código

Não avalie documentação por quantidade de comentários.

Avalie se estão documentados adequadamente:

* APIs públicas;
* interfaces;
* contratos;
* algoritmos não triviais;
* decisões arquiteturais relevantes;
* regras de negócio não óbvias.

Não adicione comentários que apenas narram o código.

Prefira código autoexplicativo.

## 21.2 Sincronia com a documentação do projeto

*Aplicável quando o projeto tem documentação estruturada. Caso contrário,
`N/A`.*

`[[DOCUMENTACAO]]` estabelece que nenhuma funcionalidade está concluída
enquanto a documentação relacionada não refletir a implementação. Esta seção é
o que verifica a regra — sem ela, a obrigação é intenção sem consequência, e
documentação desatualizada é pior que ausente: tem aparência de verdade.

### Como verificar

1. **Liste os arquivos de código do diff.** Já obtidos na seção 1.

2. **Encontre os documentos que os reivindicam.** Documentos críticos declaram
   no cabeçalho de metadados:

   ```yaml
   Implementação:
     - src/importacao/
   ```

   Busque por esse campo em toda a documentação e monte o mapa
   `caminho de código -> documento`. Um documento cujo `Implementação:` casa
   com um arquivo do diff é candidato a estar desatualizado.

3. **Classifique cada mudança.** Nem toda alteração de código exige alteração
   de documento:

   | Mudança | Documentação |
   | --- | --- |
   | Comportamento observável novo ou alterado | Obrigatória |
   | Contrato público: API, DTO, evento, esquema | Obrigatória |
   | Regra de negócio | Obrigatória |
   | Fronteira arquitetural, dependência entre módulos | Obrigatória, mais ADR |
   | Refatoração sem mudança de comportamento | Dispensada |
   | Correção que restaura o comportamento documentado | Dispensada |
   | Teste, formatação, renomeação interna | Dispensada |

   A pergunta que decide: **o documento continua verdadeiro depois desta
   alteração?** Se sim, dispensada — mesmo que o código tenha mudado muito.

4. **Compare.** Para cada mudança de documentação obrigatória, verifique se o
   documento correspondente foi alterado no mesmo diff.

5. **Verifique os campos do cabeçalho.** Documento tocado precisa ter
   `Atualizado em` e `Versão` coerentes com a alteração. Campo `Status` ainda
   `Proposto` em funcionalidade que passou a existir é achado.

### Critérios

```text
CODIGO_COM_COMPORTAMENTO_NOVO_SEM_DOC = 0
CONTRATO_PUBLICO_ALTERADO_SEM_DOC = 0
DOC_IMPLEMENTADO_DIVERGENTE_DO_CODIGO = 0
```

**BLOCKER:**

* comportamento observável novo sem documentação correspondente;
* contrato público alterado — API, DTO, evento, esquema, migração de dados —
  sem atualizar o documento que o descreve;
* documento com status `Implementado` que passou a descrever algo que o código
  não faz mais. A divergência é defeito, não dívida: quem lê é induzido ao
  erro, e a documentação inteira perde crédito;
* decisão arquitetural tomada na fase sem ADR correspondente
  (ver `[[ADR]]` para o critério do que merece registro).

**WARNING:**

* documento alterado sem atualizar `Atualizado em` ou `Versão`;
* campo `Implementação:` apontando para caminho que não existe mais;
* link interno quebrado introduzido pela alteração;
* documentação existe mas ficou incompleta para o caso novo;
* termo novo relevante ausente de `[[DOC_GLOSSARIO]]`.

### Evidência

Liste o mapa que você montou, mesmo quando não houver achado:

```text
src/importacao/leitor.rs  -> docs/spec/importacao/formatos.md   atualizado
src/importacao/mapa.rs    -> docs/spec/importacao/pipeline.md   NAO atualizado
src/util/texto.rs         -> nenhum documento reivindica        refatoração
```

Nenhum documento reivindicar um arquivo não é necessariamente achado — código
interno não precisa de documento próprio. Passa a ser achado quando o arquivo
implementa comportamento que alguém consome.

### Correção

Achado aqui **não se resolve no gate**. Execute `[[DOCUMENTACAO]]` em modo
incremental sobre os documentos apontados, depois re-execute esta seção.

Não escreva a documentação faltante de dentro do gate: o gate mede, não
produz. Misturar as duas coisas faz o gate virar juiz do próprio trabalho, que
é o viés que as Regras de Independência existem para evitar.

---

# 22. REVISÃO DE DESIGN

Realize uma revisão qualitativa adicional procurando:

* God Classes;
* God Functions;
* Feature Envy;
* Shotgun Surgery;
* Primitive Obsession;
* duplicação;
* abstrações prematuras;
* overengineering;
* responsabilidades misturadas;
* nomes ambíguos;
* dependências ocultas;
* estado global desnecessário;
* efeitos colaterais inesperados;
* problemas óbvios de performance: N+1 queries, queries dentro de loops, IO repetido desnecessariamente, alocações desnecessárias em hot paths.

A ausência de violações de métricas não significa automaticamente que o design está bom.

## 22.1 Linguagem da interface

*Aplicável quando `tem_interface: sim` no perfil. Caso contrário, `N/A`.*

Revise as strings visíveis ao usuário introduzidas ou alteradas na fase —
rótulo, placeholder, texto de ajuda, mensagem de erro e de aviso, título de
etapa, nome de opção — contra a decisão de linguagem registrada em
`[[DIR_ADR]]`, se houver. Na ausência de ADR específica, use o critério geral
abaixo.

O critério é uma pergunta:

> Uma pessoa que não conhece a implementação entende o que vai acontecer?

Não há checagem automática. A busca por termo literal (`JOIN`, `schema`,
`índice`, `null`, `timeout`, `parse`) pega o caso óbvio, mas não pega o jargão
travestido: "chave de ligação" não contém nenhum termo proibido e é igualmente
opaco. Leia as strings novas como quem não conhece o modelo de dados por trás.

Achados aqui são WARNING, salvo quando a string aparece em fluxo de erro
crítico — mensagem que a pessoa lê justamente quando já está com problema, e
que precisa dizer o que fazer a seguir.

Marcação, desempenho, indexabilidade e acessibilidade da interface **não são
avaliados aqui** — são o escopo do `[[GATE_FRONTEND]]`, que roda antes deste
gate quando `interface_web: sim`. Este gate olha o código; aquele olha
o que o navegador recebeu.

---

# 23. CONDICIONAIS

Não substitua automaticamente switch ou if/else por classes.

Considere Strategy, State, polymorphism, dispatch tables ou abstrações equivalentes quando:

* a condicional representa diferentes comportamentos;
* novos comportamentos tendem a aparecer;
* a mesma condicional está espalhada pela codebase;
* cada branch possui lógica substancial;
* diferentes regras possuem motivos independentes para mudar.

Condicionais simples e explícitas devem permanecer simples.

Evite overengineering.

---

# 24. REFATORAÇÃO

Caso alguma verificação falhe, primeiro explique objetivamente:

VIOLAÇÃO
SEVERIDADE (BLOCKER | WARNING)
ARQUIVO
LOCAL
MÉTRICA
VALOR ATUAL
VALOR ESPERADO
CAUSA PROVÁVEL

Depois refatore.

Possíveis estratégias incluem:

* extrair responsabilidades;
* decompor funções grandes;
* utilizar guard clauses;
* utilizar early returns;
* reduzir nesting;
* eliminar duplicação;
* melhorar nomes;
* reduzir acoplamento;
* quebrar ciclos de dependência;
* introduzir interfaces apenas quando houver necessidade arquitetural;
* utilizar Strategy/State/polimorfismo quando existirem comportamentos variantes;
* separar regra de negócio de IO;
* separar domínio de infraestrutura.

Preserve o comportamento externo da aplicação salvo quando a mudança funcional fizer parte explicitamente da tarefa.

---

# 25. PROIBIÇÕES

Durante a refatoração é proibido:

* remover testes para fazer o gate passar;
* enfraquecer assertions;
* marcar testes como skip;
* reduzir thresholds;
* desabilitar lint;
* desabilitar type checking;
* adicionar suppressions sem justificativa;
* ignorar vulnerabilidades;
* remover architecture tests;
* mascarar exceções;
* alterar comportamento sem necessidade;
* realizar grandes refatorações não relacionadas à tarefa;
* criar abstrações artificiais exclusivamente para melhorar métricas.

Qualquer violação destas proibições é BLOCKER e deve ser revertida.

---

# 26. RE-EXECUÇÃO E LIMITE DE ITERAÇÕES

Depois de qualquer refatoração:

execute novamente todo o conjunto de verificações afetado.

Nunca considere uma refatoração correta apenas por inspeção visual.

**Limite de iterações:**

MAX_GATE_ITERATIONS = 3

Uma iteração = ciclo completo de (verificar → refatorar → re-verificar).

Se após 3 iterações ainda existirem BLOCKERs:

1. PARE de refatorar;
2. emita STATUS = FAIL;
3. liste os BLOCKERs remanescentes com diagnóstico detalhado;
4. explique o que foi tentado em cada iteração e por que não resolveu;
5. recomende o próximo passo (ex.: redesenho da abordagem, decisão do usuário, quebra da tarefa).

Não entre em loop tentando indefinidamente. Falhar com diagnóstico claro é melhor do que iterar sem convergência ou do que rebaixar critérios para passar.

---

# 27. RESULTADO FINAL

Ao finalizar, produza obrigatoriamente:

## QUALITY GATE

STATUS: PASS | FAIL
ITERAÇÕES EXECUTADAS: N / 3

### EVIDÊNCIAS

Liste cada comando/ferramenta executada com resumo da saída relevante (exit code, contagem de erros, valor da métrica). Toda métrica reportada abaixo deve ter uma evidência correspondente nesta seção. Métrica sem evidência de execução = NOT_MEASURED.

### CORRETUDE

Build:
Typecheck:
Lint:
Tests:

### TESTABILIDADE

Line coverage (código novo):
Branch coverage (código novo):
Qualidade das assertions:

### COMPLEXIDADE

Cyclomatic complexity:
Cognitive complexity:
Max nesting:

### MANUTENIBILIDADE

Duplicação:
Funções problemáticas:
Arquivos problemáticos:
Dead code:

### ARQUITETURA

Dependency cycles:
Boundary violations:
Coupling concerns:

### DOCUMENTAÇÃO

Mapa de módulos:
Mapa código -> documento:
Comportamento novo sem doc:
Contrato alterado sem doc:
Documento Implementado divergente:
ADR faltante:

### SEGURANÇA

Critical:
High:
Secrets:
Outros:

### DESIGN REVIEW

Problemas encontrados:
Refatorações realizadas:

### BLOCKERS

Lista de BLOCKERs encontrados e status de cada um (resolvido | remanescente).

### WARNINGS ACEITOS

Lista de WARNINGs não corrigidos, cada um com justificativa explícita re-argumentada do zero (justificativas da fase de implementação não são válidas aqui).

### RISCO RESIDUAL

Descreva qualquer risco ou ponto que não pôde ser validado automaticamente.

### VEREDITO

A fase somente pode ser marcada como concluída quando:

STATUS = PASS

STATUS = PASS exige: zero BLOCKERs remanescentes E todos os WARNINGs aceitos possuem justificativa registrada.

Nunca invente métricas.

Quando determinada métrica não puder ser medida com as ferramentas disponíveis, primeiro tente instalar/executar uma ferramenta leve adequada (com tempo limitado — não gaste a sessão nisso). Somente então informe:

NOT_MEASURED

e explique o motivo.

Não substitua medição real por estimativa da LLM.