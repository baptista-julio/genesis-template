# Diretrizes

Produz `[[DOC_DIRETRIZES]]` — o arquivo de instruções que o modelo lê ao abrir
o projeto.

Este prompt tem duas partes. A **Base** é igual em todo projeto: é a forma de
trabalhar. As **Regras de projeto** derivam do perfil e dos artefatos, e mudam
caso a caso.

O arquivo resultante é lido a cada sessão. Isso impõe economia: instrução que
não muda comportamento ocupa contexto e dilui as que mudam.

---

## Parte 1 — Base

Copie para `[[DOC_DIRETRIZES]]`, ajustando idioma e preferências se o usuário
divergir.

### Idioma

- Código, comentários, documentação, mensagens de commit e nomes de teste em
  **português brasileiro**. Testes com nomes descritivos
  (`nulos_nao_alteram_o_tipo_inferido`).
- Nunca utilize emojis.

### Como responder

Resposta **breve, curta, clara, objetiva**. Texto longo quebra o raciocínio.

- Vá direto ao ponto. Sem preâmbulo, sem recapitular o que acabei de dizer.
- Prefira tabela ou lista curta a parágrafo corrido.
- Não liste alternativas que você não recomenda — dê a recomendação.
- Não repita no fim o que já foi dito no meio.
- Explicação técnica: primeiro a resposta, depois o porquê, se necessário.

Isto vale para o texto que eu leio, não para o rigor do trabalho: continue
verificando, testando e medindo antes de afirmar.

### Comentários no código

Diretos e objetivos. No máximo duas linhas curtas. Comentário explica **por
quê**, não o quê — o quê já está no código, e a narração do óbvio envelhece
mal.

### Commits

**Conventional Commits com escopo**, em português, no imperativo:

```text
feat(importacao): importa planilhas Excel
```

- Imperativo, não gerúndio: `importa`, nunca `implementando`.
- O título diz **o que o software passa a fazer** — não a etapa interna do
  roteiro. "Fase 2" não vai no título.
- Corpo: o que mudou por camada e as decisões não óbvias. ADRs relacionadas no
  fim. Objetivo e direto.

Prefixos: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.

### Versionador

- Trabalho vai em **ramo** (`feat/...`, `fix/...`), nunca direto na principal.
- **Nunca publicar** sem autorização explícita naquele momento. Autorização
  para registrar a alteração não inclui publicá-la.

### Testes

- Nunca, em hipótese alguma, exclua um teste existente. Se o teste existe, é
  por um bom motivo.
- Nunca enfraqueça asserção nem marque teste como ignorado para fazer algo
  passar.

### Gates

1. Ao final de cada fase, execute `[[GATE_CODE]]`. É essencial.
2. **Se a § 21.2 acusar divergência**, execute `[[DOCUMENTACAO]]` em modo
   incremental sobre os documentos apontados e re-execute a seção. Documentação
   desatualizada reprova a fase como qualquer outro BLOCKER.
3. Depois que o gate de código passar, execute `[[GATE_SECURITY]]`. Também é
   essencial.
4. **Depois dos gates, preencha `[[DOC_CHANGELOG]]`** com a entrada da sessão.
   O `PASS` é o que autoriza a entrada. Resumido e objetivo: o que o software
   passa a fazer, decisões, verificação, pendências.

### Leituras obrigatórias

- As trinta primeiras linhas de `[[DOC_CHANGELOG]]`.

### Onde procurar

*(se `[[DOC_MAPA]]` existir)*

Antes de varrer a base atrás de um arquivo, consulte `[[DOC_MAPA]]`: quem
importa quem, quais arquivos cada biblioteca externa afeta, e quais são os
pontos de maior alcance.

Confira o carimbo de validade no topo do mapa. Se o código mudou muito desde a
geração, ou se o que você procura não está lá, varre — o mapa pode estar
incompleto, e ausência nele não é prova de inexistência.

### Documentação

