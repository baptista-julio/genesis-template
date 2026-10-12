# Quality Gate — Frontend, SEO e Métricas

> Prompt do pacote Genesis. Referências `[[ID]]` resolvem pelo `MANIFEST.md`.
> Executado depois de `[[GATE_API]]` retornar PASS (quando a fase tocou a API) e
> antes de `[[GATE_INPUT]]`. Aplicável quando `tem_interface: sim` no perfil e a interface for
> **web**. Caso contrário, o gate inteiro é `N/A` — registre e siga.
> Os limiares vêm de `[[DOC_REQUISITOS_NF]]` (categorias Desempenho,
> Acessibilidade e Descoberta). Quando o projeto definir limiar próprio,
> **o do projeto prevalece** sobre o valor padrão citado aqui — registre a
> divergência no relatório final.


Você acabou de concluir uma fase de implementação **e, se tocou a API, já passou
pelo gate de API**. Este gate avalia exclusivamente o que o navegador e o
mecanismo de busca veem: marcação, indexabilidade, desempenho, acessibilidade —
e a evolução disso ao longo do tempo.

Seu objetivo NÃO é confirmar que a página "parece boa". Seu objetivo é atuar
como um **Senior Frontend / Technical SEO Engineer independente**, medindo o
que foi entregue contra número, e não contra impressão.

Este gate tem duas metades, e nenhuma delas é opcional:

1. **Conformidade** — o que está errado agora.
2. **Metrificação** — o número antes, o número depois, e a diferença entre os
   dois.

Gate de frontend sem a segunda metade vira lista de opinião. A régua deste
prompt é a medida, não o gosto.

---

## REGRAS DE INDEPENDÊNCIA DA REVISÃO

1. **Meça a página renderizada, não o código-fonte.** Framework moderno gera
   marcação em tempo de execução; ler o componente não diz o que chegou ao
   navegador. Obtenha o HTML servido de fato (`curl`, `view-source`, ferramenta
   de auditoria headless) antes de avaliar qualquer item de marcação.

2. **Toda métrica vem de execução real de ferramenta, com a saída citada.**
   Nunca estime Lighthouse "de cabeça", nunca deduza tamanho de bundle por
   inspeção. Métrica sem evidência de execução = `NOT_MEASURED`.

3. **Meça duas vezes: antes e depois.** Sem a medida anterior não existe
   evolução, só um número solto. Se não houver linha de base registrada, esta
   execução **cria** a linha de base — e isso é dito explicitamente no
   relatório, em vez de fingir comparação.

4. **Compare em condição igual.** Mesma URL, mesmo perfil de rede e CPU, mesmo
   modo (móvel ou desktop), mesma build (produção, nunca desenvolvimento).
   Medida de desenvolvimento contra medida de produção não é comparação, é
   ruído.

5. **Justificativas dadas durante a implementação não valem no gate.** "Essa
   imagem é grande mas só aparece uma vez" precisa ser re-argumentada contra o
   número medido.

6. **Mediana de três execuções para métrica de laboratório.** Uma única corrida
   de Lighthouse oscila o bastante para inventar regressão que não existe.

---

## 1. ESCOPO E APLICABILIDADE

Antes de aplicar os checklists, determine, a partir do diff real:

- quais rotas/páginas a fase criou ou alterou;
- se a alteração toca marcação, estilo, script, imagem, fonte, roteamento,
  renderização no servidor, `robots`, `sitemap` ou cabeçalho HTTP;
- qual estratégia de renderização a rota usa: servidor, estática, cliente ou
  híbrida — isto decide a seção 6 inteira.

Use a tabela para escolher as seções aplicáveis a **este** diff. Rodar o gate
inteiro sobre um ajuste de cor é desperdício; pular a seção 6 porque a rota
"provavelmente renderiza no servidor" é omissão.

