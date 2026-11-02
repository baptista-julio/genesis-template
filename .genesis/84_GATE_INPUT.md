# Quality Gate — Tratamento de Input

> Prompt do pacote Genesis. Referências `[[ID]]` resolvem pelo `MANIFEST.md`.
> Executado depois de `[[GATE_API]]` e `[[GATE_FRONTEND]]` (os que se aplicam)
> retornarem PASS, antes de `[[GATE_SECURITY]]` e `[[GATE_CODE]]`.
> Aplicável a toda fase que cria ou altera um ponto por onde dado externo entra
> no sistema. Fase que não toca nenhuma fronteira de entrada marca o gate como
> `N/A — fora do escopo`, com a justificativa baseada no diff.
> Os limites de tamanho e de taxa vêm de `[[DOC_REQUISITOS_NF]]`; o contrato de
> entrada das rotas, de `[[DIR_API]]`; as fronteiras entre módulos, de
> `[[DOC_ARQUITETURA]]`. Quando o projeto definir convenção própria em ADR, **a do
> projeto prevalece** — registre a divergência no relatório final.

Você acabou de concluir uma fase de implementação **e já passou pelos gates de
API e de frontend, quando se aplicam**. Este gate avalia uma única coisa: **o que acontece com
o dado antes de o sistema confiar nele**.

O `[[GATE_SECURITY]]`, que vem em seguida, pergunta se a alteração tem vulnerabilidades conhecidas.
Este gate faz a pergunta inversa e mais estreita: para cada ponto onde dado de
fora entra, existe validação, limite, normalização e codificação — e ela está no
lugar certo, no servidor, antes do primeiro uso?

Atue como um **Senior AppSec Engineer independente**, com a tarefa de mandar
entrada hostil para cada fronteira e ver o que passa.

---

## PRINCÍPIO

**Todo input externo é não confiável.** Sem exceção por origem, por camada ou
por quem chamou.

- Dado validado em outra camada **continua não confiável** nesta. A validação do
  frontend é conveniência de usuário, não controle de segurança: quem ataca não
  usa o frontend.
- Dado vindo de sistema interno, fila, banco ou serviço "de confiança" é tratado
  como externo se a sua origem puder ser influenciada por terceiro.
- Dado gravado seguro hoje pode ser lido em contexto diferente amanhã. A
  validação é na **entrada** e a codificação é na **saída**, para o contexto de
  destino.
- A decisão de aceitar é um **allowlist** (o que é permitido), nunca um
  denylist (o que é proibido). Lista de caracteres ruins sempre esquece um.

---

## REGRAS DE INDEPENDÊNCIA DA REVISÃO

1. **Rastreie o dado da fronteira até o uso.** Obtenha o diff real via
   ferramenta e, para cada dado externo, siga o caminho: onde entra, por onde
   passa, onde é consumido (query, comando, caminho de arquivo, template,
   resposta, log, chamada de rede). Validação que existe em um ponto do caminho
   mas é contornável por outro caminho não conta.

2. **Justificativas da implementação não valem.** "Esse campo já vem validado do
   cliente", "só o administrador chama isso", "o gateway filtra antes" — nenhuma
   é evidência. Reverifique no código desta camada.

3. **Toda constatação vem de evidência real**: teste com payload hostil
   executado, ferramenta rodada, ou leitura literal do trecho citado. Nunca
   declare uma entrada "tratada" porque existe uma função com nome de
   validação — leia o que ela faz e teste o que ela deixa passar.

4. **Não assuma tecnologia ausente.** Aplique somente os vetores da § 5 que o
   projeto realmente tem. NoSQL injection em projeto sem NoSQL é `N/A`, não
   achado. O inverso vale: tecnologia presente cujo vetor ficou fora da revisão
   é lacuna do gate.

5. **Pense como atacante, não como autor.** Para cada fronteira: *"Qual o pior
   valor que eu poderia mandar aqui — o maior, o vazio, o nulo, o de tipo
   errado, o com caractere de controle, o que parece válido mas significa outra
   coisa?"*

---

## 1. ESCOPO E MAPA DE FRONTEIRAS

Antes de aplicar os checklists, determine, a partir do diff real e do código
existente que ele alcança:

- quais pontos de entrada a fase criou ou alterou;
- de cada ponto, qual a origem do dado e quem consegue influenciá-la;
- qual o destino do dado: banco, sistema de arquivos, processo, rede, template,
  resposta ao cliente, log, fila.

