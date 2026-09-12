# Levantamento de Requisitos

Produz `[[DOC_REQUISITOS_F]]` e `[[DOC_REQUISITOS_NF]]`. Consome `[[DOC_VISAO]]`
e o perfil do `MANIFEST.md`.

Requisito existe para ser **verificável** e **referenciável**. Requisito que
não dá para testar é desejo; requisito sem identificador estável não consegue
ser citado por ADR, teste ou gate — e por isso não é cobrado por ninguém.

---

## Identificação

`RF-NNN` para funcional, `RNF-NNN` para não funcional. Três dígitos,
sequencial, **nunca reaproveitado**. Requisito removido vira `Revogado`, e o
número morre com ele.

O identificador é permanente porque é ele que aparece no teste, no ADR e no
commit. Renumerar quebra referência silenciosamente em todo lugar.

---

## Requisitos funcionais

O que o sistema faz. Um requisito por comportamento observável.

```markdown
### RF-001 — Importação de planilha

**Status:** Implementado
**Prioridade:** Essencial
**Origem:** [[DOC_VISAO]] § Escopo

O sistema importa arquivos `.xlsx` e `.csv`, mapeando colunas para campos
de destino.

**Critérios de aceitação**
- Arquivo com até 50.000 linhas importa sem erro
- Coluna ausente no arquivo é reportada antes de qualquer escrita
- Linha inválida não interrompe a importação; vai para o relatório de rejeição

**Verificação:** testes de integração em `tests/importacao/`
**Relacionados:** RF-002, RNF-003
```

Regras:

- **Verbo de comportamento, não de tela.** "O sistema calcula o saldo
  consolidado", não "a tela mostra o saldo". Tela muda, comportamento fica.
- **Critério de aceitação é o requisito.** A prosa descreve; o critério é o que
  se testa. Sem critério, o requisito não está pronto.
- **Um comportamento por requisito.** Se o texto tem "e também", são dois.
- **Sem solução embutida.** "Armazena o histórico" é requisito. "Armazena o
  histórico em Redis" é decisão de arquitetura, e pertence a um ADR.

Prioridades: `Essencial` (sem isso não há produto), `Importante` (produto é
pior sem, mas existe), `Desejável` (melhoria).

Se tudo virar Essencial, a priorização não foi feita. Force a distinção.

---

## Requisitos não funcionais

Como o sistema se comporta. É a categoria que mais se escreve de forma inútil,
porque aceita adjetivo no lugar de número.

**Todo RNF precisa de um valor medido ou mensurável.** "Deve ser rápido" não é
requisito. "Abre em menos de 2s em máquina de 8 GB" é.

```markdown
### RNF-003 — Tempo de resposta da importação

**Status:** Implementado
**Categoria:** Desempenho

Importação de 50.000 linhas completa em menos de 10 segundos em máquina
de referência (4 núcleos, 8 GB, disco SSD).

**Medição:** `cargo bench --bench importacao`
**Consequência de violar:** usuário abandona a operação; é o fluxo de
entrada principal do produto.
```

Categorias a percorrer — omita a que não se aplica ao perfil, não invente
requisito para preencher:

| Categoria | Pergunta que responde |
| --- | --- |
| Desempenho | Quão rápido, sob que carga, medido como? |
| Escala | Quantos usuários, registros, requisições simultâneas? |
| Disponibilidade | Quanto tempo fora do ar é tolerável? Tem janela de manutenção? |
| Segurança | Que dado é sensível? Quem pode ver o quê? |
| Privacidade | Coleta dado pessoal? Sob que base legal? Por quanto tempo? |
| Acessibilidade | Que nível? Teclado, leitor de tela, contraste? |
| Portabilidade | Que sistemas, navegadores, versões? |
| Manutenibilidade | Que cobertura, que complexidade máxima? Liga no `[[GATE_CODE]]` |
| Observabilidade | O que se registra em log, o que se mede, como se descobre falha? |
| Internacionalização | Quantos idiomas, quantos fusos, que formatos? |
| Licenciamento | Que licença de dependência entra? Liga no `[[GATE_SECURITY]]` |

---

## RNF de Licenciamento

Projeto com dependências precisa deste requisito, sempre. É a restrição que
menos se corrige depois: descobrir na véspera do lançamento que uma dependência
central é copyleft significa reescrever ou não lançar.