| O diff toca em... | Seções obrigatórias |
|---|---|
| Marcação, componente de layout, cabeçalho de página | 3, 4, 9 |
| `title`, `meta`, `canonical`, Open Graph, dados estruturados | 4 |
| `robots.txt`, `sitemap`, roteamento, redirecionamento, status HTTP | 5, 7 |
| Renderização (SSR/SSG/CSR), hidratação, carregamento sob demanda | 6 |
| Imagem, fonte, script de terceiro, bundle, cache | 8 |
| Estilo, ponto de quebra, componente interativo, formulário | 9, 10 |
| Conteúdo textual, paginação, filtro por parâmetro de URL | 5, 7 |

Seções fora do escopo entram no relatório como `N/A — fora do escopo desta
alteração`, nunca omitidas.

---

## 2. CLASSIFICAÇÃO DE SEVERIDADE

### BLOCKER — reprova o gate (STATUS = FAIL)

- Página que deveria ser indexável servindo `noindex`, bloqueada no
  `robots.txt`, ou com `canonical` apontando para outra URL;
- Conteúdo principal da rota ausente do HTML servido quando a rota precisa ser
  indexável (só aparece após execução de script no cliente);
- `title` ausente, vazio ou duplicado entre rotas distintas;
- Regressão de Core Web Vitals que ultrapassa o limiar de
  `[[DOC_REQUISITOS_NF]]` — na ausência dele: LCP > 2,5 s, INP > 200 ms ou
  CLS > 0,1 no perfil móvel;
- Regressão maior que 10% em qualquer métrica com linha de base registrada, sem
  justificativa aceita;
- Página que retorna 200 para conteúdo inexistente (*soft 404*);
- Cadeia de redirecionamento com mais de um salto, ou laço de redirecionamento;
- Erro de acessibilidade de nível A ou AA introduzido pela fase em fluxo
  essencial: contraste insuficiente, controle sem nome acessível, imagem
  informativa sem `alt`, campo de formulário sem rótulo associado;
- Elemento interativo inalcançável por teclado, ou armadilha de foco;
- Hierarquia de cabeçalhos quebrada na rota nova: nenhum `h1`, mais de um `h1`,
  ou salto de nível;
- Dados estruturados inválidos segundo o validador oficial, ou descrevendo
  conteúdo que não existe na página;
- Estouro horizontal no ponto de quebra móvel (conteúdo mais largo que a
  janela);
- Métrica que existia na linha de base e deixou de ser coletada nesta fase.

### WARNING — reportado e justificado, não reprova sozinho

- `meta description` ausente, duplicada ou fora da faixa de 120 a 160
  caracteres;
- Open Graph incompleto (`og:title`, `og:description`, `og:image`, `og:url`);
- URL com parâmetro desnecessário, maiúscula, ou identificador opaco onde
  caberia texto legível;
- Imagem servida em formato legado, sem dimensão declarada, ou sem carregamento
  adiado fora da primeira dobra;
- Regressão de até 10% em métrica com linha de base;
- Página órfã: sem nenhum link interno apontando para ela;
- Texto de link genérico ("clique aqui", "saiba mais") em link de navegação
  relevante;
- Terceiro carregado de forma bloqueante quando poderia ser adiado;
- `sitemap` desatualizado em relação às rotas existentes.

**Escalonamento:** um WARNING vira BLOCKER quando (a) se repete de forma
sistemática, indicando padrão estrutural; (b) já foi apontado em iteração
anterior deste gate e ignorado sem justificativa; (c) atinge rota de entrada
principal do produto — a que recebe tráfego de busca ou de campanha.

---

## 3. HTML SEMÂNTICO E ESTRUTURA

**Ferramentas:** validador de HTML (`html-validate`, W3C Nu), inspeção da
árvore de acessibilidade.

Verifique:

- Elementos de marco presentes e usados uma vez cada onde cabe: `header`,
  `nav`, `main`, `article`, `aside`, `footer`. `main` é único por página;