### 1.1 Fontes a considerar

Considere cada uma que o projeto de fato tem. Todas são não confiáveis.

| Origem | Exemplos |
|---|---|
| Requisição | parâmetro de rota, query string, corpo, formulário, cabeçalho, cookie |
| Arquivo recebido | upload, anexo, importação de planilha ou CSV, imagem, arquivo compactado |
| Integração | resposta de API externa, webhook, mensagem de fila ou tópico, evento |
| Cliente | dado enviado pelo frontend, aplicativo móvel, extensão, CLI |
| Ambiente | variável de ambiente, argumento de linha de comando, arquivo de configuração |
| Armazenado | campo gravado por usuário e lido em outro contexto ou por outro usuário |

Variável de ambiente e arquivo de configuração entram na lista porque quem
controla o ambiente de execução controla o comportamento do sistema. Tratar não
significa desconfiar de tudo na inicialização: significa validar tipo, formato e
faixa no carregamento e **falhar na subida** se o valor for inválido, em vez de
seguir com um padrão perigoso.

### 1.2 Quais seções se aplicam

| O diff toca em... | Seções obrigatórias |
|---|---|
| Qualquer ponto de entrada novo ou alterado | 3, 4, 8 |
| Valor que chega a query, filtro ou comando de banco | 5.1, 5.2 |
| Valor que chega a processo, shell ou interpretador | 5.4 |
| Valor usado em caminho, nome de arquivo ou extração | 5.5, 5.9 |
| URL, host ou endereço fornecido de fora, chamada de rede com destino variável | 5.6 |
| Valor renderizado em página, e-mail, PDF ou template | 5.3, 5.7 |
| Objeto montado a partir do corpo da requisição, ORM, binding automático | 5.8 |
| Upload ou recebimento de arquivo | 5.9 |
| Desserialização, parser de XML, YAML, JSON, formato binário | 5.10 |
| Variável de ambiente, configuração, argumento de linha de comando | 6 |
| Dado que entra em log ou cabeçalho de resposta | 5.11 |

Seções fora do escopo entram no relatório como `N/A — fora do escopo desta
alteração` ou `N/A — tecnologia ausente no projeto`, nunca omitidas.

---

## 2. CLASSIFICAÇÃO DE SEVERIDADE

### BLOCKER — reprova o gate (STATUS = FAIL)

- Ponto de entrada sem validação **no servidor**, ainda que o frontend valide;
- Dado externo concatenado ou interpolado em query SQL, filtro NoSQL, comando de
  shell, caminho de arquivo, template ou expressão avaliada;
- Entrada sem limite superior de tamanho: corpo, campo, array, profundidade de
  aninhamento, número de itens, tamanho de arquivo, número de partes;
- Objeto de domínio ou de persistência preenchido diretamente do corpo da
  requisição, sem lista explícita de campos aceitos (mass assignment);
- Caminho de arquivo montado com dado externo sem normalização e confinamento a
  um diretório base;
- Requisição de rede a destino controlado por dado externo, sem allowlist de
  destino e sem bloqueio de endereços internos, de loopback e de metadados de
  nuvem (SSRF);
- Dado externo renderizado em HTML, atributo, script, URL ou CSS sem
  codificação correta para o contexto, ou com sanitização desativada;
- Upload aceito sem verificação de tamanho, de tipo real do conteúdo e de
  destino de gravação seguro; arquivo enviado gravado onde pode ser executado ou
  servido com o tipo declarado pelo cliente;
- Desserialização de dado externo em formato que instancia objetos arbitrários
  (`pickle`, `yaml.load` sem `SafeLoader`, `ObjectInputStream`, `unserialize`);
- Parser de XML com entidades externas ou DTD habilitados (XXE);
- Redirecionamento para URL fornecida de fora sem allowlist;
- Falha de validação tratada como aceitação: exceção engolida, `catch` vazio,
  erro convertido em "valor padrão" que segue o fluxo (*fail-open*);
- Validação aplicada **depois** do uso, ou sobre uma cópia diferente da que é
  usada (verificar e usar valores distintos);
- Variável de ambiente ou configuração que vira comando, caminho ou consulta sem
  validação, ou cujo valor inválido faz o sistema subir em modo inseguro;
- Secret impresso em log, mensagem de erro ou resposta durante o tratamento de
  entrada inválida;
- Teste de validação removido, enfraquecido ou ignorado para o gate passar.

