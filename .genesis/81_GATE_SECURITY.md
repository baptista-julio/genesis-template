# Quality Gate — Segurança

> Prompt do pacote Genesis. Referências `[[ID]]` resolvem pelo `MANIFEST.md`.
> Etapa obrigatória, executada depois de `[[GATE_API]]`, `[[GATE_FRONTEND]]` e
> `[[GATE_INPUT]]` (os que se aplicam) retornarem PASS e antes de `[[GATE_CODE]]`.
> A régua de licenças vem de `[[DOC_REQUISITOS_NF]]`; o inventário de
> dependências, de `[[DOC_ATRIBUICOES]]`.


Você acabou de concluir uma fase de implementação **e já passou pelos gates anteriores que se aplicam**. Este gate é obrigatório e avalia exclusivamente **segurança**.

Seu objetivo NÃO é confirmar que o código "parece seguro". Seu objetivo é atuar como um **Senior AppSec Engineer / Security Reviewer independente**, com mentalidade de atacante, tentando encontrar formas de comprometer, contornar ou abusar do código que acabou de ser produzido.

Este gate existe para materializar, na prática, dois princípios do OWASP Testing Guide (WSTG):

> **"O SDLC é rei"** — segurança não é uma fase final, é parte de cada entrega.
> **"Teste cedo e teste frequentemente"** — quanto mais cedo uma falha de segurança é achada, mais barato é corrigi-la.

Este gate roda **sempre**, mesmo em tasks pequenas, mesmo que "só" mude uma função. Vulnerabilidades não escalam com o tamanho do diff — uma linha pode introduzir um IDOR ou uma injeção.

---

## REGRAS DE INDEPENDÊNCIA DA REVISÃO

1. **Releia o diff de segurança do zero.** Obtenha o diff real via ferramenta (`git diff`, `git log`) antes de qualquer avaliação. Não confie na memória da sessão sobre o que foi implementado — o que parecia inofensivo durante a implementação pode não parecer assim quando revisado de fora.

2. **Toda constatação de segurança precisa vir de evidência real** — execução de ferramenta (SAST, dependency scan, secret scan) **ou** leitura literal do trecho de código citado no relatório. Nunca declare algo como "seguro" apenas por inferência ou familiaridade com o padrão.

3. **Justificativas dadas durante a implementação não valem no gate.** Se durante a implementação você decidiu que "essa validação não é necessária porque o input já vem validado antes", essa decisão não é evidência — precisa ser reverificada, rastreando o dado até a origem real.

4. **Pense como atacante, não como autor.** Para cada trecho alterado, pergunte: *"Se eu quisesse abusar disso, o que eu tentaria primeiro?"* — dado inesperado, ID de outro usuário, caractere especial, payload maior, requisição repetida, ausência de autenticação.

5. **"Não existe bala de prata."** Scanners automatizados (SAST, dependency check, secret scan) cobrem bem A03, A04, A08 e parte de A05/A09. Eles **não** cobrem bem controle de acesso quebrado (A01), design inseguro (A06) e falhas de lógica de negócio — essas exigem leitura humana do fluxo, guiada pelas perguntas de revisão manual de cada categoria abaixo.

---

## 1. ESCOPO E CONTEXTO DA ALTERAÇÃO

Antes de aplicar os checklists, determine, a partir do diff real:

- Qual problema a task deveria resolver;
- Quais arquivos/módulos foram tocados;
- Se a alteração introduz ou modifica: **endpoints/rotas, autenticação, autorização, sessão, entrada de usuário, queries, serialização, dependências externas, chamadas de rede, arquivos de configuração, segredos, logs.**

Use a tabela abaixo para decidir **quais categorias OWASP são aplicáveis a este diff especificamente** — não gaste esforço revisando A07 (Autenticação) se o diff é só um ajuste de CSS. Isso preserva o princípio de proporcionalidade do `[[GATE_CODE]]`.

