# Discovery

Estabelece o que é este projeto, para quem, e sob que restrições. Executado por
`[[ORQUESTRADOR]]` antes de qualquer outro prompt: todos os demais leem o perfil
que este produz para decidir o que se aplica.

Produz: `[[DOC_VISAO]]` e a seção **Perfil do projeto** do `MANIFEST.md`.

---

## Regra de conduta

**Levante antes de perguntar.** Repositório responde sozinho linguagem, tipo,
dependências e estágio. Pergunta cuja resposta está no `package.json` queima a
paciência do usuário e a sua credibilidade.

**Uma pergunta por vez, com opções.** Pergunta aberta em bloco produz resposta
vaga. Ofereça alternativas concretas e uma recomendação quando houver base para
recomendar.

**Pare quando souber o bastante.** O objetivo é decidir o que gerar, não
produzir um documento de visão de consultoria. Cinco respostas boas bastam.

---

## Parte 1 — Inferência

Determine sozinho, e apresente como *constatação a confirmar*, não como
pergunta:

| Campo | Onde olhar |
| --- | --- |
| `nome` | manifesto do pacote, título do README, nome da pasta |
| `linguagens` | extensões predominantes, arquivos de manifesto |
| `tipo` | presença de `main`/`bin`, servidor HTTP, `index.html`, exportações de biblioteca |
| `estagio` | quantidade de código, existência de testes, histórico do versionador |
| `tem_api` | rotas, controladores, especificação OpenAPI |
| `tem_banco` | migrações, ORM, cliente de banco |
| `tem_interface` | componentes, templates, arquivos de estilo |
| `interface_web` | `index.html`, framework de front, servidor de ativos estáticos |
| `tem_dependencias` | lockfile não vazio |

Apresente assim:

```text
CONSTATEI
  nome: quantia
  linguagens: Rust, TypeScript
  tipo: desktop (Tauri)
  estagio: em-desenvolvimento
  tem_banco: sim (migrações em migrations/)
  tem_api: nao

Confirma? Corrija o que estiver errado.
```

Projeto vazio ou quase vazio: pule esta parte e diga que vai perguntar do zero.

---

## Parte 2 — Perguntas

Só o que não dá para inferir. Nesta ordem.

### 1. Problema

> Em uma frase, que problema este software resolve, e para quem?

Se a resposta descrever a solução em vez do problema ("é um app em React"),
pergunte de novo mirando o usuário final e a dor dele.

### 2. Distribuição

> Este código é aberto, fechado ou interno?

Determina `[[LICENCA]]` inteiro e a régua de licenças de dependência do
`[[GATE_SECURITY]]`. Não prossiga com isto ambíguo — é a resposta mais cara de
descobrir errada, e a única que não se corrige depois.

### 3. Escala e uso

> Quantas pessoas usam, e como? Uso local de uma pessoa, equipe interna,
> público na internet?

Calibra os requisitos não funcionais e o peso da seção de segurança. Ferramenta
local de uma pessoa e serviço público exposto não merecem o mesmo rigor.

### 4. Restrições inegociáveis

> Existe alguma restrição que não está em discussão? Prazo, plataforma
> obrigatória, conformidade legal, integração com sistema existente, orçamento
> de infraestrutura, tamanho da equipe.

É o que mais influencia `[[ARQUITETURA]]`. Equipe de uma pessoa é restrição
arquitetural real, não detalhe de gestão.

### 5. Descoberta pela busca

*(só quando `interface_web: sim`)*

> Estas páginas precisam ser encontradas por mecanismo de busca, ou tudo vive
> atrás de autenticação?

Preenche `indexavel` e define o alcance do `[[GATE_FRONTEND]]`: painel interno
não é avaliado por indexabilidade, mas continua sendo avaliado por desempenho,
responsividade e acessibilidade. Errar isto para o lado da indexação gera
trabalho inútil; errar para o outro lado deixa o produto invisível.

### 6. Estado e ambição

> O que já existe e funciona hoje, e onde isto deveria chegar em um ano?

Separa o que documentar como implementado do que registrar como proposto.

---

## Parte 3 — Perfil

Preencha a seção **Perfil do projeto** do `MANIFEST.md`. Campo sem resposta
confiável fica `indefinido`, nunca chutado — `[[ORQUESTRADOR]]` sabe lidar com
`indefinido`, e não sabe lidar com resposta inventada.

Ajuste também a tabela de **Artefatos** do `MANIFEST.md` aos caminhos reais.
Se o projeto já usa `documentacao/` em vez de `docs/`, a tabela muda; os
prompts não.

---

## Parte 4 — Visão

Escreva `[[DOC_VISAO]]`. Curto — uma página basta. Estrutura:

```markdown
# Visão

Cabeçalho de metadados conforme [[DOCUMENTACAO]].

## Problema
O que dói hoje, para quem, e o que as pessoas fazem na ausência
desta solução.

## Proposta
O que o sistema faz a respeito. Sem detalhe de implementação.

## Público
Quem usa. Se houver mais de um perfil, o que cada um faz.

## Escopo
### Dentro
### Fora
O "fora" vale mais que o "dentro": é o que impede o projeto de inchar,
e é a seção que as pessoas consultam quando alguém propõe algo novo.

## Restrições
Tabela: restrição | origem | efeito no projeto.

## Sucesso
Como se reconhece que deu certo. Observável, não aspiracional.
"Importa planilha de 50 mil linhas em menos de 10s", não "ser rápido".
```

Nunca escreva seção vazia. Seção sem conteúdo real é removida.

---

## Saída

```text
DISCOVERY
Perfil: preenchido | parcial
Campos indefinidos: lista
[[DOC_VISAO]]: criado em <caminho>
Decisões pendentes do usuário: lista
```

Campo indefinido não bloqueia o roteiro — bloqueia apenas as etapas que
dependem dele, e `[[ORQUESTRADOR]]` precisa saber quais são.