### WARNING — reportado e justificado, não reprova sozinho

- Validação de formato presente mas mais permissiva que o necessário (aceita
  `.*` onde cabe um padrão estreito);
- Mensagem de validação que ecoa o valor recebido sem codificação em contexto
  de baixo risco;
- Ausência de normalização (Unicode, caixa, espaços) onde a comparação não
  decide acesso nem identidade;
- Padrão de expressão regular suscetível a backtracking catastrófico, em ponto
  de baixa exposição;
- Limite de tamanho presente mas alto demais para o uso real do campo;
- Ausência de rejeição de campos desconhecidos no corpo, onde o binding já usa
  lista explícita;
- Validação duplicada e divergente entre dois pontos, com o servidor correto;
- Ausência de teste automatizado de entrada hostil em ponto de risco baixo.

**Escalonamento:** um WARNING vira BLOCKER quando (a) se repete de forma
sistemática, indicando ausência de padrão central de validação; (b) já foi
apontado em iteração anterior e ignorado sem justificativa; (c) atinge fluxo de
autenticação, pagamento, dado pessoal sensível ou endpoint público sem
autenticação.

---

## 3. CONTROLES UNIVERSAIS

Aplicam-se a **todo** ponto de entrada do mapa da § 1, independentemente do
vetor. Cada item verificado cita o trecho de código que o implementa.

### 3.1 Local da validação

- A validação decisiva ocorre **no backend**, na borda do módulo, antes de
  qualquer regra de negócio ou acesso a recurso;
- Existe um ponto central (esquema, DTO, validador, middleware) ou o padrão
  está repetido à mão em cada rota? Padrão repetido à mão é onde um ponto
  novo escapa;
- Código interno chamado por outro módulo **não presume** que o chamador
  validou. Se a função é pública para outro módulo, ela valida o que recebe ou
  recebe um tipo que só se constrói válido.

### 3.2 Tipo

- O tipo é verificado, não presumido: número é número, não texto numérico; texto
  não é objeto nem array (relevante em NoSQL e em parsers frouxos);
- Valores especiais rejeitados ou tratados de propósito: nulo, vazio, `NaN`,
  infinito, número negativo onde só cabe positivo, booleano em forma de texto;
- Conversão de tipo é explícita e falha de forma visível quando o dado não
  converte.

### 3.3 Formato

- O formato é descrito por padrão estreito (identificador, e-mail, data, UUID,
  telefone, URL) validado por **biblioteca de parsing** quando existe, não por
  expressão regular improvisada;
- Datas e horários têm formato e fuso definidos;
- Identificadores têm alfabeto e comprimento exatos.

### 3.4 Limites de tamanho

Todo dado tem teto, e o teto está no código, não só na documentação:

| O quê | Limite verificado |
|---|---|
| Corpo da requisição | tamanho total |
| Campo de texto | comprimento máximo, em caracteres **e** em bytes após codificação |
| Array e coleção | número máximo de itens |
| Objeto aninhado | profundidade máxima |
| Parâmetro de paginação | teto de `limit`/`page_size` |
| Arquivo recebido | tamanho máximo, número máximo de arquivos e partes |
| Arquivo compactado | tamanho **descompactado** e número de entradas (bomba de descompressão) |
| Cabeçalho e cookie | tamanho e quantidade |
| Operação de custo variável | tempo máximo, e limite de taxa quando exposta |

O teto vem de `[[DOC_REQUISITOS_NF]]` quando existir; senão, é decisão
registrada na revisão com a justificativa.

### 3.5 Valores permitidos

- Campo que aceita um conjunto finito (estado, papel, tipo, ordem, campo de
  ordenação) valida contra a **lista fechada**, não contra "parece válido";
- Nome de coluna, direção de ordenação e campo de filtro vindos de fora são
  mapeados por tabela, nunca interpolados.

### 3.6 Normalização

- A validação roda sobre a **forma canônica**: Unicode normalizado, codificação
  de percentual decodificada uma única vez, espaços e caixa tratados quando a
  comparação importa;
- Decodificar duas vezes (dupla codificação) abre contorno; decodificar depois
  de validar abre contorno;
- Comparação de identidade ou de permissão usa a forma normalizada dos dois
  lados.

### 3.7 Encoding e codificação de saída

- A codificação de caracteres de entrada é fixa e verificada; sequência
  inválida é rejeitada, não "corrigida" em silêncio;