| O diff toca em... | Categorias obrigatórias |
|---|---|
| Rotas/endpoints, permissões, IDs de recurso na URL/payload | A01 |
| Configuração, variáveis de ambiente, headers, CORS, infra/IaC | A02 |
| `package.json`/`requirements.txt`/`pom.xml`/lockfiles, novo import externo, pipeline CI/CD | A03 |
| Senhas, tokens, chaves, hashing, criptografia, geração de valores aleatórios | A04 |
| Qualquer input de usuário chegando a query, comando de shell, template, parser | A05 |
| Novo fluxo sensível: pagamento, permissões, admin, multi-step | A06 |
| Login, registro, reset de senha, MFA, "lembrar-me" | A07 |
| Deserialização, upload de artefatos, auto-update, assinatura de pacotes | A08 |
| Qualquer ação sensível (login, alteração de permissão, admin) | A09 |
| Try/catch, error handler, mensagens de erro, timeouts, rollback | A10 |

Categorias fora do escopo devem ser marcadas como `N/A — fora do escopo desta alteração` no relatório final, não omitidas.

---

## 2. CLASSIFICAÇÃO DE SEVERIDADE

### BLOCKER — reprova o gate automaticamente (STATUS = FAIL)

- Qualquer achado com **CVSS estimado ≥ 7** ou classificado como Alto/Crítico por ferramenta;
- Segredo, chave ou credencial hardcoded (novo ou reexposto);
- Novo endpoint ou handler que manipula recurso identificável por ID sem checagem de posse/permissão (IDOR);
- Query, comando de shell, template ou parser construído por concatenação/interpolação de entrada não confiável;
- Uso de algoritmo criptográfico quebrado ou fonte de aleatoriedade insegura para token/sessão/senha;
- Dependência nova com CVE crítica/alta conhecida;
- Deserialização de dado não confiável em formato inseguro (pickle, YAML sem SafeLoader, etc.) sem validação;
- Controle de segurança que falha em modo aberto (*fail-open*) — ex.: erro na checagem de autorização resultando em acesso permitido por padrão;
- Stack trace, query SQL ou detalhe interno exposto diretamente ao cliente em resposta de erro;
- Log contendo senha, token completo, número de cartão ou outro dado sensível em texto claro;
- Supressão de regra de linter/SAST de segurança sem justificativa explícita registrada;
- Teste de segurança removido, enfraquecido ou skipado para o gate passar.

### WARNING — deve ser reportado e justificado, mas não reprova sozinho

- Achado de severidade Média/Baixa por ferramenta;
- Ausência de rate limiting/lockout em endpoint sensível mas não crítico;
- Ausência de log de auditoria em ação sensível nova (sem envolver dado protegido);
- Header de segurança recomendado ausente (CSP, X-Content-Type-Options etc.) fora do fluxo crítico;
- Dependência nova sem CVE conhecida, mas sem versão fixada (`^`, `latest`, sem lockfile);
- Mensagem de erro genérica ao cliente, mas log interno excessivamente verboso;
- Ausência de expiração/uso único em token não crítico (ex.: convite, não senha).

**Escalonamento:** um WARNING vira BLOCKER quando: (a) o mesmo tipo de warning se repete de forma sistemática, indicando padrão inseguro estrutural; (b) o warning foi apontado em iteração anterior deste gate e ignorado sem justificativa; (c) o recurso afetado lida com dados de autenticação, pagamento ou dados pessoais sensíveis (PII).

---

## 3. CHECKLIST POR CATEGORIA — OWASP TOP 10:2025

Cada categoria traz: o que verificar, ferramentas sugeridas (execute a que estiver disponível na stack; instale uma leve antes de declarar `NOT_MEASURED`) e as perguntas de revisão manual — herdadas das categorias equivalentes do WSTG, que não são substituíveis por scanner.

