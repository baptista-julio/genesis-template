# Arquitetura

Decide a forma do sistema por entrevista, registra a decisão como ADR e produz
`[[DOC_ARQUITETURA]]`.

Consome `[[DOC_VISAO]]`, `[[DOC_REQUISITOS_NF]]` e o perfil do `MANIFEST.md`.
Sem requisitos não funcionais em mãos, **pare e execute `[[REQUISITOS]]`
antes** — arquitetura decidida sem restrição conhecida é preferência estética
com aparência de engenharia.

---

## Postura

A entrevista abaixo tem um viés declarado: **começa no monólito modular e exige
que a distribuição seja justificada**. Isso não é conservadorismo. Distribuir é
trocar um problema difícil (organizar código) por vários problemas mais
difíceis (rede não confiável, consistência eventual, depuração distribuída,
implantação coordenada). A troca compensa às vezes; quase nunca no começo.

Sistema distribuído prematuro é a forma mais cara de errar, porque é a mais
difícil de desfazer: quebrar um monólito depois é refatoração; juntar
microsserviços depois é reescrita.

A recomendação padrão é monólito modular. Ela muda com evidência, não com
ambição.

---

## Parte 1 — Entrevista

Uma pergunta por vez. Registre cada resposta: elas viram o contexto do ADR, e
é isso que permite reavaliar a decisão daqui a dois anos sabendo o que era
verdade quando ela foi tomada.

### 1. Equipe

> Quantas pessoas escrevem código neste sistema hoje, e daqui a um ano?
> Elas trabalham em times separados com prioridades próprias?

A razão original dos microsserviços é organizacional, não técnica: permitir que
times implantem sem coordenar. **Time único não colhe esse benefício e paga
todo o custo.**

| Resposta | Peso |
| --- | --- |
| 1 a 3 pessoas | Monólito, forte |
| 4 a 10, um time | Monólito modular, forte |
| Múltiplos times, backlog próprio | Abre a discussão |
| Times em fusos ou empresas diferentes | Empurra para distribuído |

### 2. Domínio

> Você consegue nomear as três a seis áreas principais do sistema, e dizer
> quais dados pertencem a cada uma?

Fronteira de módulo é a mesma coisa, esteja o módulo numa pasta ou num
servidor. Quem não sabe desenhar a fronteira dentro do processo vai desenhá-la
errado através da rede — e lá o erro custa muito mais para corrigir.

Se a resposta for vaga: **monólito modular, sem discussão.** Distribuir sem
fronteira clara produz o monólito distribuído, que tem o acoplamento de um e a
latência do outro.

### 3. Acoplamento

> Uma operação típica do usuário atravessa quantas dessas áreas?
> Ela precisa que todas tenham sucesso junto, ou parte pode falhar e ser
> reprocessada depois?

Fluxo que exige transação atômica entre áreas é o teste mais duro. Distribuído,
vira saga com compensação — muito código, muito caso de borda, muita chance de
estado inconsistente.

| Resposta | Peso |
| --- | --- |
| Consistência forte entre áreas | Monólito, forte |
| Tolera consistência eventual | Abre a discussão |
| Áreas quase independentes | Permite distribuir |

### 4. Escala

> Alguma parte do sistema tem necessidade de recursos muito diferente do
> resto? Uma área que precisa de GPU, ou processa mil vezes mais que as outras?

É o argumento técnico legítimo para distribuir: escalar só o que precisa. Vale
quando a assimetria é **grande e conhecida**, não hipotética.

Assimetria real com o resto simples costuma pedir **híbrido**: monólito mais um
serviço isolado para a parte pesada. Melhor que fatiar tudo.

### 5. Implantação

> Como o software chega ao usuário? Quem opera? Existe equipe de
> infraestrutura, ou quem escreve o código também cuida da produção?

Microsserviços exigem orquestração, descoberta de serviço, rastreamento
distribuído, log centralizado, pipeline por serviço. Sem quem opere isso, a
complexidade não some — vira indisponibilidade.

Desktop, CLI ou biblioteca: a pergunta não se aplica, e distribuição
normalmente não faz sentido. Registre e siga.

### 6. Restrições

> Existe imposição externa? Conformidade que obrigue isolar dados, sistema
> legado que dite integração, contrato de nuvem, exigência de cliente?

Restrição externa vence preferência técnica. Registre-a como tal no ADR: quem
ler depois precisa saber que a decisão foi imposta, não escolhida.

### 7. Reversibilidade

> Se em um ano a escolha se mostrar errada, quanto custa mudar?

Pergunta de fechamento. Entre duas opções de mérito parecido, **escolha a mais
fácil de desfazer**. Monólito modular com fronteira limpa é a opção mais
reversível que existe: cada módulo bem isolado já é um serviço em potencial.

---

## Parte 2 — Recomendação

Pese as respostas e recomende **uma** opção, com o trade-off explícito.

### Monólito modular

Padrão. Um artefato implantável, fronteiras internas explícitas e impostas por
ferramenta.

Indicado quando: time único ou poucos times; domínio ainda em descoberta;
consistência forte entre áreas; operação enxuta.

