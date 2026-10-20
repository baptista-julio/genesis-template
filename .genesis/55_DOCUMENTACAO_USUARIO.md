# Documentação do usuário

Cria e mantém `[[DIR_USUARIO]]`: a documentação de quem **usa** o produto, sem
a parte técnica, escrita para uma pessoa leiga. Consome `[[DOC_REQUISITOS_F]]`,
`[[DIR_SPEC]]` e `[[DOC_GLOSSARIO]]`.

`[[DOCUMENTACAO]]` documenta o sistema para quem o desenvolve. Este prompt parte
dessa documentação e escreve outra coisa, para outro público. Não é resumo: é
reescrita do que a pessoa **vê e faz**.

Aplica-se quando há requisito com comportamento que a pessoa percebe. Enquanto
não houver RF `Implementado` com efeito visível, nada é criado.

---

## Princípios

### O dev é a fonte, o usuário é derivado

A informação nasce em `[[DIR_SPEC]]` e em `[[DOC_REQUISITOS_F]]`. Este prompt
não cria comportamento novo nem corrige documentação técnica. Se a especificação
não diz o que a pessoa vê, o buraco está lá: registre em Pendente e devolva para
`[[DOCUMENTACAO]]`. Não preencha com suposição.

### Só o comportamento visível

Entra o que a pessoa percebe: tela, passo, botão, mensagem, regra de negócio
que afeta o resultado. Não entra o que ela nunca vê: arquitetura, armazenamento,
API, desempenho interno.

### Organizada por tarefa, não por função

Quem consulta quer saber "como importo uma planilha", não "o que faz o módulo de
importação". Um documento por tarefa que a pessoa quer cumprir.

### Fonte única, também aqui

Cada instrução existe em um documento. Outro que precise dela usa link. A regra
de `[[DOCUMENTACAO]]` vale igual: duas cópias divergem na primeira alteração.

---

## Quando roda

Ao final de uma **fase concluída**, com os gates em `PASS`. Nunca por commit:
a maior parte dos commits não muda nada para quem usa.

Gatilho, qualquer um:

| O que aconteceu | Ação |
| --- | --- |
| RF passou a `Implementado` | Escreve o documento da tarefa |
| Critério de aceitação de RF `Implementado` mudou | Revisa o documento da tarefa |
| Tela, fluxo, texto de botão ou mensagem de erro visível mudou | Revisa o documento afetado |
| Regra de negócio com efeito visível mudou | Revisa o documento afetado |
| RF passou a `Revogado` | Marca o documento `Obsoleto` |

Refatoração, otimização e mudança interna sem efeito visível **não geram
edição**. Documento que continua verdadeiro fica como está.

Em projeto `maduro` ou `legado`, a primeira execução cobre todo RF
`Implementado` com comportamento visível. As seguintes seguem a tabela acima.

### Posição no fluxo de fase

```text
[[GATE_CODE]]          acusa em § 21.2: documentação dev e/ou guia do usuário
    v
[[DOCUMENTACAO]]       incremental, se o achado for de documentação técnica
    v
DOCUMENTACAO_USUARIO   <- aqui: parte da documentação técnica já atualizada
    v
[[GATE_CODE]]          re-executa § 21.2, precisa ficar limpa
    v
[[DOC_CHANGELOG]]      entrada da equipe, autorizada pelo PASS
    v
[[DOC_NOVIDADES]]      entrada do usuário, escrita a partir dela (seção Novidades)
```

Vem depois da documentação técnica porque deriva dela. Vem antes da re-execução
do gate porque é o gate que confere se o guia existe e acompanha o requisito:
`[[GATE_CODE]]` § 21.2 trata RF `Implementado` com a coluna **Guia do usuário**
vazia como BLOCKER.

---

## Estrutura

Ajuste ao produto. Pasta vazia não é criada.

```text
docs/usuario/
  README.md          por onde começar, e o mapa dos guias
  <tarefa>.md        um por tarefa: "importar-planilha.md"
  problemas.md       mensagens que a pessoa vê e o que fazer
  novidades.md       o que mudou a cada entrega, em linguagem simples
```

O `README.md` é a porta de entrada: o que o produto faz em duas frases, o que
a pessoa consegue fazer, e a lista de guias. Entra também em `[[DOC_INDICE]]`.