- A **saída** é codificada para o contexto de destino: HTML, atributo, JavaScript,
  URL, CSS, SQL parametrizado, argumento de processo, JSON. Um mesmo dado exige
  codificações diferentes em destinos diferentes;
- Cabeçalho de resposta que contém dado externo rejeita quebra de linha.

### 3.8 Sanitização

- Sanitizar é último recurso, para o caso em que o dado precisa aceitar marcação
  (HTML rico, Markdown). Quando cabe rejeitar, rejeita-se;
- Sanitização usa biblioteca mantida e especializada, com allowlist de
  marcação — nunca expressão regular que remove `<script>`;
- Dado sanitizado não é tratado como confiável a partir dali: segue a codificação
  de saída do contexto.

### 3.9 Resposta à entrada inválida

- Rejeita-se com erro de cliente específico (4xx), antes de qualquer efeito
  colateral;
- A mensagem diz **qual campo** e **qual regra**, sem expor detalhe interno nem
  secret, e sem ecoar o valor recebido sem codificação;
- Nenhum caminho de erro devolve sucesso, nenhum valor padrão silencioso
  substitui o inválido;
- A falha de validação é registrada em log **sem** o conteúdo sensível recebido.

---

## 4. ORDEM DO TRATAMENTO

A sequência importa. Verifique que o código segue esta ordem; inversão é
achado.

```text
1. Limitar    corte por tamanho ANTES de ler ou parsear (corpo, upload, cabeçalho)
2. Decodificar  uma vez, na codificação esperada
3. Normalizar   forma canônica
4. Validar      tipo, formato, faixa, valores permitidos
5. Autorizar    esse chamador pode fazer isso com esse recurso (ver GATE_SECURITY A01)
6. Usar         com API segura para o destino (parametrizada, sem shell, confinada)
7. Codificar    na saída, para o contexto de destino
```

Defeitos típicos: validar antes de normalizar, ler o corpo inteiro antes de
checar o tamanho, usar o valor original depois de validar o normalizado,
autorizar sobre o identificador bruto.

---

## 5. VETORES

Aplique somente os que a tecnologia do projeto torna possíveis, conforme a
Regra 4. Para cada um aplicável: o que verificar, o que contar como evidência.

### 5.1 SQL Injection

- Toda query usa **parâmetros vinculados** ou o construtor do ORM; nunca
  concatenação, interpolação de texto ou `format`;
- ORM não é salvo-conduto: `raw`, `execute`, `literal`, fragmentos de texto em
  `order by`, `group by`, nome de tabela e de coluna interpolados são o mesmo
  defeito;
- Nome de coluna e direção de ordenação vindos de fora passam por lista fechada;
- `LIKE` com valor externo escapa `%` e `_` quando o significado literal importa;
- Procedimento armazenado que monta SQL dinâmico por dentro herda o defeito.

**Evidência:** busca no diff e no código alcançado por consultas construídas
dinamicamente; teste com payload clássico (`' OR '1'='1`, `'; --`, aspas
desbalanceadas) e a resposta observada.

### 5.2 NoSQL Injection

- Em consulta baseada em objeto, o filtro **nunca** é o objeto vindo do corpo.
  Um campo esperado como texto recebendo `{"$ne": null}` ou `{"$gt": ""}` muda a
  consulta;
- O tipo de cada campo do filtro é verificado como escalar antes do uso;
- Operadores de consulta (`$where`, `$function`, execução de JavaScript no
  servidor) estão desabilitados ou nunca recebem dado externo;
- Chaves com `$` ou `.` vindas de fora são rejeitadas ou removidas.

**Evidência:** teste enviando objeto de operador onde se espera texto; resposta
deve ser erro de validação, não resultado alterado.

### 5.3 Cross-Site Scripting (XSS)

- Todo dado externo exibido em página é codificado para o contexto (HTML,
  atributo, JavaScript, URL, CSS) pelo mecanismo do framework, **com o
  escape automático ativo**;
- Os pontos que desligam o escape são os suspeitos: `innerHTML`,
  `dangerouslySetInnerHTML`, `v-html`, `|safe`, `{{{ }}}`, `document.write`,
  `eval`, `setTimeout` com texto, `href`/`src` com valor externo;
- URL externa em `href` e `src` aceita apenas esquemas permitidos (`http`,
  `https`, `mailto`); `javascript:` e `data:` são rejeitados;
