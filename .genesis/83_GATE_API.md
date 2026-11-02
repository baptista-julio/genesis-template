# Quality Gate — Design de API

> Prompt do pacote Genesis. Referências `[[ID]]` resolvem pelo `MANIFEST.md`.
> Primeiro gate da sequência: executado ao fim da implementação, antes de
> `[[GATE_FRONTEND]]`, `[[GATE_INPUT]]`, `[[GATE_SECURITY]]` e `[[GATE_CODE]]`.
> Aplicável quando `tem_api: sim` no perfil. Caso contrário, `N/A`.
> O contrato de referência é `[[DIR_API]]`; os limiares de desempenho e
> disponibilidade vêm de `[[DOC_REQUISITOS_NF]]`; as fronteiras entre módulos,
> de `[[DOC_ARQUITETURA]]`. Quando o projeto definir convenção própria em ADR,
> **a do projeto prevalece** — registre a divergência no relatório final.

**Referência normativa:** [Práticas recomendadas de design de API Web —
Microsoft Azure Architecture Center][ms-api-design]. Quando este gate e a
referência divergirem, prevalece a referência, salvo ADR do projeto em
contrário. Os itens de implementação operacional — ETag, cache, limitação de
taxa — vêm do artigo irmão, [Práticas recomendadas de implementação de API
Web][ms-api-impl]; estão aqui porque são decisões de design de contrato, ainda
que a primeira página não os cubra.

[ms-api-design]: https://learn.microsoft.com/pt-br/azure/architecture/best-practices/api-design
[ms-api-impl]: https://learn.microsoft.com/pt-br/azure/architecture/best-practices/api-implementation


Você acabou de concluir uma fase de implementação **e este é o primeiro gate
da sequência**. Este gate avalia exclusivamente o **design da
interface pública**: o que um consumidor externo vê, chama e depende.

O objetivo não é verificar se a API funciona — isso o gate de código já mediu.
O objetivo é verificar se ela é **consistente, previsível, segura e
sustentável**:

- **Consistente** — dois endpoints que fazem coisas parecidas se parecem.
- **Previsível** — quem conhece metade da API acerta a outra metade no chute.
- **Segura** — nenhuma operação depende do cliente se comportar bem.
- **Sustentável** — dá para evoluir sem quebrar quem já integrou.

Atue como um **Senior API Designer independente**, na posição de quem vai
integrar com esta API sem poder perguntar nada a quem a escreveu.

---

## REGRAS DE INDEPENDÊNCIA DA REVISÃO

1. **Avalie o contrato servido, não a intenção do código.** Obtenha a
   especificação real (`/openapi.json`, `/swagger.json`, arquivo de contrato do
   repositório) ou, na ausência dela, levante as rotas a partir do roteador.
   Comentário no código não é contrato.

2. **Leia a API inteira, não só o diff.** Este é o único gate do pacote em que
   o que já existia importa tanto quanto o que entrou: um endpoint novo bem
   desenhado que contradiz os vinte anteriores **piora** a API. Consistência só
   se mede contra o conjunto.

3. **Toda constatação vem de evidência real** — chamada executada, resposta
   observada, linha do contrato citada. Nunca declare um status code correto
   por inferência: chame e veja o que volta.

4. **Pense como integrador, não como autor.** Para cada endpoint alterado:
   *"Se eu tivesse só a documentação, o que eu erraria na primeira tentativa?"*
   O que precisa de explicação é o que está mal desenhado.

5. **Justificativas dadas durante a implementação não valem no gate.** "O
   cliente sempre manda esse campo" precisa ser re-argumentada contra o
   contrato, que é o que o cliente de fato promete.

6. **Compatibilidade se verifica contra o contrato publicado anterior**, não
   contra a memória da sessão. Obtenha a versão anterior da especificação
   (versionador, artefato publicado) e compare.

---

## 1. ESCOPO E APLICABILIDADE

Antes de aplicar os checklists, determine, a partir do diff real:

- quais endpoints a fase criou, alterou ou removeu;
- se algum **contrato existente** mudou: campo, tipo, obrigatoriedade, formato,
  código de status, semântica;
- quem consome a API: cliente próprio, parceiro, público. Isso decide o rigor
  de compatibilidade — API pública não quebra, API interna negocia;
- se o projeto tem especificação OpenAPI, e se ela é gerada do código ou
  escrita à mão.

