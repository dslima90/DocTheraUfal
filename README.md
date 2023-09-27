- [Manual de integração com o Thera Lase Surgery](#manual-de-integração-com-o-thera-lase-surgery)
  - [Comandos](#comandos)
    - [Comandos do gerenciador de atividades](#comandos-do-gerenciador-de-atividades)
      - [Comando: am\_ctx](#comando-am_ctx)
    - [Comandos de simulação de toque](#comandos-de-simulação-de-toque)
      - [Comando: abu\_read](#comando-abu_read)
      - [Comando: abu\_click](#comando-abu_click)
    - [Comandos da tela de trabalho](#comandos-da-tela-de-trabalho)
      - [Comando: w\_powersel](#comando-w_powersel)
      - [Comando: w\_pulsesel](#comando-w_pulsesel)
      - [Comando: w\_dialogread](#comando-w_dialogread)
      - [Comando: w\_resette](#comando-w_resette)
  - [Eventos](#eventos)
    - [Evento: CTX\_CHG](#evento-ctx_chg)
    - [Evento: BTN\_CLK](#evento-btn_clk)
    - [Evento: PAR\_EDT](#evento-par_edt)
    - [Evento: PAR\_CHG](#evento-par_chg)
    - [Evento: LSR\_CHG](#evento-lsr_chg)


# Manual de integração com o Thera Lase Surgery

Aqui estão documentados alguns comandos e eventos uart que podem ser úteis na integração com o Thera Lase Surgery. Para utilizá-los, ajustar a uart com Baud Rate em 833333, 8 bits, sem paridade e um stop bit. (833333,8,N,1)

Durante a execução, o equipamento pode enviar algumas mensagens de debug, todas elas são iniciadas por "!<COD_3_LETRAS>". Essa mensagens devem ser descartadas pelo parser na hora de validar a resposta de um comando.

Toda a documentação contida aqui se refere a versão do kernel *1.2.2*.

## Comandos

Para enviar um comando para o equipamento basta transmitir a palavra de comando seguida do linefeed ('\n').

Todas as repostas aos comandos são iniciadas por "<" e finalizadas por ">". Caso a resposta ocupe múltiplas linhas, é adicionado um * ao início de cada linha. 

Todos os comandos documentados aqui acompanham exemplos que incluem os exatos *bytestrings* enviados e recebidos na comunicação. Os enviados estão precedidos pelo símbolo "->|" e os recebidos por "<-|". Além disso, como familiar para os programadores de python, as *bytestrings* estão entre aspas simples precedidas por um b.

### Comandos do gerenciador de atividades

#### Comando: am_ctx

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

### Comandos de simulação de toque

Esses comandos tem objetivo de simular o toque do usuário na tela, possibilitando a automatização da navegação entre as telas. 

#### Comando: abu_read

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

#### Comando: abu_click

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

### Comandos da tela de trabalho

#### Comando: w_powersel

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

#### Comando: w_pulsesel

**Descrição**

Pode ser utilizado para abrir ou fechar a tela de ajuste de pulso e modificar o valor e modo de pulso.


**Uso:**

```
    w_pulsesel call
    w_pulsesel set <mode>
    w_pulsesel set 1 [time_active] [period] # modo pulsado
    w_pulsesel set 2 [time]                 # modo puls. único
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

#### Comando: w_dialogread

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

#### Comando: w_resette

**Descrição**

Zera o tempo e energia.

**Uso:**

```
    w_resette
```

**Retorno:** 

Ok ou erro de contexto.

**Exemplo**

```
->| b'w_resette\n' 
<-| b'<OK>\n' 
```
## Eventos

Os eventos são mensagens enviadas pelo equipamento que indicam alguma mudança no estado interno do equipamento a fim de sincronizar as duas interfaces. Todos os eventos enviados são precedidos pelo simbolo '+' seguido de 7 caracteres com o nome do evento, um ou mais parametros separados por espaço podem vir em seguida e são finalizados por um linefeed '\n'. 

Segue a descrição de alguns eventos que podem ser detectados:

### Evento: CTX_CHG

**Descrição**

Indica uma mudança de contexto, que pode ser uma mudança de tela ou aparecimento uma tela flutuante.

**Formato** 

```
+CTX_CHG <CONTEXTO_ANTERIOR> <CONTEXTO_ATUAL> <CONTEXTO_NESTED>
```

Contexto nested indica uma tela flutuante, o valor NULL no lugar desse parâmetro indica inexistencia de uma.

**Exemplos** 

```
<-| b'+CHG_CTX NULL presentation NULL\n'
<-| b'+CHG_CTX presentation password NULL\n'
<-| b'+CHG_CTX password mainmenu NULL\n' 
<-| b'+CHG_CTX mainmenu modesel NULL\n' 
<-| b'+CHG_CTX modesel work NULL\n' 
<-| b'+CHG_CTX modesel work dialog\n' 
```

### Evento: BTN_CLK

**Descrição**

Indica um evento de click em um botão.

**Formato** 

```
+BTN_CLK <CONTEXTO> <TOUCH_ID> <TIPO_EVENTO>
```

Sendo que:

- CONTEXTO: indica o contexto em que o botão foi clicado
- TOUCH_ID: Touch id do botão, único, tal como retornado por abu_read
- TIPO_EVENTO: 4 indica que o botão foi pressionado, 3 indica que o botão foi solto.

**Exemplos** 

Colocação da senha: 2276

```
<-| b'+BTN_CLK password 191044 4\n' 
<-| b'+BTN_CLK password 191044 3\n' 
<-| b'+BTN_CLK password 191044 4\n' 
<-| b'+BTN_CLK password 191044 3\n' 
<-| b'+BTN_CLK password 334314 4\n' 
<-| b'+BTN_CLK password 334314 3\n' 
<-| b'+BTN_CLK password 262814 4\n' 
<-| b'+BTN_CLK password 262814 3\n' 
```

### Evento: PAR_EDT

**Descrição**

Indica se o equipamento mudou o seu estado para o modo de edição de parametros e 

**Formato** 

```
+PAR_EDT <ESTADO> [MODO] 
```

- ESTADO: 1 indica que está em modo de edição, 0 indica que saiu do modo de edição
- MODO: Modo indica qual o modo de edição atual, pode ser "TOUCH" ou "GLASS", indicado apenas quando o equipamento está entrando no modo de edição

**Exemplos** 


```
<-| b'+PAR_EDT 1 TOUCH\n'
<-| b'+PAR_EDT 0\n' 
```


### Evento: PAR_CHG

**Descrição**

Indica que algum parametro de ajuste do laser foi modificado. 

**Formato** 


```
+PAR_CHG <POTENCIA> <MODO_PULSO> [<PARAMETROS_PULSO>]
```

- POTENCIA: Potencia ajustada, no mesmo formato de w_powersel
- MODO_PULSO: Modo de pulso ajustado, no mesmo formato de w_pulsesel
- PARAMETROS_PULSO: Apenas se o modo for pulsado ou pulso único, segue o modelo de w_pulsesel


**Exemplos** 


```
<-| b'+PAR_CHG 20 0\n' 
<-| b'+PAR_CHG 20 1 250 50\n' 
<-| b'+PAR_CHG 20 2 500\n' 
```

### Evento: LSR_CHG

**Descrição**

Indica uma mudança no estado do laser. Esse evento é anunciado sempre que o estado de prontidão/disponível é modificado ou que o laser é acionado ou desacionado. Além disso, quando o laser está acionado, ele indica a modificação no tempo de acionamento a cada 500 ms.

**Formato** 

```
+LSR_CHG <VALOR> <TEMPO> <ENERGIA>
```

- VALOR: Valor atual do estado do laser 0 = prontidão, 1 = disponível, 2 = acionado
- TEMPO: Tempo total em segundos de acionamento do laser.
- ENERGIA: Energia total de acionamento do laser.

**Exemplos** 


```
<-| b'+LSR_CHG 0 0.000000 0.000000\n' 
<-| b'+LSR_CHG 1 0.000000 0.000000\n' 
<-| b'+LSR_CHG 0 0.000000 0.000000\n' 
<-| b'+LSR_CHG 1 0.000000 0.000000\n' 
<-| b'+LSR_CHG 2 0.346400 0.173200\n' 
<-| b'+LSR_CHG 2 0.613900 0.306950\n' 
<-| b'+LSR_CHG 2 0.881400 0.440700\n' 
<-| b'+LSR_CHG 2 1.198900 0.599450\n' 
<-| b'+LSR_CHG 2 1.416400 0.708200\n' 
<-| b'+LSR_CHG 2 1.724900 0.862450\n' 
<-| b'+LSR_CHG 2 1.974900 0.987450\n' 
<-| b'+LSR_CHG 2 2.224900 1.112450\n' 
<-| b'+LSR_CHG 2 2.486400 1.243200\n' 
<-| b'+LSR_CHG 2 2.636400 1.318200\n' 
<-| b'+LSR_CHG 1 2.636400 1.318200\n' 
```