Alteração que muda comportamento, arquitetura, API, dados ou operação atualiza
a documentação correspondente **na mesma alteração**. Critério completo em
`[[DOCUMENTACAO]]`.

Isto é verificado, não confiado: `[[GATE_CODE]]` § 21.2 compara o diff contra
o campo `Implementação:` dos documentos e reprova a fase quando comportamento
novo ou contrato alterado ficou sem documentação.

---

## Parte 2 — Regras de projeto

Gere a partir do perfil e dos artefatos existentes. Inclua apenas o que se
aplica.

### Dependências

*(se `tem_dependencias: sim`)*

**Menos dependência é melhor.** Biblioteca abandonada trava o projeto: para de
receber correção de segurança, quebra na próxima versão da linguagem, e a
migração acontece no pior momento possível. A dependência que não existe é a
única que nunca é abandonada.

- **Prefira escrever a somar dependência** quando o que ela resolve for
  pequeno.
- Antes de propor uma, me diga: o que resolve, quando foi a última publicação,
  e o que acontece se for abandonada.
- **Antes de adicionar qualquer dependência, verifique a licença e me diga
  qual é.** Vale para direta, transitiva, de desenvolvimento, fonte
  tipográfica, ícone e conjunto de dados.
- Na dúvida, não adicione — pergunte.
- Critério completo no RNF de licenciamento em `[[DOC_REQUISITOS_NF]]`.
- Dependência aceita entra em `[[DOC_ATRIBUICOES]]` na mesma alteração, com
  versão, licença e origem.

Insira aqui as listas de permitidas e proibidas do RNF, resumidas — quem lê as
diretrizes não deve precisar abrir outro arquivo para saber se pode adicionar
um pacote.

### Arquitetura

*(se `[[DOC_ARQUITETURA]]` existir)*

Transcreva as **fronteiras** declaradas, em forma imperativa:

```text
- Domínio não importa infraestrutura.
- Módulo acessa outro apenas pela interface pública.
- Nenhum módulo lê tabela pertencente a outro.
```

Aponte para `[[DOC_ARQUITETURA]]` e para o ADR fundador. Não repita o
raciocínio — a regra aqui, a razão lá.

### Comandos

Os comandos reais deste projeto, verificados:

```text
build:     <comando>
teste:     <comando>
lint:      <comando>
tipos:     <comando>
cobertura: <comando>
```

**Verifique cada um antes de escrever.** Comando que não roda é a instrução que
mais custa: o modelo tenta, falha, e gasta a sessão descobrindo o comando
certo.

### Estrutura

Mapa de uma linha por diretório de primeiro nível. Só onde o nome não é
autoexplicativo.

### Convenções próprias

Padrão que o projeto segue e que não é óbvio pelo código: forma de tratar
erro, de nomear, de organizar teste, de lidar com configuração.

Regra que já é imposta por ferramenta **não entra aqui** — o formatador cuida
da formatação melhor que uma instrução em texto.

---

## Formato do arquivo

Uma página, no máximo duas. Seções curtas, listas, imperativo.

Ordene por frequência de uso: o que vale em toda alteração vem primeiro; o que
vale em caso raro, depois.

**O que não entra:**

- explicação do que o projeto é — isso é `[[DOC_VISAO]]`;
- razão de decisão arquitetural — isso é `[[DIR_ADR]]`;
- documentação de funcionamento — isso é `[[DIR_SPEC]]`;
- regra que a ferramenta já impõe;
- instrução genérica de boa prática que qualquer modelo já segue.

---

## Manutenção

Revise quando: comando mudar, fronteira arquitetural mudar, ou uma instrução
provar-se ignorada.

Instrução repetidamente ignorada normalmente está mal colocada — enterrada,
ambígua, ou contradizendo outra. Reescreva em vez de repetir mais alto.

---

## Saída

```text
DIRETRIZES
[[DOC_DIRETRIZES]]: criado | atualizado em <caminho>
Linhas: N
Comandos verificados: N de M   <- precisa ser N = M
Fronteiras transcritas: N
Seções omitidas: lista com motivo
```
