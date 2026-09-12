# Licença e Atribuições

Produz `[[DOC_LICENCA]]` e `[[DOC_ATRIBUICOES]]`.

Licença é decisão jurídica do dono do projeto. **Você recomenda; ele decide.**
Nunca escolha por ele, e nunca gere um arquivo de licença sem confirmação
explícita — licença errada publicada é difícil de retirar, porque quem já
recebeu o código recebeu sob os termos antigos.

---

## Parte 1 — Licença do projeto

Leia `distribuicao` no perfil do `MANIFEST.md`.

### `fechado` ou `interno`

Não há escolha de licença aberta a fazer. Produza um aviso de direitos
reservados:

```text
Copyright (c) 2026 <titular>

Todos os direitos reservados.

Este software e seu código-fonte são propriedade de <titular>.
É proibida a reprodução, distribuição, modificação, engenharia
reversa ou uso, total ou parcial, sem autorização expressa por
escrito do titular.
```

Confirme com o usuário: **nome do titular** (pessoa física ou jurídica) e
**ano**. Não invente nenhum dos dois.

Para produto interno de empresa, considere também um aviso de uso restrito no
README. O grosso do trabalho aqui é a Parte 2 — em projeto fechado, o risco
real está nas dependências, não na licença própria.

### `aberto`

Apresente as opções com o trade-off real e recomende com base no perfil:

| Licença | Escolha quando | Custo |
| --- | --- | --- |
| **MIT** | Quer adoção máxima, sem exigir nada em troca | Ninguém precisa devolver melhoria; uso comercial fechado é permitido |
| **Apache-2.0** | Quer o mesmo, mais proteção de patente | Texto longo; exige preservar aviso e declarar alterações |
| **BSD-3-Clause** | Equivalente ao MIT, com cláusula de não endosso | Praticamente igual ao MIT |
| **MPL-2.0** | Quer que melhorias **nos seus arquivos** voltem | Copyleft por arquivo; empresa aceita mais que GPL |
| **GPL-3.0** | Quer que qualquer derivado seja aberto | Bloqueia adoção comercial fechada. Decisão de projeto, não detalhe |
| **AGPL-3.0** | O mesmo, incluindo uso como serviço em rede | Mais restritivo que existe em uso comum; muitas empresas proíbem |

Recomendação padrão por tipo:

- Biblioteca que quer adoção: **MIT**, ou **Apache-2.0** se a área tiver risco
  de patente.
- Aplicação que não quer virar produto fechado de terceiro: **GPL-3.0**; se for
  serviço de rede, **AGPL-3.0**.
- Na dúvida, com o projeto no começo: **MIT**. É a mais fácil de relicenciar
  depois — o caminho do permissivo para o restritivo existe; o contrário exige
  autorização de todos os contribuidores.

**Pergunte antes de escrever o arquivo.** Depois de confirmado, use o texto
oficial da licença, integral e sem alteração. Preencha ano e titular.

### Contribuição de terceiros

Se o projeto aceitar contribuição externa, decida também: contribuição sob a
mesma licença por padrão, ou acordo formal de contribuidor. Registre em ADR via
`[[ADR]]` — é uma decisão cara de reverter.

---

## Parte 2 — Atribuições

*(só se `tem_dependencias: sim`)*

`[[DOC_ATRIBUICOES]]` cumpre as obrigações de aviso de MIT, BSD, Apache-2.0 e
OFL. **Fica na raiz do projeto**, não dentro de `docs/`: é documento legal, e
precisa acompanhar a distribuição.

```markdown
# Atribuições

Este produto inclui os componentes de terceiros abaixo.

## Bibliotecas

| Componente | Versão | Licença | Origem |
| --- | --- | --- | --- |
| serde | 1.0.203 | MIT OR Apache-2.0 (adotada: MIT) | https://github.com/serde-rs/serde |
| calamine | 0.24.0 | MIT | https://github.com/tafia/calamine |

## Fontes tipográficas

| Fonte | Versão | Licença | Origem |
| --- | --- | --- | --- |
| Inter | 4.0 | OFL-1.1 | https://rsms.me/inter/ |

## Ícones e imagens

| Ativo | Licença | Origem |
| --- | --- | --- |

## Conjuntos de dados

| Dado | Licença | Origem |
| --- | --- | --- |

## Textos de licença

<texto integral de cada licença que exige preservação de aviso>
```

Regras:

1. **Licença dupla exige escolha registrada.** `MIT OR Apache-2.0` significa
   que você escolhe uma. Diga qual, na tabela.
2. **Transitivas contam.** O problema quase sempre está nelas. Ferramenta de
   verificação percorre a árvore inteira, não só o que está no manifesto.
3. **Fonte, ícone, imagem e dados são o ponto cego.** Não aparecem em nenhum
   verificador de licença — são arquivos, não pacotes. A tabela é a única
   conferência que os cobre, e ela é feita à mão.
4. **Atualizado na mesma alteração** que adiciona a dependência. Reconstituir
   depois é arqueologia, e costuma sair errado.

### Verificação

| Ecossistema | Comando |
| --- | --- |
| Rust | `cargo deny check licenses` |
| Node | `pnpm dlx license-checker-rseidelsohn --summary --excludePrivatePackages` |
| Python | `pip-licenses --format=markdown` |
| Go | `go-licenses report ./...` |
| Ativos embarcados | Conferência manual contra este arquivo |

Isto roda no `[[GATE_SECURITY]]` § A03.1, a cada alteração de dependência.

---

## Parte 3 — Cabeçalhos

Decida se os arquivos-fonte levam cabeçalho de licença.

- Projeto **aberto sob Apache-2.0 ou GPL**: sim, é o esperado pela licença.
- Projeto **aberto sob MIT ou BSD**: opcional; o arquivo `LICENSE` na raiz
  basta.
- Projeto **fechado**: normalmente não. Se houver risco de vazamento de
  trecho, um cabeçalho curto de direitos reservados ajuda a rastrear.

Decidido, registre em `[[DIRETRIZES]]` para que valha nos arquivos novos.

---

## Limite

Isto não é aconselhamento jurídico. Você aplica critérios conhecidos de
compatibilidade; questões de titularidade, patente, contrato de trabalho,
código de empregador ou distribuição em jurisdição específica são para
advogado.

Quando a pergunta sair do que se resolve por tabela, diga isso em vez de
opinar. Falso conforto em questão de licença é o pior resultado possível deste
prompt.

---

## Saída

```text
LICENÇA
Projeto: <licença> — confirmada pelo usuário
[[DOC_LICENCA]]: criado em <caminho>
[[DOC_ATRIBUICOES]]: N componentes
Licenças encontradas: lista
Incompatíveis: lista        <- BLOCKER, decisão do dono
Sem licença declarada: lista <- BLOCKER
Ativos sem verificação automática: N (conferidos à mão)
Questões para advogado: lista
```