Gere-o a partir da resposta de `distribuicao` do perfil.

### Se `distribuicao: fechado` ou `interno`

```markdown
### RNF-00N — Licenciamento de dependências

**Status:** Implementado
**Categoria:** Licenciamento

Este é um produto de código fechado. Toda dependência — direta ou
transitiva, de produção ou de desenvolvimento distribuída junto ao
binário — deve ter licença que permita uso comercial sem obrigar a
abertura do código.

Restrição jurídica, não preferência técnica.

**Permitidas:** MIT · Apache-2.0 · BSD-2-Clause · BSD-3-Clause · ISC ·
Zlib · Unlicense · CC0-1.0 · OFL-1.1 (somente fonte tipográfica)

**Proibidas:** GPL (todas) · AGPL · LGPL com ligação estática · SSPL ·
BUSL · Commons Clause · qualquer licença "somente uso não comercial" ·
qualquer dependência sem licença declarada

Ausência de licença não é permissão: sem licença explícita, o padrão
legal é todos os direitos reservados.

**Casos que exigem decisão, não regra**

| Situação | Como tratar |
| --- | --- |
| LGPL com ligação dinâmica | Permitido tecnicamente, complica distribuição. Só com ADR |
| Licença dupla (MIT/Apache-2.0) | Permitida. Registrar qual o projeto adota |
| Permissiva com cláusula extra | Proibida. A cláusula extra é o que vale |
| Fonte, ícone, imagem, dataset | Mesma regra. Escapa do scanner — conferência manual |

**Verificação:** `[[GATE_SECURITY]]` § A03.1
**Registro:** dependência aceita entra em `[[DOC_ATRIBUICOES]]` na mesma
alteração, com versão, licença e origem.
```

### Se `distribuicao: aberto`

O critério inverte: a licença **do projeto** é que restringe o que pode entrar.
Registre a licença escolhida em `[[LICENCA]]` e derive a lista de compatíveis
dela — projeto MIT não absorve dependência GPL sem virar GPL.

---

## Dependência mínima

Inclua no RNF de licenciamento, ou como RNF próprio:

> Antes da licença vem uma pergunta mais básica: **essa dependência precisa
> existir?**
>
> Toda dependência é um compromisso com a manutenção de outra pessoa. Quando é
> abandonada, o projeto herda o problema: correção de segurança para de chegar,
> a próxima versão da linguagem quebra o pacote, e a migração acontece no
> momento em que menos se pode pagar por ela.
>
> Critérios, em ordem:
>
> 1. **Cabe em pouco código próprio?** Então escreva. Código próprio pequeno
>    custa menos que dependência pequena.
> 2. **É mantida?** Última publicação, ritmo de correção, número de
>    mantenedores. Mantenedor único é risco real.
> 3. **Quanto arrasta junto?** Uma dependência com trinta transitivas são
>    trinta compromissos, não um.
> 4. **Se for abandonada amanhã, qual é a saída?** Sem resposta, a dependência
>    é ponto único de falha.
>
> Isto não é reimplementar tudo. Escrever leitura de XLSX à mão seria pior por
> qualquer métrica. O critério é proporcionalidade: dependência grande
> resolvendo problema grande e bem delimitado é bom negócio; dependência
> pequena resolvendo o que caberia em vinte linhas não é.

---

## Rastreabilidade

Ao final, monte a matriz em `[[DOC_REQUISITOS_F]]`:

| Requisito | Implementação | Teste | ADR |
| --- | --- | --- | --- |
| RF-001 | `src/importacao/` | `tests/importacao/` | ADR-0004 |

Requisito `Implementado` sem teste é achado, não detalhe. Reporte.

---

## Estados

`Proposto` · `Aprovado` · `Em desenvolvimento` · `Implementado` · `Revogado`

`Revogado` mantém o texto original e ganha o motivo e a data. Nunca apague um
requisito: quem lê o histórico precisa entender por que algo deixou de valer.

---

## Saída

```text
REQUISITOS
Funcionais: N (essencial: N, importante: N, desejável: N)
Não funcionais: N em M categorias
Sem critério de aceitação: lista  <- precisa resolver
RNF sem valor mensurável: lista   <- precisa resolver
Implementados sem teste: lista
Categorias N/A: lista com motivo
```