- HTML rico passa por sanitizador de allowlist (§ 3.8);
- XSS armazenado: o dado gravado limpo hoje é codificado na **saída** de cada
  tela que o exibe;
- Política de segurança de conteúdo (CSP) presente reduz o dano, mas não
  substitui a codificação.

**Evidência:** teste com `<script>`, `"><img onerror=...>`, `javascript:` e
verificação do HTML realmente emitido.

### 5.4 Command Injection

- Nenhum dado externo chega a shell. Processo externo é chamado **sem shell**,
  com lista de argumentos (`execve`-style), nunca com texto único
  (`shell=True`, `exec`, `system`, `Runtime.exec(string)`, crases);
- Argumento vindo de fora que começa com `-` é tratado como opção: use `--`
  para separar ou rejeite;
- O executável é fixo ou escolhido de lista fechada;
- Variável de ambiente do processo filho é montada explicitamente, sem herdar
  tudo.

**Evidência:** busca por chamadas de processo e de shell no diff e no código
alcançado; teste com `; id`, `$(id)`, `&& id`, `| id`, `\n id`.

### 5.5 Path Traversal

- Nome de arquivo recebido **nunca** compõe um caminho diretamente;
- Quando o caminho precisa de parte externa: junte ao diretório base, resolva
  para o caminho canônico (sem `..`, sem link simbólico) e **confirme que o
  resultado continua dentro do diretório base**. Rejeitar `..` por substring
  não basta;
- Preferível: guardar o arquivo com nome gerado pelo servidor e manter o nome
  original só como metadado;
- Extração de arquivo compactado confere o destino de cada entrada (zip slip);
- Bytes nulos e separadores de ambos os sistemas operacionais tratados.

**Evidência:** teste com `../../etc/passwd`, `..%2f`, `%2e%2e/`, `....//`, nome
absoluto, nome com byte nulo, link simbólico.

### 5.6 SSRF — Server-Side Request Forgery

- Destino de requisição vindo de fora (URL, host, webhook de cliente, importação
  por link, pré-visualização, redirecionamento seguido) passa por **allowlist de
  destino**;
- Bloqueio de faixas internas: loopback, redes privadas, link-local, endereço de
  metadados de nuvem, e as formas alternativas de escrevê-los (decimal, octal,
  hexadecimal, IPv6, nome que resolve para interno);
- A verificação ocorre sobre o **endereço resolvido** no momento da conexão,
  não só sobre o texto da URL — senão um DNS que muda entre a checagem e o uso
  (rebinding) contorna;
- Redirecionamentos não são seguidos às cegas, ou cada salto é revalidado;
- Esquemas restritos a `http`/`https`; `file`, `gopher`, `ftp` rejeitados;
- Tempo limite e tamanho máximo de resposta definidos.

**Evidência:** teste com `http://127.0.0.1`, `http://169.254.169.254`,
`http://[::1]`, `http://2130706433`, redirecionamento para interno.

### 5.7 Template Injection (SSTI) e injeções em formatos de saída

- Dado externo nunca é **parte do texto do template**; ele só é passado como
  variável ao template. `render(f"Olá {nome}")` é injeção, `render("Olá {{nome}}",
  nome=nome)` não;
- Sandbox do motor de template ativo quando houver template editável por
  usuário;
- Mesma lógica para outros formatos: cabeçalho de e-mail (quebra de linha em
  assunto e destinatário), CSV (células iniciadas por `=`, `+`, `-`, `@` viram
  fórmula na planilha), LDAP, XPath, expressão regular montada com dado externo.

**Evidência:** teste com `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`; resposta não pode
conter `49`.

### 5.8 Mass Assignment

- Objeto de domínio ou de persistência **não** é preenchido com o corpo inteiro
  da requisição. Há um objeto de entrada (DTO) com lista explícita dos campos
  aceitos, separado da entidade;
- Campos que o cliente jamais define — papel, permissão, `id`, dono, preço
  calculado, estado interno, flag de administrador, data de criação — não
  existem no tipo de entrada;
- Atualização parcial (`PATCH`) vale a mesma regra: só os campos permitidos para
  aquele chamador;
- Campos desconhecidos no corpo são rejeitados ou ignorados de forma explícita,
  nunca repassados adiante.

**Evidência:** teste enviando campo privilegiado extra (`"role":"admin"`,
`"is_admin":true`, `"owner_id":...`) e verificação do estado persistido.

### 5.9 Uploads maliciosos

