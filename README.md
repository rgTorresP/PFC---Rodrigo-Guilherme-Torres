GUIA DE USO - BANCADA WATER COOLER (PFC) - VERSÃO EXECUTÁVEL
=============================================================
Autor: Rodrigo Guilherme Torres
POLI - UPE, Engenharia de Controle e Automação

Este guia explica como montar, ligar e usar a bancada com o programa
executável "PFC Water Cooler - Rodrigo Torres.exe".

1. VISÃO GERAL
--------------
O projeto tem 2 partes:

  PFC_Programa_IHM_v08.ino
      Firmware do Arduino UNO. Lê os sensores, controla bomba, fans e
      cartuchos e executa as proteções térmicas. É gravado uma vez no
      Arduino pela Arduino IDE.

  PFC Water Cooler - Rodrigo Torres.exe
      O programa do PC (interface). Mostra as leituras, envia comandos e grava
      cada ensaio em CSV e no banco de dados SQLite. Já leva dentro o gerenciador
      do banco, então não precisa instalar Python nem bibliotecas.

Caminho dos dados:

  Sensores -> Arduino -> cabo USB -> programa (.exe) -> CSV (dados/*.csv)
                                                     -> banco (dados/watercooler.db)
  CSV -> Analise_PFC.py -> resultados/ (gráficos PNG, estatísticas, CSV processado)


2. O QUE É PRECISO TER
----------------------
Hardware (como está na bancada):
  - Arduino UNO
  - 4 sensores DS18B20 no mesmo barramento OneWire (pino D2): CPU, ambiente,
    entrada do radiador e saída do radiador
  - Sensor de vazão YF-S201 (D13)
  - Tacômetro da bomba (D12) e tacômetro das fans (D7)
  - Sensor de tensão SS02 (A0) e sensor de corrente ACS712 20A (A1)
  - Saídas PWM: fans nativo (D6), fans externo/MOSFET (D9), bomba (D10),
    cartuchos aquecedores (D11)

Software:
  - Windows (o .exe só roda no Windows)
  - Arduino IDE, com as bibliotecas OneWire, DallasTemperature e
    PinChangeInterrupt (NicoHood). Só é necessária pra gravar o firmware.

3. ORGANIZAÇÃO DAS PASTAS
-------------------------
Coloque o .exe numa pasta própria. Ao abrir, ele cria a pasta "dados" ao lado
dele:

  Pasta_do_projeto/
    PFC Water Cooler - Rodrigo Torres.exe
    dados/                     criada pelo programa:
                               CSVs, watercooler.db e ultimo_ensaio.json
    Analise_PFC.py             (opcional, pra análise)
    resultados/                (criada pela análise)

IMPORTANTE: abra o .exe com duplo clique, dentro da pasta onde ele está. O
programa procura a pasta "dados" no lugar de onde foi aberto. Se abrir por um
atalho com outra pasta em "Iniciar em", ou por um prompt em outra pasta, ele
cria uma pasta "dados" nova ali e parece que os ensaios sumiram. Nada se perde,
ele só está olhando pro lugar errado.

4. REAPROVEITAR ENSAIOS JÁ FEITOS
---------------------------------
Se você já tem uma pasta "dados" com o watercooler.db de uso anterior, é só
deixar essa pasta ao lado do .exe. Ele usa o banco existente, mostra os ensaios
antigos em "Ensaios no banco" e usa o conteúdo do banco nas sugestões do
diálogo (tipo, grupo e configuração). O ultimo_ensaio.json também é
reaproveitado.

Cuidados:
  - Faça uma cópia da pasta "dados" antes do primeiro uso. O banco guarda todos
    os ensaios e é o arquivo mais importante do projeto.
  - Não abra a versão em Python e o .exe ao mesmo tempo no mesmo banco.


5. PASSO 1 - GRAVAR O FIRMWARE NO ARDUINO
-----------------------------------------
1. Abra o PFC_Programa_IHM_v08.ino na Arduino IDE.
2. Se for usar outros sensores DS18B20, troque os endereços (DeviceAddress) no
   começo do código. Cada sensor tem endereço único de fábrica.
3. Escolha a placa Arduino UNO e a porta COM, e envie.
4. Feche o Monitor Serial da IDE. Só um programa pode usar a porta por vez, e
   o programa do PC não conecta com ela aberta.

Ao ligar ou reiniciar, o Arduino calibra o ACS712 por cerca de 4 segundos.
Durante isso, bomba e cartuchos precisam estar desligados, porque ele mede a
saída do sensor com corrente zero.

Detalhe importante: abrir a porta serial reinicia o Arduino. Então toda vez
que o programa conecta, o Arduino reinicia, recalibra e os PWMs voltam ao
padrão.

6. PASSO 2 - ABRIR O PROGRAMA
-----------------------------
Dê duplo clique em "PFC Water Cooler - Rodrigo Torres.exe". O primeiro start
pode demorar alguns segundos, é normal. Alguns antivírus acusam falso positivo
em executáveis desse tipo; se acontecer, libere o arquivo.

Porta serial: a lista ao lado do botão de atualizar (↻) mostra as portas
encontradas. A COM6 aparece como padrão. Escolha a porta da sua bancada. Baud
rate: 115200.

Tela principal:
  - Cards com temperaturas, vazão, RPM das fans, potência estimada dos
    aquecedores, tensão e corrente.
  - Gráficos: aba "Temperaturas" e aba "Potência • RPM • Vazão".
  - "Registro do ensaio": arquivo CSV em uso e quantidade de leituras.
  - "Camadas de segurança": estado das proteções reportadas pelo Arduino.
  - Controle manual de PWM (4 canais), travado até ativar "Liberar ajustes".
  - Botões de ação e rodapé com a última mensagem do Arduino.


7. PASSO 3 - FAZER UM ENSAIO
----------------------------
7.1 Conectar
  Clique em "Conectar ao Arduino". Antes de abrir a porta, o programa mostra o
  diálogo "Dados do ensaio". Se cancelar, nada é criado.

  Campos:
    Nome do ensaio             nome no banco (padrão: experimento_data_hora)
    Nome do arquivo CSV        acompanha o nome do ensaio até você editar. Se o
                               arquivo já existir, vira _2, _3...
    Tipo de ensaio             ex.: repetibilidade, comparação, exploratório
    Grupo de repetibilidade    ensaios repetidos nas mesmas condições devem ter
                               exatamente o mesmo texto
    Configuração               ex.: PWM da bomba/fans/cartuchos
    Observações                texto livre
    PWM iniciais               bomba, fans nativo, fans externo e cartuchos
    Duração                    encerramento automático (hh:mm:ss)

  Os campos vêm preenchidos com o último ensaio (guardado em
  dados/ultimo_ensaio.json). Os campos de texto aparecem com sugestões já
  usadas no banco, pra evitar erro de digitação.

  Se os cartuchos estiverem acima de 0 com a bomba em 0, o programa pede
  confirmação, porque aquecer sem circulação é perigoso.

7.2 O que acontece em seguida
  1. O programa abre a porta, cria o CSV e o registro no banco.
  2. O Arduino reinicia e calibra (alguns segundos sem dados).
  3. Na primeira leitura recebida, o programa envia os PWM iniciais, nesta
     ordem: bomba, fans nativo, fans externo, cartuchos (nunca aquece antes de
     circular água).
  4. Cada leitura (1 por ciclo, mais ou menos a cada 1,5 s) é gravada no CSV e
     no banco ao mesmo tempo.

7.3 Durante o ensaio
  - Para mudar PWM: ative "Liberar ajustes" e use slider ou campo numérico.
    Cada mudança vai direto pro Arduino.
  - A duração e o encerramento automático podem ser alterados no rodapé do
    gráfico, mesmo com o ensaio rodando. O tempo conta a partir da primeira
    leitura gravada.
  - "Máximo: fans + bomba" envia M (255 nas fans e na bomba, cartuchos não são
    alterados).
  - "Desligar fans completamente" envia E0 e depois F255. Nesta bancada é essa
    combinação que desliga as fans.
  - "Comando D • PWM zero" zera os PWM conforme o firmware. Não é a mesma
    coisa que desligar as fans (E0 + F255).
  - "Status em janela" consulta o Arduino (comando S) e mostra a resposta.
  - "Ajuda e comandos" lista os comandos.

7.4 Terminar
  - Automático: ao atingir a duração, o programa desconecta (fecha CSV e banco)
    e entra sozinho no modo resfriamento.
  - Manual: clique em "Desconectar". O CSV e o registro no banco são
    finalizados.

7.5 Modo resfriamento (botão "Esfriar sistema")
  Para esfriar a bancada depois do ensaio, sem gravar nada:
  - conecta, liga bomba e fans no máximo e deixa os cartuchos em 0;
  - não cria CSV nem registro no banco;
  - ao clicar em "Desconectar", desliga cartuchos, bomba e fans em sequência e
    só então fecha a porta.
  Deixe o sistema esfriar antes de desligar a bomba.


8. PROTEÇÕES TÉRMICAS (rodam no Arduino)
----------------------------------------
Usam a temperatura da CPU (sensor tempCPU) e valem mesmo no controle manual:

  Bomba      >= 75 °C  forçada em 255; volta ao manual em 65 °C
  Fans       >= 80 °C  forçadas (nativo no PWM de proteção, externo em 255);
                       voltam ao manual em 60 °C
  Cartuchos  >= 95 °C  PWM reduzido para uma fração do comandado (padrão 30%)
             >= 100 °C PWM zerado
                       só voltam ao manual quando esfriar até 85 °C

Se você pedir um PWM durante uma proteção, o valor fica guardado e é aplicado
quando a proteção terminar.

Alterar os limites: botão "Parâmetros de segurança" (pede senha).
A senha é 28112019, data de aniversário de Enzo Torres (@enzotorres.cheetos).
Os valores são enviados ao Arduino pelo comando T e valem até ele reiniciar (ou
seja, até a próxima conexão). O programa valida a ordem dos limites (liberação
sempre menor que ativação), e o Arduino valida de novo.

Trocar a senha: ela fica na constante SENHA_CONFIG_SEGURANCA, no código da
interface. No .exe isso exige editar o código e gerar o executável de novo
(ver seção 13).


9. COMANDOS SERIAIS DO ARDUINO
------------------------------
Dá pra usar também pelo Monitor Serial da Arduino IDE (fim de linha: nova
linha), mas nunca ao mesmo tempo que o programa.

  F0-F255   PWM nativo das fans
  E0-E255   PWM externo das fans (MOSFET)
  B0-B255   PWM da bomba
  C0-C255   PWM dos cartuchos
  M         fans (nativo e externo) e bomba em 255
  D         zera todos os PWM e reseta as proteções
  S         status dos PWM (aplicado / comandado) e das proteções
  H         ajuda
  T...      9 valores separados por vírgula: limites de segurança, PWM
            restante dos cartuchos (%) e PWM das fans na proteção

As linhas de dados começam com "CSV,". É esse prefixo que o programa usa pra
separar as leituras do resto das mensagens.


10. ARQUIVOS GERADOS
--------------------
  dados/<nome>.csv           um por ensaio. Separador ";" e decimal ",", abre
                             direto no Excel brasileiro.
  dados/watercooler.db       banco SQLite com todos os ensaios e amostras.
  dados/ultimo_ensaio.json   últimas informações digitadas (sugestão pro
                             próximo ensaio).

Colunas do CSV: data e hora, tempo do Arduino, tempo em segundos, 4
temperaturas, vazão, RPM da bomba (diagnóstico), RPM das fans, tensão,
corrente, potência elétrica (V x I, diagnóstico), PWM de cada canal, estados
das proteções e potência estimada dos aquecedores.

A potência estimada dos aquecedores é linear no PWM:
  P = POTENCIA_AQUECEDORES_MAX_W x PWM_cartuchos / 255
É uma estimativa. Tensão e corrente medidas aparecem separadas.


11. GERENCIAR OS ENSAIOS NO BANCO
---------------------------------
Pelo programa: botão "Ensaios no banco".
  - Lista todos os ensaios, com filtro por texto.
  - Editar (duplo clique ou botão): um ensaio altera todos os campos; vários
    ensaios alteram só os campos marcados com "alterar" (bom pra colocar o
    mesmo grupo em vários de uma vez).
  - Excluir: remove o ensaio e as amostras do banco. O CSV NÃO é apagado. Não
    dá pra excluir o ensaio que está sendo gravado.

Importar CSVs antigos ou consultar o banco fora do programa (precisa de Python
e do arquivo banco_dados_watercooler.py):

  python banco_dados_watercooler.py --listar
  python banco_dados_watercooler.py --importar dados/arquivo.csv --nome "Meu ensaio" --tipo repetibilidade --grupo G1 --configuracao "B255 F180"

O banco usa o hash do arquivo, então importar o mesmo CSV duas vezes não
duplica o ensaio.

Pelo Python (por exemplo, num notebook):

  from banco_dados_watercooler import listar_ensaios, carregar_ensaio
  listar_ensaios()          # tabela com todos os ensaios
  df = carregar_ensaio(3)   # amostras do ensaio de ID 3


12. PROBLEMAS COMUNS
--------------------
"Não foi possível abrir COMx"
  Feche o Monitor Serial da Arduino IDE (ou outro programa usando a porta),
  confira a porta na lista e o cabo USB.

Nenhum dado aparece depois de conectar
  Normal por alguns segundos: o Arduino reinicia e calibra o ACS712.

Os ensaios antigos não aparecem
  O .exe foi aberto de outra pasta e criou uma pasta "dados" nova. Feche, e
  abra com duplo clique dentro da pasta onde está o banco (seção 3).

O antivírus bloqueou o .exe
  Falso positivo comum em executáveis gerados com PyInstaller. Libere o
  arquivo.

Temperatura em -127 °C
  Sensor DS18B20 desconectado ou com mau contato. A análise descarta esses
  valores.

Fans não desligam
  O PWM nativo tem um piso de rotação (~800 RPM mesmo em PWM 0). Pra desligar
  de verdade use "Desligar fans completamente" (E0 + F255).

Os controles de PWM estão cinza
  Eles só funcionam conectado e com "Liberar ajustes" ativo.

Banco de dados indisponível
  O programa avisa e continua gravando o CSV normalmente.

Parâmetros de segurança não "pegam"
  Precisa do Arduino conectado. Confira a resposta [OK] ou [ERRO] na barra de
  status. Os valores voltam ao padrão quando o Arduino reinicia.

