- [Comandos para integração com o Thera Lase Surgery](#comandos-para-integração-com-o-thera-lase-surgery)
  - [Comandos do gerenciador de atividades](#comandos-do-gerenciador-de-atividades)
    - [Comando: am\_ctx](#comando-am_ctx)
  - [Comandos de simulação de toque](#comandos-de-simulação-de-toque)
    - [Comando: abu\_read](#comando-abu_read)
    - [Comando: abu\_click](#comando-abu_click)
  - [Comandos da tela de trabalho](#comandos-da-tela-de-trabalho)
    - [Comando: w\_powersel](#comando-w_powersel)
    - [Comando: pulsesel](#comando-pulsesel)
    - [Comando: w\_dialogread](#comando-w_dialogread)


# Comandos para integração com o Thera Lase Surgery

Aqui estão documentados alguns comandos uart que podem ser úteis na integração com o Thera Lase Surgery. Para utilizá-los, ajustar a uart com Baud Rate em 833333, 8 bits, sem paridade e um stop bit. (833333,8,N,1)

Para enviar um comando para o equipamento basta transmitir a palavra de comando seguida do linefeed ('\n').

Todas as repostas aos comandos são iniciadas por "<" e finalizadas por ">". Caso a resposta ocupe múltiplas linhas, é adicionado um * ao início de cada linha. 

Durante a execução, o equipamento pode enviar algumas mensagens de debug, todas elas são iniciadas por "!<COD_3_LETRAS>". Essa mensagens devem ser descartadas pelo parser na hora de validar a resposta de um comando.

Todos os comandos documentados aqui acompanham exemplos que incluem os exatos *bytestrings* enviados e recebidos na comunicação. Os enviados estão precedidos pelo símbolo "->|" e os recebidos por "<-|". Além disso, como familiar para os programadores de python, as *bytestrings* estão entre aspas simples precedidas por um b.

## Comandos do gerenciador de atividades

### Comando: am_ctx

**Descrição**

Retorna o contexto atual, em geral, um nome compreensível para a tela que está sendo exibida em primeiro plano. 

**Uso:**

```
    am_ctx
```


**Retorno:** 

Contexto atual e contexto anterior.

Alguns nomes de contextos que podem ser relevantes:
- work: Tela de trabalho
- powersel: Tela de ajuste de potência
- modesel: Tela de ajuste de modo
- dialog: Tela de dialogo ou alerta
- presentation: Tela inicial de apresentação
- password: Tela de senha
- mainmenu: menu principal

**Exemplo**

```
->| b'am_ctx\n' 
<-| b'<\n' 
<-| b'*ctx: work\n' 
<-| b'*prev_ctx: modesel\n' 
<-| b'*>\n' 
```

## Comandos de simulação de toque

Esses comandos tem objetivo de simular o toque do usuário na tela, possibilitando a automatização da navegação entre as telas. 

### Comando: abu_read

**Descrição**
    Retorna a lista dos botões habilitados para toque na tela TouchScreen do equipamento.

**Uso:**
```
    abu_read
```

**Retorno:**

lista de botões, cada botão é apresentado no formato:

```
    <indice>: [<touchid>]   '<nome>'
```
  
O touchid é a variável que deve ser utilizada no comando abu_click

**Exemplo**

```
->| b'abu_read\n' 
<-| b'<\n' 
<-| b"*0: [236794] 'Cirurgia' \n" 
<-| b"*1: [237094] 'Configura\xe7\xf5es' \n" 
<-| b"*2: [371962] 'Terapia' \n" 
<-| b"*3: [372262] 'Bloquear' \n" 
<-| b'*>\n' 
```

### Comando: abu_click

**Descrição**

Simula o clique em um botão que está aparecendo na tela.

**Uso:**
```
    abu_click <touchid>
```

- <u>touchid</u> como recebido em abu_read (obrigatório)

**Retorno:** 

Status do comando

**Exemplo**

```
->| b'abu_click 236794\n' 
<-| b'<OK>\n' 
```

## Comandos da tela de trabalho

### Comando: w_powersel

**Descrição**

Pode ser utilizado para abrir ou fechar a tela de ajuste de potência e modificar o valor de potência do equipamento.

**Uso:**
```
    powersel call 
    powersel set <potencia> 
    powersel close
```

- <potencia> Potência em deciwatts para ajustar o equipamento, ex. 10 significa 1W

Só é permitido o ajuste de potência se a tela de ajuste estiver visível, por isso deve ser precedido do subcomando call 

**Retorno:** 

    Para o subcomando set retorna a potencia ajustada ou um erro.

**Exemplo**

```
->| b'w_powersel call\n' 
<-| b'<OK>\n' 
->| b'w_powersel set 10\n' 
<-| b'<OK 10>\n' 
->| b'w_powersel set 90\n' 
<-| b'<OK 90>\n' 
->| b'w_powersel close\n' 
<-| b'<OK>\n'
->| b'w_powersel set 90\n' 
<-| b'<ERR CTX>\n' 
```

### Comando: pulsesel

**Descrição**

Pode ser utilizado para abrir ou fechar a tela de ajuste de pulso e modificar o valor e modo de pulso.


**Uso:**

```
    w_pulsesel call
    w_pulsesel set <mode>
    w_pulsesel set 1 [time_active] [period]
    w_pulsesel set 2 [time]
    w_pulsesel close
```

  - mode: 0 para contínuo, 1 para pulsado e 2 para pulso único
  - O subcomando set pode ser utilizado apenas seguido do modo para ativar a subtela referente a esse modo.
  - Os tempos devem ser utilizados em décimos de milissegundos.

**Retorno:** 

    Modo de pulso selecionado e valores relacionados.

**Exemplo**

```
->| b'w_pulsesel call\n' 
<-| b'<OK>\n' 
->| b'w_pulsesel set 1\n' 
<-| b'<OK 1 250 50>\n' 
->| b'w_pulsesel set 2\n' 
<-| b'<OK 2 500>\n' 
->| b'w_pulsesel set 0\n' 
<-| b'<OK 0>\n' 
->| b'w_pulsesel set 1\n' 
<-| b'<OK 1 250 50>\n' 
->| b'w_pulsesel set 1 250 350\n' 
<-| b'<OK 1 250 100>\n' 
->| b'w_pulsesel set 1 250 450\n' 
<-| b'<OK 1 250 200>\n' 
->| b'w_pulsesel set 1 250 350\n' 
<-| b'<OK 1 250 100>\n' 
->| b'w_pulsesel set 1 250 300\n' 
<-| b'<OK 1 250 50>\n' 
->| b'w_pulsesel set 1 250 500\n' 
<-| b'<OK 1 250 250>\n' 
->| b'w_pulsesel set 1 550 500\n' 
<-| b'<ERR VAL>\n' 
->| b'w_pulsesel set 2\n' 
<-| b'<OK 2 5000>\n' 
->| b'w_pulsesel set 2 1000\n' 
<-| b'<OK 2 1000>\n' 
->| b'w_pulsesel set 2 5000\n' 
<-| b'<OK 2 5000>\n' 
->| b'w_pulsesel set 2 6000\n' 
<-| b'<OK 2 6000>\n' 
```

### Comando: w_dialogread

**Descrição**

Retorna o conteúdo de uma mensagem na tela do equipamento.

**Uso:**

```
    w_dialogread
```

**Retorno:** 

Caso uma subtela de mensagem ou alerta esteja visível na tela, retorna a mensagem. Retorna um erro caso contrário. Para garantir que um diálogo esteja aprecendo pode utilizar o comando <u>am_ctx</u>, que deve retornar o contexto: dialog

**Exemplo**

```
->| b'w_dialogread\n' 
<-| b'<\n' 
<-| b'*Fibra \xf3ptica detectada!\n' 
<-| b'*Fibra conectada at\xe9 o fim de curso?\n' 
<-| b'*>\n' 
```