- Limite de tamanho aplicado **durante** a leitura (fluxo), não depois de
  carregar o arquivo inteiro em memória;
- Tipo decidido pelo **conteúdo** (assinatura do arquivo), não pelo
  `Content-Type` nem pela extensão informados pelo cliente; extensão checada
  contra allowlist;
- Nome do arquivo gerado pelo servidor; o original, quando guardado, é só dado
  de exibição e é codificado na saída;
- Gravação fora da raiz pública e sem permissão de execução. Se precisa ser
  servido, é servido com `Content-Type` fixo e `Content-Disposition`/
  `X-Content-Type-Options: nosniff` conforme o caso;
- Imagem reprocessada (decodificada e recodificada) quando o destino é exibi-la,
  para descartar carga embutida; metadados removidos quando sensíveis;
- Arquivo compactado e formato de documento com limites de descompressão, de
  número de entradas e de aninhamento;
- Quando o risco justificar, análise por antivírus ou isolamento do
  processamento.

**Evidência:** teste com arquivo acima do limite, extensão dupla
(`.php.jpg`), `Content-Type` falso, conteúdo que não corresponde à extensão,
nome com `../`.

### 5.10 Desserialização insegura e parsers

- Dado externo só é desserializado em formato **sem instanciação arbitrária de
  tipo**: JSON, e formatos com esquema. Proibido para dado externo: `pickle`,
  `marshal`, `yaml.load` genérico, `ObjectInputStream`, `unserialize`,
  `BinaryFormatter`, `eval` de literal;
- Parser de XML com DTD e entidades externas **desabilitados** (XXE, bomba de
  entidades);
- JSON validado contra esquema depois de parseado: o parse só garante sintaxe;
- Chaves como `__proto__`, `constructor` e `prototype` em objeto mesclado de
  fora não alcançam o protótipo (poluição de protótipo);
- Profundidade e tamanho do documento limitados antes do parse.

**Evidência:** busca por chamadas de desserialização no diff e no código
alcançado; configuração do parser citada.

### 5.11 Injeção em log e em cabeçalho

- Dado externo em mensagem de log tem quebra de linha e caracteres de controle
  neutralizados (falsificação de entrada de log), preferencialmente via log
  estruturado em que o valor é campo, não texto concatenado;
- Dado externo em cabeçalho de resposta ou em `Set-Cookie` rejeita `\r` e `\n`
  (divisão de resposta);
- Nenhum secret, token, senha ou dado pessoal sensível recebido como entrada vai
  para log, inclusive no caminho de erro de validação.

### 5.12 Outros vetores do contexto tecnológico

Percorra e aplique os que existirem no projeto; os demais são `N/A`:

- redirecionamento aberto (parâmetro `next`, `return_url`, `redirect`);
- expressões regulares com backtracking catastrófico (ReDoS) sobre entrada
  externa;
- poluição de parâmetros HTTP (mesmo parâmetro repetido);
- injeção LDAP, XPath, GraphQL (profundidade e custo da consulta, introspecção),
  gRPC e protocolos binários (campos de tamanho);
- CSRF em requisição que altera estado com credencial ambiente (cookie);
- inclusão de arquivo e carga dinâmica de módulo por nome externo;
- comparação de valor sensível sem tempo constante;
- confusão de tipo de conteúdo (`Content-Type` aceito diferente do esperado).

Vetor que o projeto tem e que não está na lista **também** é avaliado: a lista
é piso, não teto.

---

## 6. VARIÁVEIS DE AMBIENTE E CONFIGURAÇÃO

Aplica-se a toda leitura de variável de ambiente, argumento de linha de
comando e arquivo de configuração.

- Cada valor é lido por **um único módulo de configuração** que valida tipo,
  formato e faixa e entrega um objeto tipado ao resto do código;
- Valor ausente ou inválido **impede a subida** com mensagem clara. Não há
  padrão silencioso que enfraqueça segurança (modo debug ligado, autenticação
  desligada, verificação de certificado desativada);
- O valor da configuração não é concatenado em comando, caminho ou consulta sem
  passar pelos controles do § 5;
- O nome do arquivo de configuração indicado de fora segue o confinamento da
  § 5.5;
- Arquivo de configuração é lido com parser seguro (§ 5.10), e o conteúdo dele
  não executa código;
- Secret vem do ambiente ou de cofre, **nunca** do repositório, e o valor nunca
  é impresso: nem na subida, nem no erro de validação, nem no log de
  configuração carregada. Ao mostrar a configuração efetiva, mascare o que é
  secret.

