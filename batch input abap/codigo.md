
 Exemplo Completo de um BDC:

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