| O diff toca em... | Seções obrigatórias |
|---|---|
| Rota nova, recurso novo, mudança de URI | 3, 4, 12 |
| Método HTTP, semântica de operação, idempotência | 4, 5 |
| Corpo de resposta, envelope, formato de erro | 6, 7 |
| Coleção, listagem, busca | 8 |
| Campo, tipo, obrigatoriedade em contrato existente | 9, 11 |
| Autenticação, autorização, escopo, chave | 10 |
| Cabeçalho de cache, ETag, concorrência | 5.3, 13 |
| Limitação de taxa, cota, throttling | 10.3 |
| Especificação OpenAPI, documentação da API | 11 |
| Operação longa, processamento assíncrono, lote | 5.4 |

Seções fora do escopo entram no relatório como `N/A — fora do escopo desta
alteração`, nunca omitidas.

**A seção 12 (consistência) nunca é `N/A`** quando a fase tocou qualquer
endpoint. É a única verificação que exige olhar a API inteira, e é a que se
degrada em silêncio.

---

## 2. CLASSIFICAÇÃO DE SEVERIDADE

### BLOCKER — reprova o gate (STATUS = FAIL)

- **Quebra de compatibilidade sem versionamento**: campo removido ou renomeado,
  tipo alterado, campo opcional que virou obrigatório, valor de enumeração
  removido, código de status alterado, semântica de operação alterada — em
  contrato já publicado e consumido;
- Método HTTP com semântica errada: `GET` que altera estado, `DELETE` que cria,
  `POST` usado para tudo (nível 0 do modelo de maturidade);
- Operação declarada idempotente que não é, ou `PUT`/`DELETE` não idempotentes;
- Endpoint sem autenticação que expõe dado não público;
- Autorização ausente em endpoint que manipula recurso identificável por ID
  (o mesmo IDOR do `[[GATE_SECURITY]]` A01, aqui visto pelo contrato);
- Erro devolvido com status 200, ou erro de cliente devolvido como 5xx;
- Resposta de erro vazando detalhe interno: stack trace, SQL, caminho de
  arquivo, nome de tabela;
- Listagem de coleção sem paginação e sem limite superior — um `GET /pedidos`
  que devolve tudo é negação de serviço esperando acontecer;
- Limite de paginação sem teto: cliente pede `limit=1000000` e o servidor
  tenta;
- Contrato publicado divergente do comportamento real (a especificação promete
  o que a API não faz, ou omite o que ela faz);
- Endpoint novo ausente da especificação OpenAPI, quando o projeto mantém uma;
- Verbo no URI de recurso (`/criar-pedido`) em endpoint novo;
- Dado sensível em parâmetro de URL (token, senha, documento) — vai para log de
  servidor, histórico de proxy e cabeçalho `Referer`.

### WARNING — reportado e justificado, não reprova sozinho

- Inconsistência de nomenclatura com o resto da API (singular onde o resto usa
  plural, `camelCase` onde o resto usa `snake_case`);
- Aninhamento de URI além de *coleção/item/coleção*;
- Formato de erro divergente entre endpoints;
- Ausência de `Location` em resposta 201;
- Filtro, ordenação ou seleção de campos ausentes onde a coleção claramente
  pede;
- Ausência de ETag/cabeçalho de cache em recurso de leitura frequente e pouca
  mudança;
- Ausência de controle de concorrência otimista em recurso com escrita
  concorrente plausível;
- Ausência de limitação de taxa em endpoint custoso;
- Ausência de identificador de correlação nas respostas;
- Operação longa síncrona que caberia em 202 + endpoint de status;
- Documentação sem exemplo de requisição e resposta;
- Campo de contrato sem descrição na especificação;
- API tagarela: recurso pequeno demais, obrigando o cliente a várias chamadas
  para montar uma tela.

**Escalonamento:** um WARNING vira BLOCKER quando (a) se repete de forma
sistemática, indicando padrão estrutural; (b) já foi apontado em iteração
anterior deste gate e ignorado sem justificativa; (c) atinge endpoint de API
pública ou consumida por parceiro — ali, inconsistência custa suporte, e
suporte não escala.

---

## 3. RECURSOS, ENDPOINTS E NOMENCLATURA

*Referência: seções "Definir URIs de recurso" e "Convenções de nomenclatura".*

### 3.1 Modelagem

- A API é organizada em torno de **recursos** (substantivos), não de operações
  (verbos). `POST /pedidos`, não `POST /criar-pedido`;
- Recursos mapeiam **entidades de negócio**, não tabelas do banco. API que
  espelha o esquema interno acopla o cliente ao banco, aumenta a superfície de
  ataque e transforma migração em quebra de contrato;
