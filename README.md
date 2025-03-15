Criar um Batch Input (BDC) no SAP é uma técnica usada para automatizar a execução de transações (como criar ordens de venda, centros de custo, etc.) por meio de programas ABAP. O BDC simula a entrada manual de dados em telas do SAP, permitindo que você execute transações de forma programática.

Aqui está uma descrição passo a passo de como criar um BDC para salvar em um programa ABAP:

1. Entender a Transação que Será Automatizada
Identifique a transação que você deseja automatizar (por exemplo, VA01 para criar ordens de venda ou KS01 para criar centros de custo).

Execute a transação manualmente no SAP e anote:

As telas (dynpros) que aparecem.

Os campos que precisam ser preenchidos.

Os códigos de função (BDC_OKCODE) usados para navegar entre as telas (por exemplo, =ENT2, =BU, /00).

2. Gravar um Script com o SHDB
Use a transação SHDB (Transaction Recorder) para gravar a execução manual da transação.

Passos:

Execute a transação SHDB.

Clique em "New Recording".

Dê um nome para a gravação e informe a transação que deseja gravar (por exemplo, VA01).

Execute a transação manualmente, preenchendo todos os campos necessários.

Após concluir, salve a gravação.

O SHDB gerará um script com os dados de todas as telas e campos preenchidos.

3. Estruturar o Programa ABAP
No programa ABAP, você precisará:

Declarar as tabelas e variáveis necessárias para o BDC:

DATA: t_bdcdata TYPE TABLE OF bdcdata,   " Tabela para armazenar os dados do BDC
      s_bdcdata TYPE bdcdata,            " Estrutura para os dados do BDC
      t_messtab TYPE TABLE OF bdcmsgcoll, " Tabela para armazenar mensagens do BDC
      wa_opt    TYPE ctu_params.          " Estrutura para opções de execução do BDC


  Preencher a tabela t_bdcdata com os dados das telas e campos, usando os formulários f_bdc_dynpro e f_bdc_field:
PERFORM f_bdc_dynpro USING 'SAPMV45A' '0101'.
PERFORM f_bdc_field USING: 'BDC_CURSOR' 'VBAK-AUART',
                           'BDC_OKCODE' '=ENT2',
                           'VBAK-AUART' 'OR'.

     Configurar as opções de execução do BDC:
wa_opt-dismode = 'A'.  " Modo de exibição: 'A' (exibe todas as mensagens)
wa_opt-updmode = 'S'.  " Modo de atualização: 'S' ( síncrono )

Executar a transação usando CALL TRANSACTION:
CALL TRANSACTION 'VA01' USING t_bdcdata
                        OPTIONS FROM wa_opt
                        MESSAGES INTO t_messtab.

Tratar as mensagens de retorno:
READ TABLE t_messtab INTO DATA(s_messtab) WITH KEY msgtyp = 'E'.
IF sy-subrc = 0.
  MESSAGE 'Erro ao executar a transação.' TYPE 'E'.
ELSE.
  MESSAGE 'Transação executada com sucesso.' TYPE 'S'.
ENDIF.

 Formulários para Preencher o BDC
Crie dois formulários para facilitar o preenchimento da tabela t_bdcdata:

f_bdc_dynpro: Para iniciar uma nova tela (dynpro):
FORM f_bdc_dynpro  USING    VALUE(program)  " Nome do programa
                            VALUE(dynpro).  " Número da tela
  CLEAR s_bdcdata.
  s_bdcdata-program  = program.  " Nome do programa
  s_bdcdata-dynpro   = dynpro.   " Número da tela
  s_bdcdata-dynbegin = 'X'.      " Indica o início de uma tela
  APPEND s_bdcdata TO t_bdcdata. " Adiciona à tabela BDC
ENDFORM.
f_bdc_field: Para preencher os campos da tela:


FORM f_bdc_field  USING    VALUE(fnam)  " Nome do campo
                           VALUE(fval). " Valor do campo
  CLEAR s_bdcdata.
  s_bdcdata-fnam = fnam.  " Nome do campo
  s_bdcdata-fval = fval.  " Valor do campo
  APPEND s_bdcdata TO t_bdcdata.  " Adiciona à tabela BDC
ENDFORM.


 Exemplo Completo de um BDC

REPORT z_bdc.

DATA: t_bdcdata TYPE TABLE OF bdcdata,
      s_bdcdata TYPE bdcdata,
      t_messtab TYPE TABLE OF bdcmsgcoll,
      wa_opt    TYPE ctu_params.

START-OF-SELECTION.

  PERFORM f_bdc_dynpro USING 'SAPMV45A' '0101'.
  PERFORM f_bdc_field USING: 'BDC_CURSOR' 'VBAK-AUART',
                             'BDC_OKCODE' '=ENT2',
                             'VBAK-AUART' 'OR'.

  PERFORM f_bdc_dynpro USING 'SAPMV45A' '4001'.
  PERFORM f_bdc_field USING: 'BDC_OKCODE' '/00',
                             'VBKD-BSTKD' '123456',
                             'KUAGV-KUNNR' '1001'.

  PERFORM f_bdc_dynpro USING 'SAPMV45A' '4001'.
  PERFORM f_bdc_field USING: 'BDC_OKCODE' '=SICH'.

  wa_opt-dismode = 'A'.  " Exibir todas as mensagens
  wa_opt-updmode = 'S'.  " Modo síncrono

  CALL TRANSACTION 'VA01' USING t_bdcdata
                          OPTIONS FROM wa_opt
                          MESSAGES INTO t_messtab.

  READ TABLE t_messtab INTO DATA(s_messtab) WITH KEY msgtyp = 'E'.
  IF sy-subrc = 0.
    MESSAGE 'Erro ao criar a ordem de venda.' TYPE 'E'.
  ELSE.
    MESSAGE 'Ordem de venda criada com sucesso.' TYPE 'S'.
  ENDIF.

FORM f_bdc_dynpro  USING    VALUE(program)
                            VALUE(dynpro).
  CLEAR s_bdcdata.
  s_bdcdata-program  = program.
  s_bdcdata-dynpro   = dynpro.
  s_bdcdata-dynbegin = 'X'.
  APPEND s_bdcdata TO t_bdcdata.
ENDFORM.

FORM f_bdc_field  USING    VALUE(fnam)
                           VALUE(fval).
  CLEAR s_bdcdata.
  s_bdcdata-fnam = fnam.
  s_bdcdata-fval = fval.
  APPEND s_bdcdata TO t_bdcdata.
ENDFORM.