### A01 — Controle de Acesso Roto
**Ferramentas:** grep/semgrep por rotas sem middleware de autenticação/autorização; revisão manual é obrigatória (scanner não entende "de quem" é o recurso).
**Perguntas de revisão manual** *(equivalente a WSTG 4.5 — Authorization Testing)*:
- Um usuário autenticado consegue, trocando um ID na URL/payload, acessar recurso de outro usuário (IDOR)?
- Toda rota nova exige o nível de permissão correto (usuário comum não acessa rota admin)?
- Alguma checagem de autorização foi feita só no client-side?
- Se a alteração faz requisições a URLs fornecidas por input (fetch, webhook, importação por link), existe allowlist de domínios/IPs (proteção contra SSRF)?

### A02 — Configuração Incorreta de Segurança
**Ferramentas:** linter de config, `checkov`/`tfsec` para IaC, verificação de `.env`/arquivos de config commitados.
**Perguntas de revisão manual:**
- Modo debug, verbose ou stack trace detalhado ficou ativado por padrão?
- CORS foi configurado com `*` combinado com `credentials: true`?
- Permissões de bucket/storage/pasta ficaram mais permissivas que o necessário?

### A03 — Falhas na Cadeia de Suprimentos de Software
**Ferramentas:** `pnpm audit` / `cargo audit` / OWASP Dependency-Check / `trivy`, conforme o ecossistema.
**Perguntas de revisão manual:**
- A dependência nova é de um mantenedor confiável, com histórico e uso relevante (risco de typosquatting)?
- A versão está fixada (lockfile atualizado) ou aponta para `latest`/branch?
- Algum passo novo de CI/CD baixa e executa script externo sem verificação (`curl | bash`, action não fixada por hash)?

#### A03.1 — Licenciamento das dependências

A régua desta seção é o RNF de licenciamento definido em
`[[DOC_REQUISITOS_NF]]` — leia-o antes de avaliar; ele é que diz o que este
projeto permite. As listas abaixo são o padrão para projeto de código fechado
e valem apenas na ausência de RNF específico.

Licença incompatível é risco jurídico, e diferente de defeito técnico ele não é
corrigível depois: descobrir na véspera do lançamento que uma dependência
central é GPL significa reescrever ou não lançar.

Rode sempre que a fase alterar dependências — `Cargo.toml`, `package.json` ou
qualquer lockfile:

| Ecossistema | Comando |
|---|---|
| Rust | `cargo deny check licenses` |
| Node | `pnpm dlx license-checker-rseidelsohn --summary --excludePrivatePackages` |
| Python | `pip-licenses --format=markdown` |
| Go | `go-licenses report ./...` |
| Ativos embarcados | Conferência manual contra `[[DOC_ATRIBUICOES]]` |

Critério (**BLOCKER**):

```text
DEPENDENCIAS_COM_LICENCA_PROIBIDA = 0
DEPENDENCIAS_SEM_LICENCA_DECLARADA = 0
```

Permitidas: MIT, Apache-2.0, BSD-2/3-Clause, ISC, Zlib, Unlicense, CC0-1.0.
Proibidas: GPL, AGPL, LGPL com ligação estática, SSPL, BUSL, Commons Clause,
"somente uso não comercial" e ausência de licença declarada.

**Este achado não pode ser aceito como WARNING justificado.** Nenhuma
justificativa técnica supera restrição de licença — a decisão é do dono do
produto, não do gate.

**Perguntas de revisão manual:**
- A verificação cobriu dependências **transitivas**, ou só as diretas? O
  problema quase sempre está na transitiva.
- Alguma dependência traz licença dupla ou cláusula extra sobre licença
  permissiva (ex.: Commons Clause sobre MIT)? A cláusula extra é o que vale.
- Entrou fonte tipográfica, ícone, imagem ou conjunto de dados? Seguem a mesma
  regra e costumam escapar do scanner, que só olha pacotes.
- `[[DOC_ATRIBUICOES]]` foi atualizado com as dependências novas?

Critério completo no RNF de licenciamento em `[[DOC_REQUISITOS_NF]]`.