- Coleção e item são recursos distintos, com URIs próprios: `/clientes` e
  `/clientes/5`;
- Relação expressa por hierarquia quando é natural (`/clientes/5/pedidos`), sem
  passar de *coleção/item/coleção*. Além disso, use link no corpo da resposta;
- Operação que genuinamente não é recurso (cálculo, conversão, busca complexa)
  pode virar pseudo-recurso — com moderação, e documentada como exceção.

### 3.2 Convenções

Verifique que a fase seguiu a convenção **já existente** no projeto. Na
ausência de convenção registrada, o padrão da referência:

| Item | Convenção |
|---|---|
| Nome de coleção | substantivo, plural, minúsculo: `/pedidos` |
| Separador em URI | hífen, nunca sublinhado nem camelCase |
| Identificador | `/pedidos/{id}` |
| Campo no corpo | um único estilo em toda a API (`camelCase` ou `snake_case`) |
| Parâmetro de consulta | mesmo estilo dos campos do corpo |
| Cabeçalho personalizado | `X-` evitado em cabeçalho novo; nome descritivo |
| Data e hora | ISO 8601 com fuso, em UTC (`2026-03-22T12:11:38Z`) |
| Dinheiro | inteiro em unidade mínima, ou decimal com moeda explícita — nunca ponto flutuante binário |
| Enumeração | valor estável e legível, nunca índice numérico posicional |

**Perguntas de revisão manual:**

- Um integrador que conhece `/clientes` acerta o URI de `/pedidos` sem
  consultar a documentação?
- Algum nome de campo vaza jargão interno (`flg_st`, `tp_reg`) em vez do termo
  do domínio?
- A API é tagarela? Montar uma tela exige quantas chamadas?

---

## 4. MÉTODOS HTTP E STATUS CODES

*Referência: seção "Definir métodos de API Web RESTful".*

### 4.1 Semântica dos métodos

| Método | Semântica | Idempotente | Seguro |
|---|---|---|---|
| `GET` | lê, nunca altera estado | sim | sim |
| `POST` | cria na coleção, ou processa | não | não |
| `PUT` | substitui o item por inteiro | **sim** | não |
| `PATCH` | altera parte do item | não garantido | não |
| `DELETE` | remove | **sim** | não |
| `HEAD` | cabeçalhos do `GET`, sem corpo | sim | sim |

- `GET` com efeito colateral é BLOCKER: proxies, rastreadores e pré-carregadores
  vão disparar a operação sem pedir licença;
- `POST` para item específico (`POST /clientes/1`) devolve 400 ou 405 — o
  cliente não escolhe o URI do recurso que cria;
- `PUT` recebe a representação **completa**; `PATCH` recebe o documento de
  alteração, com tipo de mídia declarado (`application/merge-patch+json` ou
  `application/json-patch+json`);
- Efeito sobre coleção e sobre item é diferente e ambos estão documentados —
  `DELETE /clientes` apagando tudo precisa ser intenção, não descuido.

### 4.2 Status codes

Confira os códigos observados contra a tabela da referência:

| Operação | Códigos esperados |
|---|---|
| `GET` | 200, 204 (sem conteúdo), 404 |
| `POST` | 201 (criou, com `Location`), 200 (processou), 204, 400, 405 |
| `PUT` | 200, 201 (criou), 204, 409 (conflito) |
| `PATCH` | 200, 400 (documento malformado), 409, 415 (formato não suportado) |
| `DELETE` | 204, 404 |
| Tipo de mídia não suportado no corpo | 415 |
| Nenhum tipo do `Accept` atendível | 406 |
| Processamento assíncrono aceito | 202, com `Location` do endpoint de status |
| Conteúdo parcial | 206; intervalo inválido, 416 |

Regras que este gate cobra:

- **Faixa correta**: erro do cliente é 4xx, falha do servidor é 5xx. Erro de
  validação devolvido como 500 esconde defeito e engana monitoramento;
- **Nunca 200 com erro no corpo.** `{"sucesso": false}` com status 200 quebra
  todo cliente que confia no protocolo;
- 401 quando falta ou falha a credencial; 403 quando a credencial é válida mas
  não autoriza. Trocar os dois faz o cliente tentar reautenticar em laço;
- 404 e 403 distinguidos com cuidado: revelar que o recurso existe já é
  vazamento em alguns domínios. Escolha deliberada, registrada;
- 409 para conflito de estado, 422 para entidade sintaticamente válida e
  semanticamente inválida — se o projeto usa 422, usa em todos;
- 429 para limite de taxa, com `Retry-After`;
- 201 sempre com `Location` apontando para o recurso criado.

