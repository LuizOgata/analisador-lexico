# ALUNOS:
Beatriz Zorato Mendonça RA: 2414754/
Lucas Perrone Borlin Alves RA: 2548107/
Luiz Filipe Ichiro Ogata RA: 2652281/
Yume Fernandes Lima RA: 2646927/



#  Analisador Léxico — Mini-Linguagem de Ingressos

Analisador léxico desenvolvido em **Python** para uma mini-linguagem inspirada em sistemas de venda de ingressos de eventos, como Sympla e Ticketmaster.

O projeto recebe comandos relacionados à compra de ingressos, identifica os tokens da linguagem, ignora espaços e comentários e informa erros léxicos com linha, coluna e dicas específicas do domínio.

---

##  Exemplo de entrada

```text
INGRESSO 2x "Festival de Inverno" SETOR pista LOTE 2 MEIA R$ 180,00 DATA 20/07/2026
```

A entrada acima é dividida em tokens como:

```text
INGRESSO
2
x
"Festival de Inverno"
SETOR
pista
LOTE
2
MEIA
R$ 180,00
DATA
20/07/2026
```

---

##  Tecnologias utilizadas

- **Python 3**
- **re** — expressões regulares
- **pandas** — exibição da tabela de tokens
- **ipywidgets** — interface gráfica interativa
- **Google Colab**

---

---

#  Tokens reconhecidos

O analisador possui mais de 12 tipos de tokens.

| Token | Descrição | Exemplo |
|---|---|---|
| `INGRESSO` | Palavra reservada que inicia um ingresso | `INGRESSO` |
| `SETOR` | Indica o setor do evento | `SETOR` |
| `LOTE` | Indica o lote do ingresso | `LOTE` |
| `MEIA` | Indica meia-entrada | `MEIA` |
| `DATA` | Palavra reservada para data | `DATA` |
| `VALOR` | Valor monetário em reais | `R$ 180,00` |
| `DATA_LITERAL` | Data no formato brasileiro | `20/07/2026` |
| `CODIGO_EVENTO` | Código especial do evento | `EVT-2026` |
| `MULTIPLICADOR` | Quantidade multiplicada | `x` |
| `STRING` | Nome do evento entre aspas | `"Festival de Inverno"` |
| `IDENTIFICADOR` | Nome de setor ou identificador | `pista` |
| `NUMERO` | Número inteiro | `2` |
| `SIMBOLO` | Símbolos aceitos pela linguagem | `,` |

---

#  Palavras reservadas

As palavras reservadas são reconhecidas sem diferenciação entre maiúsculas e minúsculas.

São utilizadas as expressões:

```regex
\bingresso\b
\bsetor\b
\blote\b
\bmeia\b
\bdata\b
```

Com a opção:

```python
re.IGNORECASE
```

Portanto, exemplos como:

```text
INGRESSO
ingresso
Ingresso
InGrEsSo
```

são reconhecidos como o mesmo token.

A utilização de `\b` garante que a palavra reservada seja identificada como uma palavra completa.

---

# Literais específicos

A linguagem possui literais definidos por expressões regulares.

## Valor

```regex
R\$\s*\d+(?:,\d{2})
```

Exemplo:

```text
R$ 180,00
```

---

## Data

```regex
\d{2}/\d{2}/\d{4}
```

Exemplo:

```text
20/07/2026
```

---

## Código de evento

```regex
EVT-\d{4,6}
```

Exemplo:

```text
EVT-2026
```

---

#  Prioridade dos tokens

A ordem das regras é importante porque alguns tokens podem entrar em conflito.

Tokens específicos são colocados antes dos tokens genéricos.

Por exemplo:

```text
20/07/2026
```

poderia ser interpretado incorretamente como:

```text
20
/
07
/
2026
```

ou como vários números.

Para evitar isso, `DATA_LITERAL` possui prioridade maior que `NUMERO`.

O mesmo acontece com:

```text
R$ 180,00
```

que deve ser reconhecido como um único token `VALOR`.

### Conflito documentado no código

No código existe um comentário explicando esse conflito:

```python
# CONFLITO:
# A entrada "20/07/2026" poderia inicialmente ser interpretada
# como vários números separados por "/".
#
# SOLUÇÃO:
# DATA_LITERAL aparece antes de NUMERO.
```

---

#  Comentários

Comentários começam com:

```text
#
```

e são ignorados pelo analisador.

Exemplo:

```text
# Compra para o evento
INGRESSO 2x "Festival de Inverno" SETOR pista
```

A linha iniciada por `#` não gera tokens.

---

# ␠ Espaços

Espaços, tabulações e quebras de linha são ignorados.

Por exemplo:

```text
INGRESSO    2x     "Festival"
```

é processado normalmente.

---

#  Erros léxicos

Quando um caractere não reconhecido é encontrado, o analisador informa:

- linha;
- coluna;
- caractere inválido;
- dica amigável relacionada a ingressos/eventos.

Exemplo:

```text
INGRESSO 2x "Festival @ Inverno"
```

Pode gerar:

```text
Linha 1, coluna XX:
Caractere inesperado '@'

 Dica:
Verifique o nome do evento, setor ou código do ingresso.
```

Outro exemplo:

```text
DATA 20/07/26
```

gera uma orientação relacionada ao formato correto:

```text
DD/MM/AAAA
```

---

#  Interface

A interface utiliza **ipywidgets** e possui:

1. Área para inserir a entrada;
2. Botão para executar a análise;
3. Resultado colorido;
4. Mensagens de erro;
5. Dicas específicas do domínio;
6. Tabela contendo os tokens reconhecidos.

A tabela apresenta:

| Token | Lexema | Linha | Coluna |
|---|---|---:|---:|
| `INGRESSO` | `INGRESSO` | 1 | 1 |
| `NUMERO` | `2` | 1 | 10 |
| `MULTIPLICADOR` | `x` | 1 | 11 |

---

#  Casos de teste

##  Teste válido 1

```text
INGRESSO 2x "Festival de Inverno" SETOR pista LOTE 2 MEIA R$ 180,00 DATA 20/07/2026
```

Resultado esperado:

```text
Entrada válida!
Nenhum erro léxico encontrado.
```

---

## Teste válido 2

```text
INGRESSO 1x "Show de Rock" SETOR premium LOTE 1 R$ 350,00 DATA 15/08/2026
```

---

## Teste válido 3

```text
# Compra para o evento
INGRESSO 3x "Festival de Verão" SETOR arquibancada LOTE 3 MEIA R$ 95,50 DATA 10/01/2027
```

O comentário é ignorado.

---

## Teste inválido 1

```text
INGRESSO 2x "Festival @ Inverno" SETOR pista R$ 180,00 DATA 20/07/2026
```

Problema:

```text
@
```

é um caractere não reconhecido pela linguagem.

---

## Teste inválido 2

```text
INGRESSO 2x "Festival de Inverno" SETOR pista DATA 20/07/26
```

Problema:

```text
20/07/26
```

não segue o formato esperado:

```text
DD/MM/AAAA
```

---

#  Diário de ambiguidade

Durante a construção do analisador foi identificado um conflito entre `DATA_LITERAL` e `NUMERO`. Uma data como `20/07/2026` poderia ser interpretada como três números separados por `/`. Para solucionar essa ambiguidade, `DATA_LITERAL` foi colocado antes de `NUMERO na ordem de reconhecimento. Dessa maneira, o analisador tenta reconhecer primeiro a data completa. Um conflito semelhante ocorre com valores monetários: `R$ 180,00` precisa ser reconhecido como um único `VALOR`, e não como uma combinação de identificadores, números e símbolos. Por isso, as expressões mais específicas recebem prioridade sobre as regras genéricas.

---




O projeto demonstra a implementação de um **analisador léxico** para uma linguagem específica de domínio (DSL), utilizando expressões regulares para reconhecer elementos relacionados à venda de ingressos de eventos.

O analisador representa a primeira etapa de um processo de compilação/interpretação: transformar uma sequência de caracteres em uma sequência estruturada de tokens que poderá ser utilizada posteriormente por um analisador sintático.