---

## 7. TESTES DE ENTRADA HOSTIL

Constatação sem execução é `NOT_MEASURED`. Para cada fronteira do mapa da § 1,
exercite com entrada hostil e registre a resposta real.

Conjunto mínimo por campo, escolhendo os que se aplicam ao tipo:

| Categoria | Valores |
|---|---|
| Vazio e ausente | campo ausente, `""`, `null`, espaços |
| Tipo errado | número onde cabe texto, texto onde cabe número, array, objeto |
| Limite | exatamente no teto, teto + 1, muito acima (megabytes) |
| Estrutura | aninhamento profundo, array enorme, chave duplicada, campo extra |
| Caracteres | Unicode exótico, caracteres de controle, byte nulo, quebra de linha, RTL |
| Codificação | percentual, dupla codificação, sequência UTF-8 inválida |
| Payload por vetor | os da § 5 correspondentes |

O resultado esperado é sempre **rejeição clara e antes do efeito**: erro de
cliente, nenhum dado gravado, nenhuma chamada de rede feita, nenhum detalhe
interno devolvido.

Quando o projeto já tem testes, **acrescente casos hostis aos testes
existentes** do ponto de entrada alterado, no padrão do projeto. Verificação
manual de uma vez só não protege a próxima alteração.

Ferramentas, quando disponíveis na stack (instale uma leve antes de declarar
`NOT_MEASURED`): `semgrep` e o linter de segurança do ecossistema para
localizar usos perigosos; geração de casos por propriedade (`hypothesis`,
`fast-check`) ou fuzzing leve para campos de formato livre; ferramenta de
varredura dinâmica contra instância local quando o projeto expõe HTTP.

---

## 8. FRONTEND NÃO É CONTROLE

Verificação específica, porque é onde o erro mais comum acontece.

- Para cada validação existente no frontend, localize a **equivalente no
  backend**. Ausente é BLOCKER;
- Atributos de formulário (`maxlength`, `pattern`, `required`, `type=email`),
  campos ocultos e desabilitados, opções de lista suspensa e valores calculados
  no cliente são todos editáveis por quem ataca: o servidor os revalida;
- Dado "calculado" pelo cliente (total, desconto, preço, papel) é recalculado ou
  consultado no servidor, nunca aceito;
- O cliente nunca decide o que o usuário pode fazer: botão escondido não é
  autorização;
- Mensagem de erro de validação do servidor tem o mesmo rigor da do cliente e é
  tratada pelo cliente como texto, nunca como HTML.

---

## 9. PROIBIÇÕES

Durante a correção dos achados deste gate é proibido:

- Resolver um achado apenas no frontend;
- Resolver com denylist (lista de caracteres ou palavras proibidas) quando cabe
  allowlist;
- Remover ou enfraquecer validação, limite ou teste para o gate passar, ou
  afrouxar a regra para que um caso hostil seja aceito;
- Engolir a exceção de validação ou convertê-la em valor padrão que segue o
  fluxo;
- Sanitizar com expressão regular caseira onde existe biblioteca especializada;
- Desligar o escape do framework, o modo seguro do parser ou a verificação de
  certificado para "resolver rápido";
- Adicionar supressão de regra de SAST ou de linter de segurança sem
  justificativa explícita registrada;
- "Resolver" um achado com comentário, `TODO` ou documentação prometendo
  tratar depois;
- Declarar tratado um ponto de entrada sem executar a entrada hostil contra ele;
- Mascarar a falha para obter PASS: reclassificar BLOCKER como WARNING sem
  justificativa técnica registrada, ou redefinir o escopo para excluir o ponto
  que falha;
- Registrar em log, em teste ou em relatório um secret real usado para exercitar
  o ponto de entrada.

Violação destas proibições é BLOCKER e deve ser revertida.

---

## 10. RE-EXECUÇÃO E LIMITE DE ITERAÇÕES

Toda correção é seguida de **nova validação**: repita a entrada hostil que
provou o defeito e todas as que passam pelo mesmo caminho. Nunca considere
corrigido por inspeção visual do código.

```text
MAX_GATE_ITERATIONS = 3
```

Uma iteração = ciclo completo de (verificar → corrigir → re-verificar).

Se após 3 iterações ainda existirem BLOCKERs:

1. PARE de corrigir;
2. emita STATUS = FAIL;
3. liste os BLOCKERs remanescentes com fronteira, vetor, arquivo, linha e a
   entrada que explora;
