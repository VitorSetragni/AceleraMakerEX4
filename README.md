# aceleraMakerEX4
# Projeto 4 - Conta Bancaria em COBOL

Projeto desenvolvido em COBOL para ambiente Mainframe TK5/MVS 3.8j. O objetivo e processar dois arquivos de contas bancarias, ordenar os registros por agencia com o utilitario SORT, executar um programa COBOL e gerar um relatorio final com as contas validas, contas invalidas e resumo geral.

O projeto foi separado em arquivos de codigo, dados, copybook e JCL para ficar parecido com uma estrutura real de batch em mainframe.

## Estrutura do projeto

```text
Projeto_4_Conta_Bancaria/
├── COBOL/
│   ├── CONTABAN.cbl           # Programa principal usado no TK5
├── COPY/
│   └── REGCONTA.cpy           # Layout do registro de conta
├── DATA/
│   ├── CONTAS.txt             # Primeiro arquivo de entrada
│   └── NOVAS.txt              # Segundo arquivo de entrada
├── JCL/
│   ├── CRIA_DATASETS.jcl      # Cria os PDS, se ainda nao existirem
│   ├── INSTALA_FONTES_HERC01.jcl # Carrega fontes e dados via card reader
│   └── CONTABAN.jcl           # Compila, linka, ordena e executa
├──  RELATORIO/
│    └── P4RELAT_EXEMPLO.txt    # Exemplo do relatorio
└── Prints      #imagens dos prints da compilação
```

## Logica do programa

O JCL concatena os arquivos `CONTAS` e `NOVAS`, ordena todos os registros pelo campo de agencia e chama o programa `CONTABAN`. O programa le cada registro, identifica o tipo da conta, valida se o tipo e aceito e grava uma linha no relatorio.

Tipos aceitos:

| Codigo | Descricao |
|---|---|
| C | Conta corrente |
| P | Conta poupanca |

Qualquer outro tipo, como `X` ou `Z`, e marcado como invalido. As contas invalidas aparecem no relatorio, mas nao entram no saldo total geral.

## Layout do registro de entrada

Cada linha dos arquivos de dados possui 54 caracteres.

| Campo | Posicao | Tamanho | Descricao |
|---|---:|---:|---|
| NUM-CONTA | 1 a 8 | 8 | Numero da conta |
| NOME-CLIENTE | 9 a 38 | 30 | Nome do cliente |
| AGENCIA | 39 a 42 | 4 | Agencia bancaria |
| TIPO-CONTA | 43 | 1 | Tipo da conta: C, P ou invalido |
| SALDO | 44 a 54 | 11 | Saldo com 9 inteiros e 2 decimais implicitos |

Exemplo:

```text
00010021NEREU BARCELOS                1002C00000245030
```

Nesse exemplo:

- Conta: `00010021`
- Cliente: `NEREU BARCELOS`
- Agencia: `1002`
- Tipo: `C`
- Saldo: `2450,30`

## Datasets usados no TK5

```text
HERC01.P4COBOL(CONTABAN)   -> programa COBOL
HERC01.P4COBOL(REGCONTA)   -> copybook/layout
HERC01.P4DADOS(CONTAS)     -> primeiro arquivo de dados
HERC01.P4DADOS(NOVAS)      -> segundo arquivo de dados
HERC01.P4LOAD(CONTABAN)    -> load module gerado pelo link-editor
HERC01.P4RELAT             -> relatorio gerado pela execucao
```

## Como executar no TK5

### 1. Criar os datasets

Se os datasets ainda nao existirem, use o JCL:

```text
JCL/CRIA_DATASETS.jcl
```

Ele cria:

```text
HERC01.P4COBOL
HERC01.P4DADOS
HERC01.P4LOAD
HERC01.P4JCL
```

Se os datasets ja existirem, nao rode esse JCL de novo.

### 2. Carregar fontes e dados pelo Mac

Com o TK5 ligado, no Terminal do Mac, entre na pasta do projeto em outro terminal e rode:

```bash
nc -w 3 localhost 3505 < JCL/INSTALA_FONTES_HERC01.jcl
```

isso ira carrega os membros:

```text
HERC01.P4COBOL(CONTABAN)
HERC01.P4COBOL(REGCONTA)
HERC01.P4DADOS(CONTAS)
HERC01.P4DADOS(NOVAS)
```

O arquivo `INSTALA_FONTES_HERC01.jcl` esta com `USER=HERC01` e `PASSWORD=HERC01`. 

### 3. Salvar o JCL principal no TSO

O conteudo de `JCL/CONTABAN.jcl` deve ficar no membro:

```text
HERC01.P4JCL(CONTABAN)
```

### 4. Verificar o resultado

Depois que o job terminar, abra em BROWSE:

```text
'HERC01.P4RELAT'
```

Esse dataset e criado automaticamente pelo passo `RUNCOB`. Nao precisa criar o `P4RELAT` manualmente.


## Observacao sobre a copybook

O arquivo `COPY/REGCONTA.cpy` contem o layout oficial do registro de conta. Para facilitar a compilacao no TK5 usado durante os testes, o programa principal `CONTABAN.cbl` tambem declara o layout diretamente no `FILE SECTION`.


## Evidencias esperadas

Estão na pasta Prints

## Autor

Vitor Setragni