### 4.3 Idempotência

- `PUT` e `DELETE` são idempotentes **de fato**: repetir a chamada leva ao mesmo
  estado final. Verifique chamando duas vezes;
- `DELETE` repetido devolve 404 ou 204 de forma consistente — escolha uma e
  documente;
- `POST` que cria recurso com efeito financeiro, de cobrança ou de
  irreversibilidade aceita **chave de idempotência** (cabeçalho
  `Idempotency-Key`), e chamada repetida com a mesma chave devolve o resultado
  original em vez de criar de novo. Ausência disso em operação de pagamento é
  BLOCKER;
- Reenvio por *timeout* de rede é o caso normal, não a exceção: o cliente não
  sabe se a requisição chegou.

### 4.4 Operações longas e em lote

- Operação que não conclui em tempo razoável devolve **202 Accepted** com
  `Location` de um endpoint de status;
- O endpoint de status devolve o estado atual e, ao concluir criando recurso,
  **303 See Other** com `Location` do recurso final;
- Operação em lote tem semântica de falha parcial declarada: tudo ou nada, ou
  relatório por item. Ambiguidade aqui gera corrupção silenciosa de dados.

---

## 5. NEGOCIAÇÃO DE CONTEÚDO, CACHE E CONCORRÊNCIA

### 5.1 Content-Type e Accept

*Referência: seção "Tipos de recursos MIME".*

- Toda requisição com corpo declara `Content-Type`; a resposta declara o seu;
- Conjunto de caracteres explícito (`application/json; charset=utf-8`);
- `Content-Type` não suportado devolve **415**; `Accept` não atendível devolve
  **406**;
- A API não adivinha formato pelo conteúdo do corpo.

### 5.2 Cache

- Recurso de leitura frequente e mudança rara declara `Cache-Control` com
  duração deliberada;
- Recurso que depende de quem chama é `private`; recurso por locatário ou por
  usuário **nunca** é `public` — cache compartilhado servindo resposta de um
  cliente para outro é vazamento;
- Resposta que varia por cabeçalho (versão, idioma, locatário) declara `Vary`;
- Dado sensível declara `no-store`.

### 5.3 Concorrência otimista

*Referência: artigo de implementação, seção de concorrência.*

- Recurso com escrita concorrente plausível devolve `ETag` no `GET`;
- Escrita condicional aceita `If-Match`, e devolve **412** quando o ETag não
  confere;
- `If-None-Match` no `GET` devolve **304** quando nada mudou;
- Sem isso, o último a escrever vence em silêncio, e o cliente não tem como
  saber que sobrescreveu alguém.

### 5.4 Respostas parciais

*Aplicável a recurso binário grande.*

- `Accept-Ranges: bytes` declarado; `Range` atendido com **206** e
  `Content-Range`; intervalo inválido devolve **416**;
- `HEAD` disponível para o cliente descobrir tamanho antes de baixar.

---

## 6. ESTRUTURA DAS RESPOSTAS

- **Forma única em toda a API.** Ou toda resposta vem envelopada, ou nenhuma
  vem. Metade e metade é o defeito mais caro desta seção, porque obriga o
  cliente a saber caso a caso;
- Coleção devolve objeto com a lista e os metadados de paginação, não um array
  cru na raiz — array na raiz não tem onde crescer sem quebrar;
- Ausência de valor representada de forma única: `null` ou campo ausente,
  escolhido e aplicado sempre igual;
- Tipos estáveis: número que às vezes chega como string obriga o cliente a
  defender-se de quem deveria ser confiável;
- Identificador sempre no mesmo formato e no mesmo campo;
- A resposta não devolve mais do que o consumidor precisa: campo interno,
  contagem cara ou objeto aninhado inteiro por padrão são custo permanente;
- Links de relação (HATEOAS) quando o projeto adotou o padrão — adotado, é para
  todos os recursos. Nível 3 do modelo de maturidade de Richardson é uma
  decisão de projeto, registrada em ADR, não um item avulso de endpoint.

---

## 7. TRATAMENTO DE ERROS

- **Formato único de erro em toda a API**, com: código de erro estável e
  legível por máquina, mensagem legível por pessoa, e identificador de
  correlação. Campo por campo, quando for erro de validação;
- Adote um padrão e cite-o na documentação. `application/problem+json`
  (RFC 9457) resolve isto e já tem cliente pronto em toda linguagem — divergir
  dele é decisão, não descuido;
