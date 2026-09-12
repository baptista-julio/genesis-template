# Humanizer

Reescreve texto para que não soe como saída de modelo de linguagem, preservando
o que ele afirma.

Ferramenta avulsa: não faz parte da sequência do `[[ORQUESTRADOR]]`. Chame
quando houver texto para revisar.

**Origem.** Os padrões estruturais desta ferramenta derivam do projeto
[humanizer](https://github.com/blader/humanizer) de blader, sob licença MIT.
Adaptado aqui para português brasileiro e para texto de interface. Se o projeto
distribuir este pacote, a atribuição entra em `[[DOC_ATRIBUICOES]]`.

---

## Modos

Determine qual antes de começar. Eles pedem coisas diferentes.

| Modo | Entrada | Regra dominante |
| --- | --- | --- |
| **Prosa** | Artigo, README, documentação, e-mail, post | Pode reestruturar. Preserve as afirmações |
| **Interface** | Rótulo, botão, erro, estado vazio, tooltip, placeholder | Não pode mudar o que a ação faz, nem estourar o espaço |

Texto misto — uma landing page tem prosa e microcopy — processa por bloco, não
de uma vez.

---

## Princípio

**Reescrever, não remendar.** Trocar uma palavra proibida por sinônimo mantém a
frase com o mesmo esqueleto, e o esqueleto é o que denuncia. Releia o que o
trecho afirma, e diga de novo do seu jeito.

**Preservar o conteúdo.** Toda afirmação sustentada do original sobrevive.
Humanizar não é resumir, suavizar nem cortar informação — se o resultado diz
menos, falhou.

**Não inventar.** Nada de fato novo, número novo, exemplo novo. Se o original
era vago, o resultado é vago em linguagem normal — não vago com aparência de
preciso.

---

## Fluxo

1. **Marcar.** Leia o texto inteiro e aponte os padrões, dos mais fortes para
   os mais fracos. Categoria A denuncia mais que categoria D.
2. **Reescrever.** Refaça os trechos marcados dizendo a mesma coisa de outro
   jeito. Não edite palavra a palavra.
3. **Conferir.** Leia em voz alta. Onde tropeçar, o trecho está errado. Varra
   de novo atrás de padrão sobrevivente — reescrita costuma introduzir outros.
4. **Fechar.** Confirme que toda afirmação do original continua lá.

---

## Categoria A — Encenar em vez de afirmar

Os mais fortes. Um só já entrega o texto.

**A1. Contraste "não é X, é Y".** Estrutura que dá peso sem acrescentar nada.
> Não é apenas uma ferramenta, é um jeito novo de trabalhar.
> → A ferramenta muda como a equipe organiza o trabalho.

Em pt-BR, aparece também como "mais que X, Y" e "X? Y."

**A2. Fecho de uma linha.** Frase curta e dramática que repete o que o
parágrafo acabou de dizer. Corte. Se o parágrafo precisava dela, ele não estava
pronto.

**A3. Máxima fabricada.** "No fundo", "a verdadeira questão é", "como se diz
por aí". Inventa sabedoria de origem nenhuma.

**A4. Aquecimento.** "Vamos explorar", "antes de mais nada", "neste artigo
veremos". Comece pela afirmação.

**A5. Objeção inventada.** Rebater o que ninguém levantou. "Você deve estar se
perguntando se isso é seguro." Corte, ou responda direto, sem encenar a
pergunta.

---

## Categoria B — Ritmo por fórmula

**B1. Tríade forçada.** Três itens onde há um ou dois, o terceiro só pelo
ritmo. "Rápido, confiável e eficiente." Mantenha os que dizem algo distinto.

**B2. Abertura repetida.** Vários períodos seguidos começando igual — em pt-BR,
"Isso", "Além disso", "Com isso", "Dessa forma". Varie, ou junte as frases.

**B3. Travessão universal.** Travessão como conector de tudo. Ponto, vírgula e
parênteses existem e dizem coisas diferentes.

**B4. Qualificadores empilhados.** "Pode, em certos casos, ser eventualmente
útil." Afirme, ou diga o que não se sabe.

**B5. Par hifenizado.** "Solução pronta-para-uso", "abordagem centrada-no-
usuário". Excesso de composto denuncia.

**B6. Voz passiva escondendo o agente.** "Foi decidido que", "erros foram
encontrados". Diga quem decidiu, quem encontrou.

---

## Categoria C — Inflação e autoridade

**C1. Vocabulário de modelo.** Em pt-BR, os mais frequentes:

> no mundo de hoje · cada vez mais · de forma eficiente · solução completa ·
> robusto · poderoso · revolucionário · transformar · impulsionar · alavancar ·
> otimizar · potencializar · experiência única · ecossistema · jornada ·
> mergulhar fundo · vale destacar · é importante ressaltar · em suma ·
> por fim, mas não menos importante · fundamental · crucial · essencial

Não é lista de proibição: "essencial" cabe onde algo é mesmo essencial. O
problema é a frequência e o uso decorativo.

**C2. Importância inflada.** Detalhe comum descrito como marco. "Um passo
fundamental na evolução do setor" para uma atualização de rotina.

**C3. Associação vaga.** Ligar duas coisas sem dizer a relação. "Relacionado a
questões de desempenho e escalabilidade." Diga qual é a relação, ou corte.

**C4. Gerúndio pendurado.** Fato seguido de oração que não acrescenta.
"A API retorna JSON, garantindo compatibilidade." Garante como? Se importa,
vira frase. Se não, sai.

**C5. Linguagem de venda.** "Incrível", "imperdível", "surpreendente",
"aconchegante", "vibrante". Fora de peça publicitária, é ruído.

**C6. Autoridade emprestada.** "Especialistas afirmam", "estudos mostram",
"é sabido que". Cite quem, ou assuma a afirmação como sua.

**C7. Fuga dos verbos simples.** Rodeio para não escrever "é", "tem", "faz".
"Apresenta como característica" → "tem".

---

## Categoria D — Formatação

**D1. Negrito decorativo.** Negrito em palavra aleatória no meio da frase, ou
lista onde todo item começa com termo em negrito seguido de dois-pontos. Use
para o que o olho precisa achar de volta.

**D2. Título enfeitado.** Emoji em cabeçalho, título em Maiúsculas De Cada
Palavra fora de norma do projeto.

**D3. Aspas curvas onde cabe aspas retas.** Em código, atributo HTML, exemplo
de terminal.

---

## Categoria E — Sobras

Descuido puro. Varra sempre.

**E1. Cortesia de chatbot.** "Claro!", "Ótima pergunta!", "Espero ter
ajudado!", "Fico à disposição."

**E2. Ressalva de conhecimento.** "Até minha última atualização", "não tenho
acesso a dados em tempo real".

**E3. Título repetido na primeira frase.** Cabeçalho "Instalação" seguido de
"Nesta seção veremos como instalar."

**E4. Resto de rascunho.** "Como mencionado anteriormente", "na versão
anterior deste texto", "conforme solicitado".

---

## Modo Interface

Microcopy tem tiques próprios, e a maioria da lista acima quase não aparece num
botão. Estes são os que importam.

### I1. Desculpa no lugar de instrução

> Ops! Algo deu errado. Tente novamente mais tarde.
> → Não foi possível salvar. Verifique a conexão e tente de novo.

Mensagem de erro tem uma função: dizer **o que houve** e **o que fazer**.
"Algo deu errado" não faz nem uma nem outra. É o pior tique de interface que
existe, e o mais comum.

### I2. Alegria deslocada

"Oba!", "Uhul!", "Pronto! 🎉" em fluxo neutro. Pessoa que acabou de perder
trabalho não quer confete.

### I3. Jargão de implementação vazando

"Erro 500", "token inválido", "falha ao fazer parse", "timeout na requisição",
"registro não encontrado no banco". Diga em termos do que a pessoa estava
fazendo.

Cuidado com o jargão travestido: "chave de ligação" não tem termo técnico
nenhum e é igualmente opaco. Este é o mesmo critério da seção 22.1 do
`[[GATE_CODE]]`.

### I4. Botão que não diz o que faz

"OK", "Enviar", "Continuar", "Confirmar" em diálogo de decisão. O rótulo
descreve a ação: "Excluir 3 arquivos", "Salvar e sair".

Isso importa mais em diálogo destrutivo, onde "OK" faz a pessoa confirmar o que
não entendeu.

### I5. Estado vazio que só constata

> Nenhum item encontrado.
> → Nenhuma fatura ainda. As faturas aparecem aqui depois da primeira venda.

Estado vazio é a primeira coisa que se vê. Diga o que vai aparecer ali, ou o
que fazer para preencher.

### I6. Voz passiva e impessoal

"O arquivo foi salvo" → "Arquivo salvo". "Não foi possível pelo sistema
processar" → "Não deu para processar".

### I7. Rodeio de cortesia

"Por favor, insira o seu endereço de e-mail" → "E-mail". Rótulo de campo não
pede licença.

### I8. Inglês desnecessário

"Upload", "deletar", "salvar as settings", "seu dashboard". Use quando o termo
em português não existe ou soa artificial; não por hábito.

### Restrições do modo Interface

1. **Não mude o que a ação faz.** "Arquivar" não vira "Excluir" porque soa
   melhor.
2. **Respeite o espaço.** Rótulo que dobra de tamanho quebra o layout. Se a
   versão boa não couber, diga isso em vez de entregar algo que estoura.
3. **Mantenha o mesmo termo para a mesma coisa.** Se a tela chama de "projeto",
   o erro não chama de "workspace". Consistência vence elegância.
4. **Preserve o que é técnico de propósito.** Código de erro que o suporte
   pede, nome de campo de API, atalho de teclado.
5. **Não remova aviso de consequência.** Ação destrutiva diz o que se perde.
   Suavizar isso é piorar a interface, não humanizá-la.

---

## Voz

Com amostra de escrita do usuário: extraia o padrão real — comprimento de
frase, densidade de vírgula, formalidade, uso de primeira pessoa — e escreva
naquilo. A amostra manda mais que qualquer regra acima.

Sem amostra, use a voz do tipo de texto:

| Texto | Voz |
| --- | --- |
| Documentação técnica | Neutra, direta, segunda pessoa quando instrui |
| Interface | Curta, concreta, sem adjetivo |
| Ensaio, post | Pode ter opinião e personalidade |
| Comunicado, release | Factual, sem entusiasmo fabricado |

Se o projeto tiver `[[DOC_DIRETRIZES]]` com regra de idioma ou tom, ela vence
esta tabela.

---

## Sobre detecção automática

Você pediu que o texto não pareça feito por IA. Duas coisas honestas sobre
isso:

**O que esta ferramenta faz** é remover os padrões que fazem um leitor humano
pensar "isto saiu de um modelo". Isso é verificável: você lê e julga.

**O que ela não promete** é passar em detector automático. Detectores de IA
erram nos dois sentidos com frequência alta — reprovam texto humano e aprovam
texto de modelo —, e nenhum expõe critério estável para se otimizar contra. Um
prompt que garantisse "passa no detector X" estaria prometendo o que não se
verifica.

Na prática, texto que não tem os padrões acima tende a ir bem nos detectores,
porque é neles que a maioria se apoia. Mas o alvo é o leitor.

Contexto onde há declaração de autoria — trabalho acadêmico, concurso, algo que
exija dizer quem escreveu — a questão deixa de ser de estilo, e nenhuma
reescrita resolve.

---

## Saída

Entregue o texto reescrito. Depois, o registro:

```text
HUMANIZER
Modo: prosa | interface
Padrões encontrados: A1, B2, C1 (7x), I3 ...
Trechos reescritos: N de M
Afirmações preservadas: sim | divergências
Não reescrito: trecho + motivo
```

**"Não reescrito"** é seção importante. Rótulo que não cabe menor, termo
técnico que precisa continuar técnico, trecho cuja reescrita mudaria o sentido
— diga em vez de forçar.

Se o texto de entrada já estiver limpo, diga isso. Reescrever texto bom para
justificar a execução é a pior saída possível.