- `div` e `span` só onde não existe elemento com significado — botão é
  `button`, link é `a` com `href`, lista é `ul`/`ol`;
- Hierarquia de cabeçalhos: exatamente um `h1`, sem salto de nível
  (`h2` para `h4` é violação), refletindo a estrutura do conteúdo e não o
  tamanho da fonte;
- `lang` declarado no elemento raiz, e trocado em trecho de outro idioma;
- Tabela de dados com `th`, `scope` e legenda; tabela nunca usada para layout;
- Formulário com `label` associado por `for`/`id`, agrupamento por `fieldset`
  quando houver conjunto de opções.

**Perguntas de revisão manual:**

- Lendo só os cabeçalhos, em ordem, dá para entender do que a página trata?
- Um leitor de tela pulando de marco em marco chega ao conteúdo principal em um
  salto?
- Algum `div` com `onclick` está fazendo o trabalho de um `button`?

---

## 4. METADADOS, IDENTIDADE E COMPARTILHAMENTO

**Ferramentas:** extração do `head` servido; validador de dados estruturados
(Schema Markup Validator, Rich Results Test).

### 4.1 Título e descrição

- `title` único por rota, entre 30 e 60 caracteres, com o termo principal à
  esquerda, sem repetição de marca em toda página além do sufixo;
- `meta description` única, entre 120 e 160 caracteres, descrevendo o que a
  página entrega — não é palavra-chave empilhada, é a frase que a pessoa lê
  antes de decidir clicar.

### 4.2 Canonical

- Toda rota indexável declara `<link rel="canonical">` **absoluto**;
- Autorreferente por padrão; apontando para outra URL apenas quando a página é
  de fato variação de outra;
- Consistente com o `sitemap` e com o destino final dos redirecionamentos —
  canonical apontando para URL que redireciona é contradição.

### 4.3 Robots

- `<meta name="robots">` e cabeçalho `X-Robots-Tag` conferidos juntos: um
  contradiz o outro com frequência, e o cabeçalho vence;
- `noindex` presente **somente** onde é intencional (área autenticada,
  resultado de busca interna, ambiente de homologação);
- Ambiente de homologação nunca serve a mesma configuração da produção.

### 4.4 Open Graph e cartão social

- `og:title`, `og:description`, `og:url`, `og:type` e `og:image` presentes;
- Imagem com dimensão adequada (1200x630 recomendado), URL absoluta, acessível
  sem autenticação;
- `twitter:card` quando o projeto distribui link nessa rede.

### 4.5 Dados estruturados

- Formato JSON-LD, no `head` ou no corpo, nunca microdata espalhado;
- Tipo correspondente ao conteúdo real: `Article`, `Product`, `FAQPage`,
  `BreadcrumbList`, `Organization`, `WebSite`;
- **Todo campo declarado existe visivelmente na página.** Marcar preço ou nota
  que a pessoa não vê é marcação enganosa, e é BLOCKER;
- Validado por ferramenta oficial, com zero erro e os avisos justificados.

---

## 5. INDEXABILIDADE E DESCOBERTA

**Ferramentas:** `curl -I`, leitura do `robots.txt` e do `sitemap.xml`
servidos, rastreador local (`lychee`, `linkinator`, rastreador de SEO).

### 5.1 robots.txt

- Existe, é servido na raiz, responde 200;
- Não bloqueia recurso necessário à renderização (CSS, JS, imagem) — página
  bloqueada nesses recursos é avaliada como se estivesse quebrada;
- Aponta para o `sitemap`.

### 5.2 Sitemap

- Lista **apenas** URLs canônicas, indexáveis, que respondem 200;
- Sem URL redirecionada, sem `noindex`, sem duplicata;
- Atualizado quando a fase cria ou remove rota — sitemap velho é BLOCKER na
  fase que mexeu em roteamento.

### 5.3 URLs