- **Código de erro é contrato.** Cliente que ramifica por mensagem de texto
  quebra na primeira tradução; ramificar por código é o comportamento correto,
  e só funciona se o código for estável;
- Erro de validação relata **todos** os campos inválidos de uma vez, não o
  primeiro. Formulário que erra quatro campos não deve exigir quatro
  requisições;
- Mensagem diz o que fazer, não só o que houve. "Campo `dataEmissao` é
  obrigatório" serve; "requisição inválida" não;
- Zero vazamento interno: sem stack trace, SQL, caminho de arquivo, nome de
  tabela, nome de biblioteca. O detalhe vai para o log, associado ao mesmo
  identificador de correlação devolvido ao cliente;
- 5xx transitório acompanha `Retry-After` quando o cliente pode tentar de novo.

---

## 8. COLEÇÕES: PAGINAÇÃO, FILTROS E ORDENAÇÃO

*Referência: seção "Implementar paginação e filtragem de dados".*

### 8.1 Paginação

- **Toda coleção que pode crescer é paginada.** Sem exceção, e desde o primeiro
  dia: coleção que hoje tem 12 itens é a que amanhã derruba o servidor;
- Padrões declarados e sensatos (a referência sugere `limit=25`, `offset=0`);
- **Teto obrigatório.** `limit` acima do máximo devolve 400 ou é reduzido ao
  teto — decida uma e documente. Sem teto, a paginação não protege nada;
- Metadados de paginação na resposta: total quando for barato calcular, e como
  pedir a próxima página;
- Estratégia coerente com o dado: `limit`/`offset` é simples e serve à maioria;
  paginação por cursor é o que mantém a ordem correta em coleção que muda
  durante a navegação. Escolha registrada, não acidental;
- Mesmo estilo de paginação em todas as coleções da API.

### 8.2 Filtros

- Passados na cadeia de consulta, com nome igual ao do campo que filtram;
- Campo filtrável validado contra lista conhecida — filtro repassado direto ao
  banco é injeção (`[[GATE_SECURITY]]` A05) e vazamento de esquema;
- Filtro por campo que o consumidor não pode ler não existe: ele vira oráculo
  para descobrir dado por tentativa;
- Combinação de filtros com semântica declarada (`E` entre parâmetros
  distintos, por padrão);
- Filtro caro tem índice correspondente, ou vira o gargalo que ninguém previu.

### 8.3 Ordenação

- Parâmetro único e consistente (`sort=preco`, `sort=-preco` para descendente,
  ou a convenção que o projeto já usa);
- Campos ordenáveis restritos a uma lista conhecida;
- **Ordem total e estável**: ordenação por campo não único precisa de critério
  de desempate, senão a paginação repete e pula itens entre páginas;
- Ordem padrão definida e documentada — coleção sem ordem declarada é ordem do
  banco, que muda sem aviso;
- A referência lembra: parâmetro de ordenação fragmenta cache, porque entra na
  chave. Considere ao definir a política de cache.

### 8.4 Seleção de campos

- `fields=id,nome` quando existir, com validação de que o consumidor pode ler
  cada campo pedido;
- Campo inválido devolve erro, não silêncio.

---

## 9. CONTRATOS E COMPATIBILIDADE RETROATIVA

*Referência: seção "Implementar controle de versão".*

### 9.1 O que quebra

Mudança **incompatível** em contrato publicado e consumido:

- remover ou renomear campo de resposta;
- alterar tipo, formato ou unidade de um campo;
- tornar obrigatório um parâmetro antes opcional;
- restringir o que era aceito (faixa, tamanho, enumeração);
- remover valor de enumeração de resposta;
- alterar código de status ou semântica de operação;
- alterar o significado de um campo mantendo o nome — o pior de todos, porque
  não quebra: passa a mentir.

Mudança **compatível**:

- adicionar campo opcional à requisição;
- adicionar campo à resposta (cliente tolerante ignora o que não conhece);
- adicionar endpoint, ou valor de enumeração em campo de entrada;
- relaxar validação.

### 9.2 Versionamento

Estratégias, com o trade-off que a referência aponta:

| Estratégia | Exemplo | A favor | Contra |
|---|---|---|---|
| Nenhuma | — | simples | só serve a API interna com cliente tolerante |
| URI | `/v2/clientes/3` | explícito, roteável, **amigável a cache** | mesmo recurso em dois URIs; complica HATEOAS |
| Consulta | `?version=2` | URI estável | navegador/proxy antigo pode não cachear; complica HATEOAS |
| Cabeçalho | `Custom-Header: api-version=2` | URI limpo | invisível no navegador; fragmenta cache |
| Tipo de mídia | `Accept: application/vnd.contoso.v1+json` | correto no protocolo, bom com HATEOAS | mais difícil de testar à mão |

