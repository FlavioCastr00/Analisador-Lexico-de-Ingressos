# Analisador Léxico de Ingressos

Projeto de um analisador léxico desenvolvido em Python utilizando a biblioteca Lark.
O projeto simula uma mini-linguagem para representar informações de ingressos de
eventos, identificando palavras reservadas, quantidades, valores, datas, nomes de
eventos e setores.

Além da análise léxica, o projeto possui uma interface interativa desenvolvida com
ipywidgets, permitindo visualizar os tokens encontrados, suas posições e as
prioridades utilizadas pelo analisador.

## Grupo

Flávio Henrique - 2595528 \
Laryssa Dantas Vieira - 2342968 \
Gabriel Diogo - 2683210 \
Guilherme Pedigone - 2712633 \

## Tabela de Tokens

| Token        | Descrição                                | Regex                            | Exemplo                 | Prioridade |
| ------------ | ---------------------------------------- | -------------------------------- | ----------------------- | ---------- |
| `INGRESSO`   | Palavra reservada que inicia um ingresso | `\bINGRESSO\b`                   | `INGRESSO`              | 3 |
| `SETOR`      | Palavra reservada para o setor           | `\bSETOR\b`                      | `SETOR`                 | 3 |
| `LOTE`       | Palavra reservada para o lote            | `\bLOTE\b`                       | `LOTE`                  | 3 |
| `MEIA`       | Palavra reservada para meia-entrada      | `\bMEIA\b`                       | `MEIA`                  | 3 |
| `INTEIRA`    | Palavra reservada para ingresso inteiro  | `\bINTEIRA\b`                    | `INTEIRA`               | 3 |
| `DATA`       | Palavra reservada para data              | `\bDATA\b`                       | `DATA`                  | 3 |
| `QTD`        | Quantidade de ingressos                  | `\d+x\b`                         | `2x`                    | 4 |
| `NUMERO`     | Número inteiro                           | `\d+`                            | `2`                     | 2 |
| `VALOR`      | Valor monetário em reais                 | `R\$\s*\d+(?:\.\d{3})*,\d{2}`    | `R$ 180,00`             | 2 |
| `DATA_VAL`   | Data do evento                           | `\d{2}/\d{2}/\d{4}`              | `20/07/2026`             | 4 |
| `TEXTO`      | Nome do evento entre aspas               | `"[^"\n]+"`                      | `"Festival de Inverno"` | 2 |
| `SETOR_NOME` | Nome do setor                            | `[A-Za-zÀ-ÿ]+`                   | `pista`                 | 1 |
| `SIMBOLO_X`  | Unidade da quantidade                    | `x`                              | `x`                     | 1 |
| `COMENTARIO` | Comentário                               | `#[^\n]*`                        | `# evento especial`     | — |

## Diário de Ambiguidade

Durante o desenvolvimento do analisador léxico, identificamos um conflito entre as regras NUMERO e DATA_VAL. A regra NUMERO, definida pela expressão \d+, poderia reconhecer apenas o início de uma data, como 20 em 20/07/2026, enquanto DATA_VAL deveria reconhecer toda a sequência como uma única data. Para resolver esse conflito, definimos prioridades diferentes para as regras, atribuindo prioridade 2 a NUMERO e prioridade 4 a DATA_VAL. Dessa forma, quando o lexer encontra uma sequência que representa uma data, a regra mais específica possui prioridade e reconhece 20/07/2026 como um único token DATA_VAL. Esse conflito mostrou a importância de definir corretamente as prioridades das expressões regulares, principalmente quando uma regra mais genérica pode reconhecer parte de uma sequência que deveria ser tratada por uma regra mais específica.

## Célula 0: Preparação do Ambiente

A Célula 0 é responsável pela preparação do ambiente utilizado pelo projeto.

São instaladas as bibliotecas necessárias para o funcionamento do analisador e
da interface:

- `lark`: utilizada para construir o analisador léxico;
- `ipywidgets`: utilizada para criar os componentes interativos da interface.

Também são importadas funções e classes utilizadas nas células seguintes.

## Célula 1: Ferramentas de Visualização

Esta célula não é responsável pela análise léxica. Sua função é apresentar
visualmente os resultados produzidos pelo analisador.

Para isso, é criado o dicionário `CORES`, que relaciona cada tipo de token
a uma determinada cor.

Por exemplo:

```python
"INGRESSO": "#1565c0"
```

## Célula 2: Analisador Léxico

Nesta célula ocorre a análise léxica propriamente dita. Uma sequência de
caracteres é transformada em uma sequência de tokens utilizando expressões
regulares e a biblioteca Lark.

Por exemplo, ao fornecer:

```text
INGRESSO 2x "Festival de Inverno" SETOR pista LOTE 2 MEIA R$ 180,00 DATA 20/07/2026
```

## Célula 3: A Interface

Nesta célula é construída a interface de interação com o analisador léxico
utilizando `ipywidgets`.

A variável `exemplos_ingresso` é um dicionário que armazena diferentes
entradas que podem ser selecionadas pelo usuário.

Entre os exemplos estão entradas válidas, uma entrada contendo comentário
e uma entrada com uma data inválida.

A interface possui um menu suspenso (`Dropdown`) para seleção dos exemplos,
um campo de entrada para edição manual do texto, um botão para executar a
análise e um controle deslizante para alterar a prioridade da regra `DATA_VAL`.

### Laboratório de Prioridade

O controle `prioridade_data` permite alterar a prioridade da regra `DATA_VAL`
entre 1 e 5.

A gramática é reconstruída dinamicamente de acordo com o valor selecionado:

```python
gramatica = gramatica_ingresso.replace(
    "DATA_VAL.4",
    f"DATA_VAL.{prioridade_data.value}"
)
```