- Legíveis, em minúsculas, separadas por hífen, sem parâmetro de sessão nem
  identificador opaco onde caberia texto;
- Estáveis: mudar URL exige redirecionamento 301 do endereço antigo;
- Padrão consistente entre rotas do mesmo tipo.

### 5.4 Links internos

- Toda rota nova é alcançável por link a partir de alguma página já indexada —
  página sem link de entrada é página que não existe para o rastreador;
- Texto do link descreve o destino;
- Navegação principal em HTML, não construída só por script;
- Zero link interno quebrado no diff. Verifique com rastreador, não a olho.

### 5.5 Conteúdo indexável

- O conteúdo que justifica a existência da página está no HTML servido;
- Texto essencial não vive dentro de imagem nem de `canvas`;
- Conteúdo atrás de aba, acordeão ou "carregar mais" está no documento, ainda
  que oculto por estilo — ou tem sua própria URL.

---

## 6. RENDERIZAÇÃO PARA MECANISMOS DE BUSCA

*Seção decisiva em aplicação de página única. Só é `N/A` quando a rota não
precisa ser indexável.*

**Ferramentas:** `curl` da URL sem executar script; comparação com o DOM após
execução; teste de URL em ferramenta de webmaster quando disponível.

Verifique:

- **O que existe antes do script rodar.** Baixe a rota com `curl` e leia. Se o
  conteúdo principal não estiver lá e a rota precisar ser indexável, é
  BLOCKER — independentemente de o navegador exibir corretamente;
- Metadados (`title`, `description`, `canonical`) presentes na resposta inicial,
  não injetados depois pelo cliente;
- Estado de erro refletido no status HTTP: rota inexistente responde 404, não
  200 com mensagem;
- Conteúdo idêntico para rastreador e para pessoa — servir versão diferente por
  agente de usuário é *cloaking*, e não é uma técnica que este gate aceita;
- Rolagem infinita acompanhada de paginação com URL própria, ou o conteúdo além
  da primeira página é invisível para a busca;
- Hidratação sem alterar o conteúdo textual servido — texto que muda entre
  servidor e cliente é sinal de estado inconsistente.

---

## 7. STATUS HTTP, REDIRECIONAMENTOS E DUPLICAÇÃO

**Ferramentas:** `curl -sIL` seguindo a cadeia; comparação de conteúdo entre
variações de URL.

### 7.1 Status

| Situação | Esperado |
|---|---|
| Página existente | 200 |
| Página removida em definitivo | 410, ou 404 |
| Endereço trocado em definitivo | 301 |
| Desvio temporário | 302 / 307 |
| Conteúdo inexistente | **nunca** 200 |

### 7.2 Redirecionamentos

- Um salto até o destino final. Cadeia com dois ou mais é BLOCKER;
- Sem laço;
- Preferência única e forçada por 301: com ou sem `www`, com ou sem barra
  final, `http` para `https`;
- Redirecionamento apontando sempre para página equivalente, nunca despejando
  tudo na inicial.

### 7.3 Conteúdo duplicado

- A mesma página não responde em mais de uma URL sem `canonical` resolvendo;
- Variação por parâmetro (ordenação, filtro, rastreamento) resolvida por
  canonical ou bloqueada por regra explícita;
- Paginação com canonical autorreferente — página 2 não é cópia da 1;
- Texto padrão repetido entre rotas não domina o conteúdo próprio de cada uma.

---

## 8. DESEMPENHO E CORE WEB VITALS

**Ferramentas:** Lighthouse (`lighthouse --preset=desktop` e perfil móvel),
`unlighthouse` para varredura de várias rotas, analisador de bundle do
empacotador, `curl -w` para tempo até o primeiro byte. Rode sempre contra build
de **produção**.

### 8.1 Limiares

Régua vinda de `[[DOC_REQUISITOS_NF]]`. Na ausência dela, o padrão:

```text
LCP   <= 2.5 s     (movel, mediana de 3 execucoes)
INP   <= 200 ms
CLS   <= 0.1
TTFB  <= 800 ms
Lighthouse Performance      >= 90
Lighthouse Accessibility    >= 95
Lighthouse SEO              >= 95
Lighthouse Best Practices   >= 90
```

### 8.2 Verificações

- Bundle de JavaScript da rota: comparado com a linha de base, com o delta
  declarado em kB;
- Divisão por rota funcionando — a rota não baixa o código da aplicação
  inteira;
- Imagem em formato moderno, dimensionada para o uso real, com `width` e
  `height` declarados (evita deslocamento), `loading="lazy"` fora da primeira
  dobra e **nunca** na imagem do LCP;
- Fonte com `font-display: swap` e pré-carregamento da face crítica;
- Terceiros inventariados: quantos, o que pesam, quais bloqueiam a
  renderização;
- Cabeçalho de cache e compressão (Brotli ou gzip) presentes nos ativos
  estáticos;
- CSS crítico entregue sem bloqueio desnecessário; folha não usada
  identificada.

**Perguntas de revisão manual:**

- Qual é o elemento do LCP nesta rota, e ele é o que deveria ser?
- O que exatamente causa o deslocamento de layout medido — anúncio, fonte,
  imagem sem dimensão, conteúdo injetado?
- Algum terceiro adicionado nesta fase justifica o custo que impôs?

---

## 9. RESPONSIVIDADE

**Ferramentas:** auditoria em perfil móvel; inspeção nos pontos de quebra do
projeto.

- `<meta name="viewport" content="width=device-width, initial-scale=1">`
  presente, sem `maximum-scale` nem `user-scalable=no` — impedir zoom é falha
  de acessibilidade;
- Zero estouro horizontal em 320, 375, 768, 1024 e 1440 px;
- Alvo de toque com pelo menos 44x44 px e espaçamento entre alvos vizinhos;
- Texto legível sem zoom; sem tamanho fixo em pixel que impeça a preferência do
  sistema;
- Imagem responsiva com `srcset`/`sizes` quando o peso justificar;
- Conteúdo e função equivalentes entre telas — versão móvel que esconde
  conteúdo indexável é problema de SEO, não só de layout.

---

## 10. ACESSIBILIDADE

**Ferramentas:** `axe-core` (`@axe-core/cli`, `axe-playwright`), `pa11y`,
seção de acessibilidade do Lighthouse. Nenhuma delas cobre mais que uma parte —
a revisão manual abaixo é obrigatória.

### 10.1 Automatizável

```text
Violacoes axe de nivel A e AA introduzidas pela fase = 0
```

Cobre contraste, nome acessível, `alt` ausente, rótulo de formulário, atributo
ARIA inválido, ordem de cabeçalho.

### 10.2 Revisão manual

- Percorra a rota inteira **só com teclado**: Tab, Shift+Tab, Enter, Espaço,
  Esc. Todo controle é alcançável, o foco é visível, nada prende o foco;
- Modal devolve o foco ao elemento que a abriu e fecha no Esc;
- Mensagem de erro de formulário associada ao campo e anunciada;
- Conteúdo que aparece de forma dinâmica é anunciado (região viva) quando
  importa;
- Cor não é o único portador de informação;
- Animação respeita `prefers-reduced-motion`;
- `alt` descreve a função da imagem, não o arquivo. Imagem decorativa leva
  `alt=""`, nunca uma descrição que o leitor de tela terá de atravessar.

Acessibilidade e SEO medem a mesma coisa por dois caminhos: se o rastreador não
entende a página, o leitor de tela também não.

---

## 11. METRIFICAÇÃO

A metade que transforma este gate em régua. Conformidade diz o que está errado
hoje; metrificação diz se o projeto está melhorando ou piorando.

### 11.1 Linha de base