O gate não impõe estratégia — impõe que exista **uma**, registrada em ADR
(`[[DIR_ADR]]`), e que a fase a tenha seguido. Duas estratégias convivendo é
BLOCKER de consistência.

Verifique ainda:

- versão ausente na requisição tem comportamento definido (versão padrão
  declarada, ou erro explícito);
- versão anterior continua respondendo enquanto houver consumidor;
- depreciação anunciada antes de remover: cabeçalhos `Deprecation` e `Sunset`,
  registro na documentação e no `[[DOC_CHANGELOG]]`;
- **cliente tolerante documentado**: a API declara que consumidor deve ignorar
  campo desconhecido. Sem essa cláusula, adicionar campo já é quebra.

### 9.3 Verificação

- Compare a especificação atual com a anterior. Havendo ferramenta de
  comparação de contrato (`oasdiff` e equivalentes), execute e cite a saída;
- Sem ferramenta, a comparação é manual, campo a campo, e o relatório diz isso;
- **Toda quebra encontrada é BLOCKER até que se prove que ninguém consome** — e
  a prova é evidência de uso, não suposição.

---

## 10. SEGURANÇA DO CONTRATO

Complementa o `[[GATE_SECURITY]]`, que olha a implementação. Aqui o objeto é o
**contrato**: o que ele obriga, permite e revela.

### 10.1 Autenticação

- Todo endpoint declara explicitamente se exige autenticação. Endpoint sem
  declaração é achado — o padrão implícito é onde mora o vazamento;
- Esquema padrão (`Bearer` com token, mTLS, chave de API para servidor), sempre
  em cabeçalho, **nunca em URL**;
- Credencial em parâmetro de consulta é BLOCKER: vai para log de servidor,
  histórico de proxy e `Referer`;
- Falha de autenticação devolve 401 com `WWW-Authenticate`, sem revelar se a
  identidade existe;
- Prazo de validade e renovação do token documentados.

### 10.2 Autorização

- Regra de autorização declarada por endpoint: quem pode, sobre qual recurso;
- Escopo/permissão nomeado por operação, não por papel genérico — papel muda,
  operação não;
- Posse do recurso verificada **no servidor**, sempre. Identificador na URL não
  é prova de nada;
- Autorização no nível do campo quando houver: campo que só o dono vê não pode
  aparecer para terceiros, nem por seleção de campos, nem por filtro, nem por
  ordenação;
- Recurso de outro locatário é inalcançável **por construção**, não por
  filtro esquecível na consulta.

### 10.3 Limitação de taxa

- Endpoint custoso ou sensível tem limite declarado;
- Limite excedido devolve **429** com `Retry-After`;
- Limite corrente informado em cabeçalhos (`RateLimit-Limit`,
  `RateLimit-Remaining`, `RateLimit-Reset`) — cliente que não enxerga o limite
  só descobre batendo nele;
- Política documentada: por chave, por IP, por locatário, por janela;
- Endpoint de autenticação tem limite mais estrito que o resto.

### 10.4 Superfície

- Tamanho máximo de corpo e de coleção declarados e impostos;
- Nenhum campo interno exposto: identificador sequencial de banco, hash de
  senha, marca de exclusão lógica, controle de versão de linha;
- Enumeração de recursos por identificador sequencial considerada — se
  `/pedidos/1` até `/pedidos/n` mapeia a base inteira, o identificador deveria
  ser opaco;
- Mensagem de erro que não distingue "não existe" de "não é seu", quando o
  domínio exigir.

---

## 11. DOCUMENTAÇÃO E OPENAPI

*Referência: seção "Iniciativa OpenAPI".*

- Especificação OpenAPI existe, é válida (`spectral`, `swagger-cli validate`,
  `redocly lint`) e **corresponde ao comportamento real**;
- Especificação divergente do comportamento é BLOCKER, e é pior que ausência:
  documentação errada é confiada;
- Endpoint novo entra na especificação **na mesma fase**, não depois;
- Cada operação tem: resumo, descrição, parâmetros com tipo e obrigatoriedade,
  esquema de requisição e resposta, **todos** os códigos de status possíveis
  (inclusive os de erro), e esquema de segurança;
- Cada esquema tem descrição por campo, formato, e exemplo;
- Exemplo de requisição e de resposta que **funcionam** se copiados;
- Erros documentados com o mesmo peso dos sucessos — a lista de códigos de erro
  estáveis faz parte do contrato;