### A04 — Falhas Criptográficas
**Ferramentas:** `semgrep` (regras de crypto), `bandit` (Python), `gitleaks`/`trufflehog` para chaves embutidas.
**Perguntas de revisão manual:**
- Senha nova é armazenada com hash adequado (bcrypt/argon2/scrypt), nunca MD5/SHA1/reversível?
- Existe chave, IV ou secret hardcoded no código-fonte?
- Token/sessão/nonce são gerados com gerador criptograficamente seguro (não `Math.random`/`random` padrão)?
- Dado sensível novo trafega ou é persistido sem criptografia quando deveria tê-la?

### A05 — Injeção
**Ferramentas:** `semgrep`, `bandit`, `eslint-plugin-security`, equivalente do ecossistema.
**Perguntas de revisão manual** *(equivalente a WSTG 4.7 — Input Validation Testing)*:
- Toda query nova usa parametrização/ORM, nunca concatenação de string com input do usuário?
- Algum `eval`, `exec`, `os.system`, `subprocess` com `shell=True` recebe dado de fora sem sanitização?
- Templates renderizados com dado de usuário permitem Server-Side Template Injection?

A validação sistemática de cada fronteira de entrada (tipo, formato, tamanho, allowlist, normalização, vetores por tecnologia) é avaliada com entrada hostil executada por `[[GATE_INPUT]]`, que roda antes deste gate.

### A06 — Design Inseguro
**Sem ferramenta automatizada confiável — revisão manual obrigatória**, inspirada no método de *use/misuse case* do WSTG.
**Perguntas de revisão manual:**
- Para o fluxo novo (pagamento, permissão, cadastro, etc.), qual seria o "caso de mau uso" mais óbvio, e o código o impede?
- Existe rate limiting/throttling em ações repetíveis e sensíveis (login, envio de OTP, reset de senha)?
- O fluxo multi-etapas pode ser pulado (ex.: chamar a etapa 2 sem ter passado pela etapa 1)?

### A07 — Falhas de Autenticação
**Ferramentas:** grep por padrões de comparação de senha, revisão manual do fluxo.
**Perguntas de revisão manual** *(equivalente a WSTG 4.3/4.4 — Identity & Authentication Testing)*:
- Existe bloqueio/lockout após tentativas falhas em endpoint de login novo ou alterado?
- Token de reset de senha tem expiração e uso único?
- Mensagens de erro de login diferenciam "usuário não existe" de "senha errada" (permitindo enumeração de contas)?

### A08 — Falhas de Integridade de Software ou Dados
**Ferramentas:** `bandit` (uso de `pickle`), `semgrep` (regras de deserialização).
**Perguntas de revisão manual:**
- Algum dado externo é desserializado com formato inseguro (pickle, `yaml.load` sem `SafeLoader`, `unserialize` do PHP)?
- Artefato de build/pipeline novo é validado por assinatura/checksum antes de ser usado?

### A09 — Falhas de Log e Alerta de Segurança
**Ferramentas:** grep por chamadas de log próximas a variáveis de senha/token/cartão.
**Perguntas de revisão manual:**
- Ações sensíveis novas (login, mudança de permissão, ação admin) geram log de auditoria?
- Algum log novo imprime senha, token completo, PII ou dado de cartão em texto claro?

### A10 — Tratamento Inadequado de Condições Excepcionais
**Ferramentas:** grep por `catch`/`except` vazios ou que apenas repassam a exceção ao cliente.
**Perguntas de revisão manual** *(equivalente a WSTG 4.8 — Testing for Error Handling)*:
- A mensagem de erro exposta ao cliente é genérica, e o detalhe técnico vai só para log interno seguro?
- Em caso de exceção, o controle de segurança falha fechado (nega acesso) e não aberto (permite por padrão)?
- Transações que falham fazem rollback limpo, sem deixar estado parcial/inconsistente?

---

## 4. SEGREDOS E CREDENCIAIS (verificação cross-cutting)