Mantida em `[[DOC_METRICAS_FRONT]]`. Na primeira execução do gate, este arquivo
é criado com a medição atual e o relatório declara `LINHA DE BASE CRIADA` — sem
inventar comparação com um passado que não foi medido.

Cada registro guarda, por rota medida:

```text
data | rota | perfil (movel|desktop) | build | LCP | INP | CLS | TTFB |
Perf | A11y | SEO | BP | JS (kB) | erros axe | itens de SEO em falha
```

Rotas medidas: a inicial, a rota de entrada mais relevante para busca, e toda
rota tocada pela fase. Medir tudo em projeto grande é caro e desnecessário —
medir sempre as mesmas é o que faz a série valer.

### 11.2 Condição de medição

Registre junto com o número, ou a série não significa nada:

- URL exata e ambiente (produção, pré-produção, local);
- Perfil de rede e CPU, e modo móvel ou desktop;
- Ferramenta e versão;
- Mediana de 3 execuções, para métrica de laboratório;
- Identificador da build ou do commit.

### 11.3 Comparação

Para cada métrica: valor anterior, valor atual, variação absoluta e percentual,
direção (melhorou, piorou, estável — variação abaixo de 3% é estável).

```text
METRICA   ANTES    DEPOIS   VARIACAO   AVALIACAO
LCP       2.9 s    2.1 s    -0.8 s     melhorou, dentro do limiar
CLS       0.04     0.11     +0.07      piorou, acima do limiar -> BLOCKER
JS rota   412 kB   388 kB   -24 kB     melhorou
axe AA    3        0        -3         resolvido
```

### 11.4 Campo e laboratório

Métrica de laboratório (Lighthouse) é reproduzível e serve para comparar
builds. Métrica de campo (dados de uso real, se o projeto coletar) é a que
descreve a experiência de fato. Quando as duas existirem, reporte as duas — e
quando divergirem, a de campo é a que vale.

Se o projeto não coleta métrica de campo e tem tráfego real, registre isso como
pendência no relatório. Não é BLOCKER; é lacuna conhecida.

---

## 12. AVALIAÇÃO OBJETIVA POR ACHADO

Todo achado deste gate é reportado no formato abaixo, sem exceção. Achado sem
métrica afetada e sem estado esperado é opinião, e opinião não reprova nem
aprova fase nenhuma.

```text
PROBLEMA ENCONTRADO
IMPACTO
SEVERIDADE            BLOCKER | WARNING
METRICA AFETADA
ESTADO ATUAL
ESTADO ESPERADO
CORRECAO APLICADA
RESULTADO APOS A CORRECAO
```

Preenchimento:

- **Problema encontrado** — o que está errado, com arquivo, rota e linha.
  "Imagem grande" não é achado; "`hero.png`, 1,8 MB, elemento do LCP em
  `/produtos`" é.
- **Impacto** — o que isso causa para quem usa ou para quem rastreia. Ligado ao
  efeito real, não à regra violada.
- **Severidade** — pela seção 2.
- **Métrica afetada** — o número que se move quando isto for corrigido. Se
  nenhum se move, reavalie se o achado existe.
- **Estado atual** — o valor medido, com a evidência.
- **Estado esperado** — o limiar de `[[DOC_REQUISITOS_NF]]`, ou o padrão da
  seção 8.1. Nunca "melhor".
- **Correção aplicada** — o que foi feito. `NAO CORRIGIDO` é resposta válida
  para WARNING aceito, acompanhada da justificativa.
- **Resultado após a correção** — o valor **remedido**, não o valor esperado.
  Correção sem remedição fica `NOT_MEASURED`, e BLOCKER com correção não
  remedida continua BLOCKER.

---

## 13. PROIBIÇÕES

Durante a correção dos achados deste gate é proibido:

- Marcar item como resolvido sem remedir;
- Comparar medida de desenvolvimento com medida de produção, ou móvel com
  desktop, e chamar de evolução;