---

## Cabeçalho de metadados

Todo documento começa com este bloco. Ele serve a quem mantém, não a quem lê.

```yaml
Status: Implementado
Versão: 1.2
Criado em: 2026-07-19
Atualizado em: 2026-09-12
```

Mesma regra de `[[DOCUMENTACAO]]` para `Status`, `Versão` e datas.

**Omita** `Implementação`, `Depende de`, `Usado por`, `Relacionados` e qualquer
identificador interno (`RF-NNN`, `ADR-NNNN`): apontam para a parte técnica, e
quem lê o guia não deve encontrá-los.

### A ponte com a documentação técnica

O vínculo fica **do lado técnico**, na coluna **Guia do usuário** da matriz de
rastreabilidade de `[[DOC_REQUISITOS_F]]`:

```text
| Requisito | Implementação | Teste | ADR | Guia do usuário |
| RF-001 | src/importacao/ | tests/importacao/ | ADR-0004 | usuario/importar-planilha.md |
| RF-002 | src/log/ | tests/log/ | — | n/a — sem efeito visível |
```

O sentido é sempre técnico → usuário. O guia não aponta de volta. Para saber
que requisito um guia explica, procure o caminho dele na matriz.

Um RF sem efeito visível recebe `n/a` e um motivo curto. Coluna vazia significa
que ninguém decidiu, e `[[GATE_CODE]]` § 21.2 acusa.

---

## Linguagem

Escreva para quem nunca viu o sistema por dentro.

| Regra | Em vez de | Escreva |
| --- | --- | --- |
| Use o rótulo exato da interface | "acione a função de exportação" | "clique em **Exportar**" |
| Sem tecnologia | "o servidor grava no banco" | "suas alterações ficam salvas" |
| Sem identificador interno | "conforme RF-004" | descreva o comportamento |
| Mensagem como aparece na tela | "erro E_VAL_12" | a frase que a pessoa lê |
| Verbo de ação, passo numerado | "o usuário deverá proceder" | "1. Abra o menu **Arquivo**" |
| Termo do negócio, o que o cliente usa | termo do código | termo de `[[DOC_GLOSSARIO]]` |

Sigla e termo técnico só entram quando aparecem na tela. Aí, explique na
primeira vez em que surgem.

Cada guia responde: **o que a pessoa quer fazer, o que precisa ter antes, os
passos, o que ela vê quando dá certo, e o que fazer quando não dá.**

Corpo do documento nunca tem link para fora de `[[DIR_USUARIO]]`.

---

## Passos

1. **Monte a lista de RF afetados** pela tabela de gatilho. Use o diff da fase
   e o status dos RF em `[[DOC_REQUISITOS_F]]`.

2. **Siga cada RF até a especificação.** A matriz de rastreabilidade leva do RF
   ao documento de `[[DIR_SPEC]]`. Leia dele só o que a pessoa vê: telas,
   passos, mensagens, regras visíveis.

3. **Decida se algo mudou para quem usa.** Se não, o RF fica sem edição e vai
   para "Sem necessidade de mudança" na saída, com o motivo. RF sem nenhum
   efeito visível recebe `n/a — <motivo>` na coluna **Guia do usuário**.

4. **Escreva ou edite o guia da tarefa.** Preserve o que continua válido; não
   reescreva o documento por causa de um parágrafo.

5. **Atualize `problemas.md`** com as mensagens novas ou alteradas, cada uma
   com a frase da tela e o que fazer.

6. **Atualize o cabeçalho.** `Atualizado em`, `Versão` e `Status`.

7. **Propague.**

   | Mudou | Verifique também |
   | --- | --- |
   | Guia criado ou removido | Coluna **Guia do usuário** da matriz em `[[DOC_REQUISITOS_F]]`, `README.md` de `[[DIR_USUARIO]]`, `[[DOC_INDICE]]` |
   | Mensagem de erro | `problemas.md` |
   | Nome de tela, botão ou termo | Todos os guias que usam o rótulo antigo |
   | Termo novo do negócio | `[[DOC_GLOSSARIO]]` |