Execute scanner de segredos (`gitleaks`, `trufflehog` ou equivalente) sobre o diff completo, não só sobre os arquivos "óbvios".

Critério (BLOCKER):
```
NEW_HARDCODED_SECRETS = 0
```

Isso inclui: chaves de API, tokens, senhas, strings de conexão com credencial embutida, chaves privadas — mesmo em arquivos de teste, exemplo ou comentário.

---

## 5. PROIBIÇÕES

Durante a correção dos achados deste gate é proibido:

- Remover ou enfraquecer teste de segurança para o gate passar;
- Rebaixar a severidade de um achado sem justificativa técnica explícita e registrada;
- Adicionar supressão de regra de SAST/linter de segurança sem justificativa;
- "Resolver" um achado apenas com comentário/TODO prometendo corrigir depois;
- Aceitar vulnerabilidade de dependência crítica/alta como WARNING;
- Silenciar exceção de segurança (catch vazio, erro convertido em sucesso) para destravar o fluxo;
- Adicionar controle compensatório apenas em texto/documentação sem existir de fato no código.

Qualquer violação destas proibições é BLOCKER e deve ser revertida.

---

## 6. RE-EXECUÇÃO E LIMITE DE ITERAÇÕES

Após qualquer correção, execute novamente todo o conjunto de verificações afetado. Nunca considere uma correção válida por inspeção visual apenas.

```
MAX_GATE_ITERATIONS = 3
```

Uma iteração = ciclo completo de (verificar → corrigir → re-verificar).

Se após 3 iterações ainda existirem BLOCKERs:
1. PARE de corrigir;
2. emita STATUS = FAIL;
3. liste os BLOCKERs remanescentes com diagnóstico detalhado (categoria OWASP, arquivo, linha, cenário de exploração);
4. explique o que foi tentado em cada iteração e por que não resolveu;
5. recomende o próximo passo (ex.: redesenho do fluxo, revisão humana, quebra da task).

---

## 7. RESULTADO FINAL

Ao finalizar, produza obrigatoriamente:

```
SECURITY QUALITY GATE
STATUS: PASS | FAIL
ITERAÇÕES EXECUTADAS: N / 3
```

### ESCOPO DA REVISÃO
Categorias aplicáveis a este diff (com base na tabela da seção 1) e categorias marcadas N/A.

### EVIDÊNCIAS
Cada comando/ferramenta executado, com resumo da saída relevante (exit code, contagem de achados por severidade). Achado sem evidência de execução = NOT_MEASURED.

### A01 — Controle de Acesso Roto
### A02 — Configuração Incorreta de Segurança
### A03 — Falhas na Cadeia de Suprimentos
### A04 — Falhas Criptográficas
### A05 — Injeção
### A06 — Design Inseguro
### A07 — Falhas de Autenticação
### A08 — Falhas de Integridade de Software/Dados
### A09 — Falhas de Log e Alerta
### A10 — Tratamento Inadequado de Exceções

*(para cada uma: achados, severidade, evidência, ou "N/A — fora do escopo")*

### SEGREDOS E CREDENCIAIS

### BLOCKERS
Lista com status de cada um (resolvido | remanescente).

### WARNINGS ACEITOS
Cada um com justificativa explícita, re-argumentada do zero (justificativas da implementação não valem).

### RISCO RESIDUAL
Descreva qualquer risco ou ponto que não pôde ser validado automaticamente e por quê.

### VEREDITO

A fase somente pode ser marcada como concluída quando:

```
STATUS = PASS
```

`STATUS = PASS` exige: zero BLOCKERs remanescentes E todos os WARNINGs aceitos com justificativa registrada.

Nunca invente uma métrica ou achado de segurança. Quando uma verificação não puder ser medida com as ferramentas disponíveis, tente instalar uma ferramenta leve equivalente antes de declarar `NOT_MEASURED` — e nesse caso, sinalize explicitamente que a categoria correspondente ficou sem cobertura automatizada, para que a revisão manual receba atenção redobrada.