4. explique o que foi tentado em cada iteração e por que não resolveu;
5. recomende o próximo passo (camada central de validação, redesenho do
   contrato de entrada, revisão humana).

Defeito que exige introduzir uma camada central de validação em todo o sistema
**não se resolve dentro da fase**: registre como dívida com plano e ADR, corrija
o ponto de entrada da fase e não deixe o gate iterar contra um problema de
outra escala.

---

## 11. RASTREABILIDADE

Cada achado e cada correção ficam rastreáveis até a alteração que os causou e a
que os resolveu:

- o relatório cita arquivo e linha de cada fronteira avaliada;
- cada correção é registrada com o achado a que responde e o teste que a
  prova;
- o teste hostil novo fica no repositório, no padrão de testes do projeto, para
  que a regressão seja pega depois;
- a entrada em `[[DOC_CHANGELOG]]` da fase cita a classe de achado corrigida,
  sem reproduzir payload funcional nem secret.

---

## 12. FORMATO DO ACHADO

```text
[BLOCKER|WARNING] <fronteira>: <vetor ou controle>
  FRONTEIRA     origem do dado e ponto de entrada (arquivo:linha)
  DESTINO       onde o dado é consumido (arquivo:linha)
  ENTRADA       o que foi enviado para provar
  OBSERVADO     o que o sistema fez, citado
  ESPERADO      o comportamento concreto: status, campo, regra
  CORRECAO      o que foi feito, e o teste que a prova
  REVALIDACAO   resultado da repetição da entrada hostil
```

`OBSERVADO` e `REVALIDACAO` sem execução real tornam o achado `NOT_MEASURED`.

---

## 13. RESULTADO FINAL

Ao finalizar, produza obrigatoriamente:

```text
INPUT HANDLING QUALITY GATE
STATUS: PASS | FAIL
ITERAÇÕES EXECUTADAS: N / 3
FRONTEIRAS AVALIADAS: N (novas: N, alteradas: N)
FRONTEIRAS COM ENTRADA HOSTIL EXECUTADA: N de N   <- precisa ser N = N
```

### ESCOPO DA REVISÃO
Fronteiras tocadas, fontes envolvidas (§ 1.1), vetores aplicáveis, vetores
`N/A` com o motivo (fora do escopo ou tecnologia ausente).

### EVIDÊNCIAS
Cada comando, teste e ferramenta executados, com a saída relevante — entrada
enviada, status, resposta. Item sem evidência = `NOT_MEASURED`.

### MAPA DE FRONTEIRAS

```text
FRONTEIRA (arquivo:linha)   ORIGEM   DESTINO   VALIDA NO SERVIDOR   LIMITE   OBSERVAÇÃO
```

### CONTROLES UNIVERSAIS
Tipo, formato, tamanho, valores permitidos, normalização, encoding, sanitização
e resposta à entrada inválida — cada um com achados no formato da § 12, ou
"N/A — fora do escopo".

### VETORES
SQL, NoSQL, XSS, comando, caminho, SSRF, template, mass assignment, upload,
desserialização, log e cabeçalho, outros — cada um com achados, ou
"N/A — tecnologia ausente".

### VARIÁVEIS DE AMBIENTE E CONFIGURAÇÃO

### FRONTEND NÃO É CONTROLE
Validações do cliente e a equivalente no servidor, lado a lado.

### BLOCKERS
Lista com status de cada um (resolvido | remanescente).

### WARNINGS ACEITOS
Cada um com justificativa re-argumentada do zero.

### DÍVIDA
Defeito estrutural que a fase não podia resolver sozinha, com plano e ADR.

### RISCO RESIDUAL
O que não pôde ser exercitado e por quê.

### VEREDITO

A fase somente pode ser marcada como concluída quando:

```text
STATUS = PASS
```

`STATUS = PASS` exige: zero BLOCKERs remanescentes, entrada hostil executada
contra toda fronteira avaliada, todo WARNING aceito com justificativa
registrada, e todo achado corrigido seguido de nova validação.

Nunca invente achado nem declare "tratado" o que não foi testado. Quando uma
verificação não puder ser medida com as ferramentas disponíveis, tente instalar
uma ferramenta leve equivalente antes de declarar `NOT_MEASURED` — e nesse caso,
sinalize que a fronteira ficou sem cobertura automatizada, para que a revisão
manual receba atenção redobrada.