Exige, sob pena de virar monólito comum:
- cada módulo com interface pública explícita e o resto privado;
- dependência entre módulos verificada automaticamente (é um item do `[[GATE_CODE]]`);
- dados de um módulo acessados só pela interface dele, nunca por consulta direta à tabela alheia.

Custo: escala como um bloco; um módulo instável afeta o processo inteiro.

### Microsserviços

Serviços independentes, cada um com dados e ciclo próprios.

Indicado quando: múltiplos times autônomos; fronteiras comprovadas pelo uso;
assimetria real de escala; operação madura.

Custo: rede falha; consistência eventual em quase tudo; depurar exige
rastreamento distribuído; implantação vira coordenação; ambiente local fica
caro.

**Nunca recomende para projeto em estágio `bootstrap`.** Não existe informação
suficiente para desenhar as fronteiras, e fronteira errada em rede é a pior
dívida possível.

### Híbrido

Monólito modular mais um ou dois serviços isolados para o que realmente exige
isolamento — processamento pesado, componente de terceiros, parte com
conformidade própria.

Costuma ser a resposta certa quando a entrevista indica "quase monólito, exceto
por uma coisa". Não é meio-termo por indecisão; é a forma que corresponde ao
problema.

### Outras formas

Considere quando o perfil pedir, sem forçar:

| Forma | Quando |
| --- | --- |
| Camadas | Sistema pequeno, domínio simples, equipe iniciante |
| Hexagonal / Portas e adaptadores | Domínio rico, muitas integrações externas, teste sem infraestrutura |
| Orientada a eventos | Fluxos assíncronos, integração fraca, auditoria por histórico |
| Pipeline | Transformação de dados em etapas |
| Cliente pesado + backend fino | Desktop, aplicação local, uso offline |
| Serverless | Carga intermitente, operação mínima, aceita aprisionamento |

Apresente assim:

```text
RECOMENDAÇÃO: monólito modular

POR QUÊ
  Equipe de 2 (P1) — benefício organizacional de distribuir é zero
  Fronteiras ainda em descoberta (P2) — dividir agora chuta errado
  Fluxo principal exige consistência forte (P3) — saga seria caro

O QUE ISSO CUSTA
  Escala como bloco único
  Exige disciplina de fronteira; sem verificação automática, degrada

QUANDO REAVALIAR
  Terceiro time entrar, ou uma área precisar escalar sozinha

ALTERNATIVA SÉRIA
  Híbrido, se o processamento de imagens crescer como você descreveu
```

**Pare e espere a decisão do usuário.** Arquitetura é decisão de dono. Você
recomenda; ele decide.

---

## Parte 3 — Registro

Decidido, gere o ADR fundador via `[[ADR]]`. Normalmente `ADR-0001`.

O ADR guarda a entrevista: as respostas são o contexto, e o contexto é o que
permite reavaliar honestamente depois. ADR sem contexto vira dogma — ninguém
lembra por que a regra existe, e ninguém ousa mudá-la.

---

## Parte 4 — Documento

Escreva `[[DOC_ARQUITETURA]]`. Cabeçalho de metadados conforme
`[[DOCUMENTACAO]]`.

```markdown
# Arquitetura

## Visão geral
O que o sistema é, em um parágrafo, para quem chega hoje.

## Estilo
A forma escolhida e o link para o ADR. Não repita o raciocínio —
ele vive no ADR.

## Diagrama de contexto
Mermaid. O sistema, seus usuários e os sistemas externos.

## Módulos
Tabela: módulo | responsabilidade | dados que possui | depende de.
A coluna "dados que possui" é a que evita o acoplamento pela base
de dados — o vazamento mais comum e o mais difícil de reverter.

## Dependências entre módulos
Mermaid. O grafo precisa ser acíclico; ciclo aqui é BLOCKER no
[[GATE_CODE]].

## Fluxos principais
Diagrama de sequência para os dois ou três fluxos que mais importam.

## Dados
Onde o estado vive. Modelo, armazenamento, quem escreve, quem lê.

## Fronteiras
As regras que a implementação deve respeitar, redigidas de forma
verificável:
- domínio não importa infraestrutura
- módulo acessa outro só pela interface pública
- nenhum módulo lê tabela de outro
Cada regra diz como é verificada. Regra sem verificação é decoração.

## Implantação
Como vai para produção. Ambientes, artefatos, configuração.
Omita se o perfil não tiver implantação.

## Atributos de qualidade
Como a arquitetura atende cada RNF relevante de [[DOC_REQUISITOS_NF]].
Tabela: requisito | como a arquitetura atende | risco.

## Decisões
Índice dos ADRs que afetam arquitetura.

## Evolução
O que se sabe hoje que vai precisar mudar, e o gatilho de cada
mudança. É a seção que envelhece melhor.
```

Nunca escreva seção vazia. Seção sem conteúdo é removida, não preenchida com
"a definir".

---

## Saída

```text
ARQUITETURA
Estilo: <escolhido>
ADR: <id e caminho>
[[DOC_ARQUITETURA]]: criado | atualizado
Módulos: N
Fronteiras declaradas: N (verificáveis automaticamente: N)
Gatilhos de reavaliação: lista
```

Fronteira declarada mas não verificável é a que mais degrada — reporte
explicitamente quantas são, para que a equipe saiba do que depende da
disciplina humana.