- Depreciação marcada na especificação (`deprecated: true`) com o que usar no
  lugar;
- Especificação gerada do código, ou verificada contra ele por teste de
  contrato. Escrita à mão e não verificada, ela envelhece em uma semana;
- `[[DIR_API]]` atualizado, com o que a especificação não cabe: fluxos de
  autenticação, ordem de chamadas, limites, política de versionamento, guia de
  migração entre versões.

---

## 12. CONSISTÊNCIA ENTRE ENDPOINTS

A verificação que só existe olhando a API inteira, e a que este gate cobra com
mais rigor. Uma API previsível vale mais que uma API perfeita endpoint a
endpoint: quem integra aprende o padrão uma vez e aplica ao resto.

Monte a tabela do que a fase alterou contra o que já existia:

```text
DIMENSAO                    PADRAO DA API      NESTA FASE     CONFERE
Nome de coleção             plural minusculo   plural         sim
Estilo de campo             camelCase          camelCase      sim
Formato de data             ISO 8601 UTC       epoch ms       NAO
Envelope de resposta        objeto com "data"  array cru      NAO
Formato de erro             problem+json       proprio        NAO
Paginação                   limit/offset       page/size      NAO
Ordenação                   sort=-campo        orderBy        NAO
Autenticação                Bearer             Bearer         sim
Versionamento               URI /v1            URI /v1        sim
Identificador               UUID string        int            NAO
```

Cada `NAO` é um achado, no formato da seção 14. Divergência introduzida pela
fase é WARNING; divergência sistemática, ou em API pública, é BLOCKER.

Quando a fase estiver **certa** e o resto da API errado, isso não é aprovação
automática: registre como dívida, com o caminho de convergência. Padrão novo
convivendo com padrão velho sem plano é como APIs se tornam impossíveis de
aprender.

**Perguntas de revisão manual:**

- Alguém que integrou com três endpoints acerta o quarto sem ler a
  documentação?
- Duas operações parecidas se comportam de forma parecida em erro, paginação e
  formato?
- O que a fase introduziu é padrão novo ou exceção? Se é padrão, quem converte
  o resto, e quando?

---

## 13. OBSERVABILIDADE DO CONTRATO

*Referência: seção "Habilitar o rastreamento distribuído e o contexto de
rastreamento em APIs".*

- Identificador de correlação aceito na requisição (`Correlation-ID`,
  `X-Request-ID`, ou `traceparent` do padrão W3C) e **devolvido na resposta**;
- Ausente na requisição, o servidor gera e devolve;
- O mesmo identificador aparece no log do servidor e na resposta de erro — é
  ele que transforma "deu erro ontem" em uma requisição localizável;
- Propagado aos serviços internos que a requisição aciona;
- Métricas por endpoint e por status: contagem, latência, taxa de erro;
- Uso por versão medido — sem esse número, ninguém sabe quando é seguro
  desligar a versão antiga, e a depreciação nunca acontece;
- Log da API não registra credencial, token completo nem dado pessoal no corpo
  (cruza com `[[GATE_SECURITY]]` A09).

---

## 14. AVALIAÇÃO OBJETIVA POR ACHADO

Todo achado deste gate é reportado no formato abaixo, sem exceção:

```text
PROBLEMA ENCONTRADO
ENDPOINT
IMPACTO NO CONSUMIDOR
SEVERIDADE            BLOCKER | WARNING
DIMENSAO              nomenclatura | metodo | status | idempotencia |
                      paginacao | filtro | ordenacao | versionamento |
                      compatibilidade | erro | resposta | contrato |
                      content-type | autenticacao | autorizacao |
                      rate-limit | cache | concorrencia | observabilidade |
                      documentacao | seguranca | consistencia
REFERENCIA            seção deste gate, e o ponto da referência Microsoft
ESTADO ATUAL
ESTADO ESPERADO
CORRECAO APLICADA
QUEBRA CONTRATO?      sim | nao — se sim, como foi versionado
```

Preenchimento:

- **Impacto no consumidor** — o que quebra, ou o que fica mais difícil, para
  quem chama. Se não dá para descrever, reavalie se o achado existe;
- **Estado atual** — o que a chamada devolveu de fato, citado;
- **Estado esperado** — o comportamento concreto, com o código de status e o
  formato. Nunca "mais consistente";
- **Correção aplicada** — o que foi feito. `NAO CORRIGIDO` é resposta válida
  para WARNING aceito, com justificativa;