- Escolher a melhor de várias execuções em vez da mediana;
- Desligar, ignorar ou reduzir o escopo de uma verificação para o gate passar —
  inclui remover rota da lista de medição e afrouxar limiar sem registrar em
  `[[DOC_REQUISITOS_NF]]`;
- Suprimir violação de acessibilidade com regra de exceção sem justificativa
  registrada;
- Declarar conformidade de dados estruturados sem passar pelo validador;
- Marcar conteúdo em dados estruturados que não aparece na página;
- Servir conteúdo diferente para rastreador e para pessoa;
- Ocultar texto para mecanismo de busca (fora da tela, cor igual ao fundo,
  tamanho zero);
- "Resolver" achado com comentário ou TODO prometendo corrigir depois.

Violação destas proibições é BLOCKER e deve ser revertida.

---

## 14. RE-EXECUÇÃO E LIMITE DE ITERAÇÕES

Depois de qualquer correção, execute novamente as medições afetadas. Nunca
considere uma correção de desempenho ou de acessibilidade válida por inspeção
visual — o número é a prova.

```text
MAX_GATE_ITERATIONS = 3
```

Uma iteração = ciclo completo de (medir -> corrigir -> remedir).

Se após 3 iterações ainda existirem BLOCKERs:

1. PARE de corrigir;
2. emita STATUS = FAIL;
3. liste os BLOCKERs remanescentes no formato da seção 12, com o número atual;
4. explique o que foi tentado em cada iteração e por que não convergiu;
5. recomende o próximo passo (mudança de estratégia de renderização, remoção de
   terceiro, decisão do dono sobre o limiar).

---

## 15. RESULTADO FINAL

Ao finalizar, produza obrigatoriamente:

```text
FRONTEND QUALITY GATE
STATUS: PASS | FAIL
ITERACOES EXECUTADAS: N / 3
LINHA DE BASE: existente | criada nesta execucao
```

### ESCOPO DA REVISÃO
Rotas medidas, seções aplicáveis (pela tabela da seção 1) e seções `N/A`.

### CONDIÇÃO DE MEDIÇÃO
Ambiente, build/commit, ferramenta e versão, perfil de rede e CPU, número de
execuções.

### EVIDÊNCIAS
Cada comando executado, com a saída relevante. Métrica sem evidência =
`NOT_MEASURED`.

### CONFORMIDADE

- HTML semântico e estrutura
- Título, descrição e canonical
- Robots e sitemap
- Open Graph e dados estruturados
- URLs e links internos
- Conteúdo indexável e renderização
- Status HTTP, redirecionamentos e duplicação
- Imagens e texto alternativo
- Desempenho e Core Web Vitals
- Responsividade
- Acessibilidade

*(para cada item: achados no formato da seção 12, ou "N/A — fora do escopo")*

### QUADRO DE MÉTRICAS

```text
ROTA   METRICA   ANTES   DEPOIS   VARIACAO   LIMIAR   AVALIACAO
```

### BLOCKERS
Cada um com o formato da seção 12 e status (resolvido | remanescente).

### WARNINGS ACEITOS
Cada um com justificativa re-argumentada do zero e a métrica que se aceitou não
mover.

### DÍVIDA CONHECIDA
O que não foi medido e por quê; ausência de métrica de campo; rota fora da
lista de medição.

### VEREDITO

A fase somente pode ser marcada como concluída quando:

```text
STATUS = PASS
```

`STATUS = PASS` exige: zero BLOCKERs remanescentes, todo WARNING aceito com
justificativa registrada, e `[[DOC_METRICAS_FRONT]]` atualizado com a medição
desta fase.

Nunca invente número. Quando uma verificação não puder ser medida com as
ferramentas disponíveis, tente instalar uma leve equivalente antes de declarar
`NOT_MEASURED` — e, nesse caso, sinalize que a área ficou sem cobertura
automatizada, para que a revisão manual receba atenção redobrada.