8. **Revise a escrita.** Passe `[[HUMANIZER]]` sobre o texto corrido, com as
   mesmas exceções de `[[DOCUMENTACAO]]`, e depois aplique o teste do leigo.

9. **Valide os links** dos documentos tocados.

---

## Novidades

`[[DOC_NOVIDADES]]` é o changelog de quem usa o produto. `[[DOC_CHANGELOG]]`
registra o que mudou para a equipe, com requisito, decisão e verificação. Este
registra o que mudou **na vida de quem usa**, sem nada disso.

Escreve-se **depois** da entrada do changelog, que o `PASS` dos gates autoriza,
e parte dela. Uma entrada do changelog sem efeito visível não gera novidade:
correção interna, refatoração e mudança de infraestrutura ficam só na equipe.

```markdown
# Novidades

## Próxima atualização

### Novo
- **Importar planilhas.** Agora você pode trazer seus dados de um arquivo
  Excel (`.xlsx`). Veja [Importar planilha](importar-planilha.md).

### Melhorou
- A importação de arquivos grandes ficou mais rápida: 50 mil linhas em
  menos de 10 segundos.

### Corrigido
- Datas que apareciam com um dia de diferença agora aparecem certas.

### Mudou
- O botão **Enviar** agora se chama **Importar**.

### Saiu
- Arquivos `.xls` antigos não são mais aceitos.
  **O que fazer:** abra o arquivo no Excel e salve como `.xlsx`.
```

| Categoria | Quando |
| --- | --- |
| `Novo` | Algo que a pessoa não podia fazer antes |
| `Melhorou` | Já existia e ficou melhor de um jeito que ela percebe |
| `Corrigido` | Algo que dava errado e agora dá certo |
| `Mudou` | Algo que ela fazia de um jeito e agora faz de outro |
| `Saiu` | Algo que deixou de existir |

Categoria vazia não aparece. Regras:

1. **Diga o que a pessoa passa a poder fazer**, não o que a equipe fez. Número
   só quando a pessoa o sente ("em menos de 10 segundos").
2. **`Mudou` e `Saiu` dizem o que fazer.** Quem lê precisa saber o que tem de
   ajustar antes de usar de novo.
3. **Link para o guia**, sempre que houver um, relativo e dentro de
   `[[DIR_USUARIO]]`.
4. **Sem identificador interno**: nada de `RF-NNN`, `ADR-NNNN`, hash de commit,
   nome de tecnologia.
5. **Segurança** entra só quando a pessoa precisa agir (trocar senha, atualizar
   o aplicativo). Nunca descreva a falha corrigida: o texto vira roteiro para
   quem quer explorá-la.
6. **Mesma ordem e mesma data do changelog**: versão mais recente no topo,
   `AAAA-MM-DD`, `Próxima atualização` acima de tudo. Ao publicar, renomeie
   para a versão ou a data, como no changelog. Entrada publicada não se
   reescreve.
7. **Uma entrada por mudança percebida.** Três commits que entregam uma
   funcionalidade são uma linha.

O documento leva o cabeçalho de metadados da seção acima, e passa pelo teste
do leigo como qualquer guia.

---

## Teste do leigo

Antes de concluir cada documento:

- ☐ nenhuma palavra que só quem desenvolve conhece;
- ☐ nenhum nome de tecnologia, caminho, endpoint ou código interno;
- ☐ cada botão, menu e mensagem citado existe na interface com esse texto;
- ☐ os passos, seguidos na ordem, levam ao resultado descrito;
- ☐ há o que fazer quando algo dá errado;
- ☐ nenhum comportamento descrito que a especificação não confirme.

Item reprovado volta para o passo 4. Quando o item é a última linha, o problema
está na especificação: vai para Pendente.

---

## Saída

```text
DOCUMENTAÇÃO DO USUÁRIO
RF no gatilho: N
RF Implementado sem guia e sem n/a: lista   <- precisa ser vazia
Criados: lista        Atualizados: lista
Marcados Obsoleto: lista com motivo
Novidades adicionadas: N
Entradas do changelog sem efeito visível (sem novidade): N
Sem necessidade de mudança: lista com motivo
Links quebrados: N   <- precisa ser 0
Reprovados no teste do leigo: lista   <- precisa resolver
Pendente: o que a especificação não diz e por quê
```