- **Quebra contrato?** — se a correção altera contrato publicado, ela própria é
  uma quebra e precisa da estratégia de versionamento da § 9.2. Corrigir um
  defeito de design quebrando o cliente em silêncio troca um problema por um
  pior.

---

## 15. PROIBIÇÕES

Durante a correção dos achados deste gate é proibido:

- Alterar contrato publicado sem versionar, mesmo para corrigir um erro de
  design;
- Remover campo, endpoint ou versão sem período de depreciação anunciado;
- Documentar comportamento que a API não tem, ou "corrigir" divergência
  ajustando a especificação em vez do código, quando o código é que está
  errado;
- Marcar item da § 12 como conferido sem comparar com os endpoints existentes;
- Desativar validação de contrato ou de especificação para o gate passar;
- Devolver erro com status 200 para não quebrar cliente mal escrito — corrija o
  contrato e versione;
- Aceitar ausência de paginação em coleção porque "hoje tem poucos registros";
- Expor campo interno para resolver rapidamente uma necessidade de tela;
- "Resolver" achado com comentário ou TODO prometendo corrigir depois.

Violação destas proibições é BLOCKER e deve ser revertida.

---

## 16. RE-EXECUÇÃO E LIMITE DE ITERAÇÕES

Depois de qualquer correção, execute novamente as verificações afetadas —
inclusive as chamadas reais. Nunca considere corrigido um status code ou um
formato de erro por inspeção do código.

```text
MAX_GATE_ITERATIONS = 3
```

Uma iteração = ciclo completo de (verificar → corrigir → re-verificar).

Se após 3 iterações ainda existirem BLOCKERs:

1. PARE de corrigir;
2. emita STATUS = FAIL;
3. liste os BLOCKERs remanescentes no formato da seção 14;
4. explique o que foi tentado em cada iteração e por que não convergiu;
5. recomende o próximo passo (nova versão da API, ADR de convenção, redesenho
   do recurso, decisão do dono sobre quebrar compatibilidade).

Achado de consistência que exige converter a API inteira **não se resolve
dentro da fase**: registre como dívida com plano e ADR, e não deixe o gate
iterar contra um problema que é de outra escala.

---

## 17. RESULTADO FINAL

Ao finalizar, produza obrigatoriamente:

```text
API DESIGN QUALITY GATE
STATUS: PASS | FAIL
ITERACOES EXECUTADAS: N / 3
ENDPOINTS AVALIADOS: N (novos: N, alterados: N, removidos: N)
QUEBRAS DE CONTRATO: N (versionadas: N)
```

### ESCOPO DA REVISÃO
Endpoints tocados, seções aplicáveis (tabela da seção 1), seções `N/A`, e quem
consome esta API.

### EVIDÊNCIAS
Cada chamada e comando executado, com a saída relevante — status, cabeçalhos,
corpo resumido. Item sem evidência = `NOT_MEASURED`.

### CONFORMIDADE

- Recursos, endpoints e nomenclatura
- Métodos HTTP e status codes
- Idempotência
- Content-Type e negociação
- Cache e concorrência
- Estrutura das respostas
- Tratamento de erros
- Paginação, filtros e ordenação
- Contratos e compatibilidade retroativa
- Versionamento
- Autenticação e autorização
- Limitação de taxa
- Observabilidade
- Documentação e OpenAPI
- Segurança do contrato

*(para cada item: achados no formato da seção 14, ou "N/A — fora do escopo")*

### QUADRO DE CONSISTÊNCIA
A tabela da seção 12, preenchida.

### COMPATIBILIDADE

```text
MUDANCA   TIPO (compativel|quebra)   VERSIONADA   CONSUMIDOR AFETADO
```

### BLOCKERS
Cada um no formato da seção 14, com status (resolvido | remanescente).

### WARNINGS ACEITOS
Cada um com justificativa re-argumentada do zero.

### DÍVIDA DE DESIGN
Divergência que a fase não podia resolver sozinha, com o plano de convergência
e o ADR que a registra.

### VEREDITO

A fase somente pode ser marcada como concluída quando:

```text
STATUS = PASS
```

`STATUS = PASS` exige: zero BLOCKERs remanescentes, todo WARNING aceito com
justificativa registrada, especificação OpenAPI correspondendo ao comportamento
real, e toda quebra de contrato versionada e anunciada.

Nunca invente comportamento de endpoint. Quando uma verificação não puder ser
executada — ambiente indisponível, dependência externa, dado de produção
necessário — declare `NOT_MEASURED` e sinalize que a área ficou sem cobertura,
para que a revisão manual receba atenção redobrada.
