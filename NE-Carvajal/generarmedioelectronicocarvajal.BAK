*/
*!*
*!*		Nombre: Generar y enviar XML Carvajal Nómina Electrónica via Web Service (V6) - ERP OFIMA
*!*
*!*		Autor: Nicolás David Cubillos
*!*
*!*		Contenido: Función main: enviarMedio. | El PRG contiene todas las funciones encargadas de generar
*!*				   y/o enviar el medio electrónico XML Carvajal por medio del Web Service en su versión 6 (1 OCT 2021).
*!*
*!*		Fecha: 29 septiembre 2021.
*!*
*!*		Versión: 1.4 - Octubre 29 de 2021.
*!*		Versión: 2.0 - Enero 10 de 2022.
*/

#DEFINE ccCRLF CHR(13) + CHR(10) && Salto de línea
#DEFINE ccTAB CHR(9) && Tabulación

*---------------------------------------------------

FUNCTION enviarMedio && Función principal: Hace uso de las demás funciones existentes en el PRG.
*!* 	    Retorna verdadero si se timbra el medio, falso si ocurre algún error en cualquier momento.

LPARAMETERS MEDIOBASE64, CURRENTPATH, ON, C_GLOBAL, TIPODCTO, NRODCTO, NOMBREARCHIVO, VERREQUESTRESPONSE

STORE "" TO TRANSACTIONID
STORE "" TO DIANCUNESHA256

LOCAL XMLrequest
SELECT C_GLOBAL

XMLrequest = crearRequestUpload(S_USUARIO, S_CONTRASENA, S_COMPANYID, S_ACCOUNTID, MEDIOBASE64, NOMBREARCHIVO)
TRANSACTIONID = enviarRequest(XMLrequest, CURRENTPATH, TIPODCTO, NRODCTO, 1, VERREQUESTRESPONSE)

IF EMPTY(TRANSACTIONID)
	RETURN .F.
ELSE
	almacenarTransacID(TIPODCTO, NRODCTO, TRANSACTIONID, ON)

*!*				XMLrequest = crearRequestDownload(S_USUARIO, S_CONTRASENA, S_COMPANYID, S_ACCOUNTID, TIPODCTO, NRODCTO)
*!*				DIANCUNESHA256 = enviarRequest(XMLrequest, CURRENTPATH, TIPODCTO, NRODCTO, 3)
*!*				IF EMPTY(DIANCUNESHA256)
*!*					RETURN .F.
*!*
*!*				ENDIF

*!*				&&almacenarCuneDIAN(TIPODCTO, NRODCTO, DIANCUNESHA256, ON)
*!*
*!*				MESSAGEBOX("CUNEDIAN:" + DIANCUNESHA256)

*!*			XMLrequest = crearRequestDocumentStatus(S_USUARIO, S_CONTRASENA, S_COMPANYID, S_ACCOUNTID, TRANSACTIONID)
*!*			_CLIPTEXT = XMLrequest
*!*			WAIT "" TIMEOUT 2
*!*			&&MESSAGEBOX("Copiar acá el XML request.")
*!*			IF enviarRequest(XMLrequest, CURRENTPATH, TIPODCTO, NRODCTO, 2) = "T"
*!*				MESSAGEBOX("PASO BIEN" + TIPODCTO + NRODCTO")
*!*			ELSE
*!*				RETURN .F.
*!*			ENDIF

ENDIF

RETURN .T. && Medio subido.

ENDFUNC

*---------------------------------------------------

FUNCTION FORMATEARFECHA AS STRING && Formatea una fecha recibida por paráemtro a una cadena de caracteres separada por '-'

LPARAMETERS FECHA
STORE "" TO RESULT
RESULT = TRANSFORM(YEAR(FECHA)) + "-"
RESULT = RESULT + IIF(MONTH(FECHA) <10, "0", "") && Si el día está entre 1 y 9, le agrega un 0 antes. Ej día 9 = Queda día 09.
RESULT = RESULT + TRANSFORM(MONTH(FECHA)) + "-"
RESULT = RESULT + IIF(DAY(FECHA) < 10, "0", "")
RESULT = RESULT + TRANSFORM(DAY(FECHA))
RETURN RESULT

ENDFUNC

*---------------------------------------------------

FUNCTION GENERARMEDIO AS STRING && Función que genera el XML y retorna el mismo encriptado en BASE64.

LPARAMETERS C_EMPLEADO, C_GLOBAL, C_DETALLEEMPLEADO, CURRENTPATH, ON, NOMBREARCHIVO
&& Recibe por parámetros los datos del empleado, datos globales del emisor, el detalle de nómina del empleado y el PATH en el que se
&& almacenará el medio, conexión a SQL y el nombre del archivo con el que se almacenará en el FileSystem en la ubicación PATH.

LOCAL C_EMISOR AS CURSOR

STORE "" TO returnBase64
STORE "" TO FECHAACTUAL
STORE "" TO HORAACTUAL
STORE "" TO CODIGOEMPLEADO

SELECT C_EMPLEADO

IF DIRECTORY(CURRENTPATH + "NOMINA_ELECTRONICA\") = .F.
	MKDIR CURRENTPATH + "\NOMINA_ELECTRONICA"
ENDIF

SET CENTURY OFF

pathSalida = CURRENTPATH + "NOMINA_ELECTRONICA\" + NOMBREARCHIVO && Ruta quemada.

TRY && Generando el medio, etiqueta por etiqueta con espacios y tabulaciones, el objeto MSXML2.DOMDOCUMENT no tabula automáticamente.
	oXML = CREATEOBJECT("MSXML2.DOMDOCUMENT.4.0")

	oXML.appendChild(oXML.createNode("PROCESSINGINSTRUCTION", "xml", ""))
	oNomina = oXML.appendChild(oXML.createElement("NOMINA"))
	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))
	oENC = oNomina.appendChild(oXML.createElement("ENC"))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC1 = oENC.appendChild(oXML.createElement("ENC_1"))
	oENC1.TEXT = "NominaIndividual"

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC2 = oENC.appendChild(oXML.createElement("ENC_2"))
	oENC2.TEXT = FORMATEARFECHA(S_FECHAING)

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC3 = oENC.appendChild(oXML.createElement("ENC_3"))

	IF (MONTH(S_FECRETIRO) = YEAR(DATE()) AND YEAR(S_FECRETIRO) = YEAR(DATE()))
		oENC3.TEXT = FORMATEARFECHA(S_FECRETIRO)
	ELSE
		oENC3.TEXT = ""
	ENDIF

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC4 = oENC.appendChild(oXML.createElement("ENC_4"))
	oENC4.TEXT = ALLTRIM(FORMATEARFECHA(S_FECINI))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC5 = oENC.appendChild(oXML.createElement("ENC_5"))
	oENC5.TEXT = ALLTRIM(FORMATEARFECHA(S_FECFIN))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC6 = oENC.appendChild(oXML.createElement("ENC_6"))
	oENC6.TEXT = ALLTRIM(CAST(S_TIEMPOLAB AS CHARACTER (10)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC7 = oENC.appendChild(oXML.createElement("ENC_7"))
	oENC7.TEXT = FORMATEARFECHA(DATE())

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC8 = oENC.appendChild(oXML.createElement("ENC_8"))
	oENC8.TEXT = ALLTRIM(CAST(S_CODIGO AS CHARACTER (15)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC9 = oENC.appendChild(oXML.createElement("ENC_9"))
	oENC9.TEXT = ALLTRIM(S_TIPODCTO)

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC10 = oENC.appendChild(oXML.createElement("ENC_10"))
	oENC10.TEXT = ALLTRIM(CAST(S_NRODCTO AS CHARACTER (10)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC11 = oENC.appendChild(oXML.createElement("ENC_11"))
	oENC11.TEXT = ALLTRIM(S_CONSECUT)

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC12 = oENC.appendChild(oXML.createElement("ENC_12"))
	oENC12.TEXT = ALLTRIM(CAST(S_PAIS AS CHARACTER(10)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC13 = oENC.appendChild(oXML.createElement("ENC_13"))
	oENC13.TEXT = ALLTRIM(CAST(S_DPTO AS CHARACTER (10)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC14 = oENC.appendChild(oXML.createElement("ENC_14"))
	oENC14.TEXT = ALLTRIM(CAST(S_MCPIO AS CHARACTER (10)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC15 = oENC.appendChild(oXML.createElement("ENC_15"))
	oENC15.TEXT = "es"

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC16 = oENC.appendChild(oXML.createElement("ENC_16"))
	oENC16.TEXT = "V1.0: Documento Soporte de Pago de Nómina Electrónica" && Validar si es documento de ajuste.

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC17 = oENC.appendChild(oXML.createElement("ENC_17")) && Bajar del cursor que viene de mtglobal.
	oENC17.TEXT = ALLTRIM(CAST(C_GLOBAL.S_AMBIENTE AS CHARACTER (2)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC18 = oENC.appendChild(oXML.createElement("ENC_18"))
	oENC18.TEXT = "102" && Validar si es documento de ajuste.

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC19 = oENC.appendChild(oXML.createElement("ENC_19"))
	oENC19.TEXT = "" && CUNE.

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC20 = oENC.appendChild(oXML.createElement("ENC_20"))
	oENC20.TEXT = FORMATEARFECHA(DATE())

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC21 = oENC.appendChild(oXML.createElement("ENC_21"))
	oENC21.TEXT = FECHAACTUAL(2)

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC22 = oENC.appendChild(oXML.createElement("ENC_22"))
	oENC22.TEXT = ALLTRIM(CAST(S_PERIODOPAGO AS CHARACTER(5)))

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC23 = oENC.appendChild(oXML.createElement("ENC_23"))
	oENC23.TEXT = "COP"

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC24 = oENC.appendChild(oXML.createElement("ENC_24"))
	oENC24.TEXT = ""

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC25 = oENC.appendChild(oXML.createElement("ENC_25"))
	oENC25.TEXT = ""

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC26 = oENC.appendChild(oXML.createElement("ENC_26"))
	oENC26.TEXT = ""

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oENC27 = oENC.appendChild(oXML.createElement("ENC_27"))
	oENC27.TEXT = ""

	oENC.appendChild(oXML.createTextNode(ccCRLF))
	oENC.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

	oEMI = oNomina.appendChild(oXML.createElement("EMI"))

	SELECT C_GLOBAL

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI1 = oEMI.appendChild(oXML.createElement("EMI_1"))
	oEMI1.TEXT = ALLTRIM(CAST(S_NOMCIA AS CHARACTER(100)))

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI2 = oEMI.appendChild(oXML.createElement("EMI_2"))
	oEMI2.TEXT = ""

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI3 = oEMI.appendChild(oXML.createElement("EMI_3"))
	oEMI3.TEXT = ""

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI4 = oEMI.appendChild(oXML.createElement("EMI_4"))
	oEMI4.TEXT = ""

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI5 = oEMI.appendChild(oXML.createElement("EMI_5"))
	oEMI5.TEXT = ""

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI6 = oEMI.appendChild(oXML.createElement("EMI_6"))
	oEMI6.TEXT = CAST(SUBSTR(S_NITCIA, 1, 9) AS CHARACTER(9))

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI7 = oEMI.appendChild(oXML.createElement("EMI_7"))
	oEMI7.TEXT = CAST(SUBSTR(S_NITCIA, 11, 12) AS CHARACTER(1))

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI8 = oEMI.appendChild(oXML.createElement("EMI_8"))
	oEMI8.TEXT = ALLTRIM((CAST(S_PAIS AS CHARACTER(5))))

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI9 = oEMI.appendChild(oXML.createElement("EMI_9"))
	oEMI9.TEXT = "11"

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI10 = oEMI.appendChild(oXML.createElement("EMI_10"))
	oEMI10.TEXT = "11001"

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oEMI11 = oEMI.appendChild(oXML.createElement("EMI_11"))
	oEMI11.TEXT = ALLTRIM(CAST(S_DIRECCION AS CHARACTER(100)))

	oEMI.appendChild(oXML.createTextNode(ccCRLF))
	oEMI.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

	oREC = oNomina.appendChild(oXML.createElement("REC"))

	SELECT C_EMPLEADO

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC1 = oREC.appendChild(oXML.createElement("REC_1"))
	oREC1.TEXT = ALLTRIM(CAST(S_TIPOEMPLEADO AS CHARACTER(15)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC2 = oREC.appendChild(oXML.createElement("REC_2"))
	oREC2.TEXT = ALLTRIM(CAST(S_SUBTIPOEMPLEADO AS CHARACTER(15)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC3 = oREC.appendChild(oXML.createElement("REC_3"))
	oREC3.TEXT = "false"

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC4 = oREC.appendChild(oXML.createElement("REC_4"))
	oREC4.TEXT = ALLTRIM(CAST(S_TIPDOC AS CHARACTER(15)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC5 = oREC.appendChild(oXML.createElement("REC_5"))
	oREC5.TEXT = ALLTRIM(CAST(S_CEDULA AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC6 = oREC.appendChild(oXML.createElement("REC_6"))
	oREC6.TEXT = ALLTRIM(CAST(S_APELLIDO AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC7 = oREC.appendChild(oXML.createElement("REC_7"))
	oREC7.TEXT = ALLTRIM(CAST(S_APELLIDO2 AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC8 = oREC.appendChild(oXML.createElement("REC_8"))
	oREC8.TEXT = ALLTRIM(CAST(S_NOMBRE AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC9 = oREC.appendChild(oXML.createElement("REC_9"))
	oREC9.TEXT = ALLTRIM(CAST(S_NOMBRE2 AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC10 = oREC.appendChild(oXML.createElement("REC_10"))
	oREC10.TEXT = ALLTRIM(CAST(S_PAIS AS CHARACTER(50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC11= oREC.appendChild(oXML.createElement("REC_11"))
	oREC11.TEXT = ALLTRIM(CAST(S_DPTO AS CHARACTER (10)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC12= oREC.appendChild(oXML.createElement("REC_12"))
	oREC12.TEXT = ALLTRIM(CAST(S_MCPIO AS CHARACTER (10)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC13= oREC.appendChild(oXML.createElement("REC_13"))
	oREC13.TEXT = ALLTRIM(CAST(S_DIRECCION AS CHARACTER (50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC14= oREC.appendChild(oXML.createElement("REC_14"))
	oREC14.TEXT = ALLTRIM(CAST(S_SALARIOINTEG AS CHARACTER (10)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC15= oREC.appendChild(oXML.createElement("REC_15"))
	oREC15.TEXT = ALLTRIM(CAST(S_TIPCONTRA AS CHARACTER (10)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC16= oREC.appendChild(oXML.createElement("REC_16"))
	oREC16.TEXT = ALLTRIM(CAST(S_SUELDO AS CHARACTER (50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oREC17= oREC.appendChild(oXML.createElement("REC_17"))
	oREC17.TEXT = ALLTRIM(CAST(S_CODIGO AS CHARACTER (50)))

	oREC.appendChild(oXML.createTextNode(ccCRLF))
	oREC.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))
	oPAG = oNomina.appendChild(oXML.createElement("PAG"))

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oPAG1= oPAG.appendChild(oXML.createElement("PAG_1"))
	oPAG1.TEXT = "1"

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oPAG2= oPAG.appendChild(oXML.createElement("PAG_2"))
	oPAG2.TEXT = ALLTRIM(CAST(S_METODOPAG AS CHARACTER(50)))

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oPAG3= oPAG.appendChild(oXML.createElement("PAG_3"))
	oPAG3.TEXT = ALLTRIM(CAST(S_BANCO AS CHARACTER (50)))

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oPAG4= oPAG.appendChild(oXML.createElement("PAG_4"))
	oPAG4.TEXT = ALLTRIM(CAST(S_TIPOCUENTA AS CHARACTER (50)))

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oPAG5= oPAG.appendChild(oXML.createElement("PAG_5"))
	oPAG5.TEXT = ALLTRIM(CAST(S_CTACTE AS CHARACTER (50)))

	oPAG.appendChild(oXML.createTextNode(ccCRLF))
	oPAG.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

	oFEP = oNomina.appendChild(oXML.createElement("FEP"))

	oFEP.appendChild(oXML.createTextNode(ccCRLF))
	oFEP.appendChild(oXML.createTextNode(ccTAB))
	oFEP.appendChild(oXML.createTextNode(ccTAB))

	oFEP1= oFEP.appendChild(oXML.createElement("FEP_1"))
	oFEP1.TEXT = FORMATEARFECHA(S_FECFIN)

	oFEP.appendChild(oXML.createTextNode(ccCRLF))
	oFEP.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

	oITE = oNomina.appendChild(oXML.createElement("ITE"))

	oITE.appendChild(oXML.createTextNode(ccCRLF))
	oITE.appendChild(oXML.createTextNode(ccTAB))
	oITE.appendChild(oXML.createTextNode(ccTAB))

	oITE1= oITE.appendChild(oXML.createElement("ITE_1"))
	oITE1.TEXT = ALLTRIM(CAST(S_DIASPAG AS CHARACTER (50)))

	oITE.appendChild(oXML.createTextNode(ccCRLF))
	oITE.appendChild(oXML.createTextNode(ccTAB))
	oITE.appendChild(oXML.createTextNode(ccTAB))

	oITE2= oITE.appendChild(oXML.createElement("ITE_2"))
	oITE2.TEXT = "0"

	oITE.appendChild(oXML.createTextNode(ccCRLF))
	oITE.appendChild(oXML.createTextNode(ccTAB))
	oITE.appendChild(oXML.createTextNode(ccTAB))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

	oITS = oNomina.appendChild(oXML.createElement("ITS"))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))

	oITS.appendChild(oXML.createTextNode(ccCRLF))
	oITS.appendChild(oXML.createTextNode(ccTAB))
	oITS.appendChild(oXML.createTextNode(ccTAB))

	oITS1 = oITS.appendChild(oXML.createElement("ITS_1")) && salud
	oITS1.TEXT = "0.00"

	oITS.appendChild(oXML.createTextNode(ccCRLF))
	oITS.appendChild(oXML.createTextNode(ccTAB))
	oITS.appendChild(oXML.createTextNode(ccTAB))

	oITS2 = oITS.appendChild(oXML.createElement("ITS_2")) && salud
	oITS2.TEXT = "0.00"

	oITS.appendChild(oXML.createTextNode(ccCRLF))
	oITS.appendChild(oXML.createTextNode(ccTAB))
	oITS.appendChild(oXML.createTextNode(ccTAB))

	oSPE = oITS.appendChild(oXML.createElement("SPE"))

	oSPE.appendChild(oXML.createTextNode(ccCRLF))
	oSPE.appendChild(oXML.createTextNode(ccTAB))
	oSPE.appendChild(oXML.createTextNode(ccTAB))
	oSPE.appendChild(oXML.createTextNode(ccTAB))

	oSPE1 = oSPE.appendChild(oXML.createElement("SPE_1"))
	oSPE1.TEXT = "0.00"

	oSPE.appendChild(oXML.createTextNode(ccCRLF))
	oSPE.appendChild(oXML.createTextNode(ccTAB))
	oSPE.appendChild(oXML.createTextNode(ccTAB))
	oSPE.appendChild(oXML.createTextNode(ccTAB))

	oSPE2= oSPE.appendChild(oXML.createElement("SPE_2"))
	oSPE2.TEXT = "0.00"

	oSPE.appendChild(oXML.createTextNode(ccCRLF))
	oSPE.appendChild(oXML.createTextNode(ccTAB))

	oSPE.appendChild(oXML.createTextNode(ccTAB))

	oITS.appendChild(oXML.createTextNode(ccCRLF))
	oITS.appendChild(oXML.createTextNode(ccTAB))

	SELECT C_DETALLEEMPLEADO && Se trae el detalle del empleado de MVLIQNOMNE y se baja al XML a las etiquetas que correspondan.

	GO TOP

	SCAN

		IF S_VALOR != 0
			DO CASE
			CASE S_CONCEPDIAN = '105' && CONCEPTO DIAN 105 PARA HORAS EXTRAS
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEHEG = oITE.appendChild(oXML.createElement("EHE"))

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE1 = oEHEG.appendChild(oXML.createElement("EHE_1"))
				oEHE1.TEXT = ALLTRIM(CAST(S_TIPOHE AS CHARACTER (2)))

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE2 = oEHEG.appendChild(oXML.createElement("EHE_2"))
				oEHE2.TEXT = ""

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE3 = oEHEG.appendChild(oXML.createElement("EHE_3"))
				oEHE3.TEXT = ""

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE4 = oEHEG.appendChild(oXML.createElement("EHE_4"))
				oEHE4.TEXT = ALLTRIM(CAST(S_NROHORAS AS CHARACTER (50)))

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE5 = oEHEG.appendChild(oXML.createElement("EHE_5"))
				oEHE5.TEXT = ALLTRIM(CAST(S_PORCENTAJEHE AS CHARACTER (100)))

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

				oEHE6 = oEHEG.appendChild(oXML.createElement("EHE_6"))
				oEHE6.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (100)))

				oEHEG.appendChild(oXML.createTextNode(ccCRLF))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))
				oEHEG.appendChild(oXML.createTextNode(ccTAB))

			CASE S_CONCEPDIAN = '101' && BASICO
				oITE2.TEXT = ALLTRIM(CAST(CAST(oITE2.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '102' && AUXILIO TRANSPORTE
				IF VARTYPE (oETR) = 'U'
					oETR = oITE.appendChild(oXML.createElement("ETR"))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR1 = oETR.appendChild(oXML.createElement("ETR_1"))
					oETR1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR2 = oETR.appendChild(oXML.createElement("ETR_2"))
					oETR2.TEXT = ""

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR3 = oETR.appendChild(oXML.createElement("ETR_3"))
					oETR3.TEXT = ""

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

				ELSE
					oETR1.TEXT = ALLTRIM(CAST(CAST(oETR1.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '103' && VIATICO SALARIALES
				IF VARTYPE (oETR) = 'U'
					oETR = oITE.appendChild(oXML.createElement("ETR"))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR1 = oETR.appendChild(oXML.createElement("ETR_1"))
					oETR1.TEXT = ""

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR2 = oETR.appendChild(oXML.createElement("ETR_2"))
					oETR2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR3 = oETR.appendChild(oXML.createElement("ETR_3"))
					oETR3.TEXT = "0"

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

				ELSE
					oETR2.TEXT = ALLTRIM(CAST(CAST(oETR2.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '104' && VIATICO NO SALARIALES
				IF VARTYPE (oETR) = 'U'
					oETR = oITE.appendChild(oXML.createElement("ETR"))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR1 = oETR.appendChild(oXML.createElement("ETR_1"))
					oETR1.TEXT = ""

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR2 = oETR.appendChild(oXML.createElement("ETR_2"))
					oETR2.TEXT = "0"

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

					oETR3 = oETR.appendChild(oXML.createElement("ETR_3"))
					oETR3.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oETR.appendChild(oXML.createTextNode(ccCRLF))
					oETR.appendChild(oXML.createTextNode(ccTAB))
					oETR.appendChild(oXML.createTextNode(ccTAB))

				ELSE
					oETR3.TEXT = ALLTRIM(CAST(CAST(oETR3.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF


			CASE S_CONCEPDIAN = '201' && CONCEPTO DIAN SALUD
				oITS1.TEXT = ALLTRIM(CAST(C_GLOBAL.S_PORCENTAJESALUD AS CHARACTER (50)))
				oITS2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '202' && CONCEPTO DIAN PENSIÓN
				oSPE1.TEXT = ALLTRIM(CAST(C_GLOBAL.S_PORCENTAJEPENSION AS CHARACTER (50)))
				oSPE2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '106' && CONCEPTO DIAN VACACIONES DISFRUTADAS
				IF VARTYPE(oEVC) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVC = oITE.appendChild(oXML.createElement("EVC"))

					oEVC.appendChild(oXML.createTextNode(ccCRLF))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))

					oEVC1 = oEVC.appendChild(oXML.createElement("EVC_1"))
					oEVC1.TEXT = ""

					oEVC.appendChild(oXML.createTextNode(ccCRLF))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))

					oEVC2 = oEVC.appendChild(oXML.createElement("EVC_2"))
					oEVC2.TEXT = ""

					oEVC.appendChild(oXML.createTextNode(ccCRLF))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))

					oEVC3 = oEVC.appendChild(oXML.createElement("EVC_3"))
					oEVC3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

					oEVC.appendChild(oXML.createTextNode(ccCRLF))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))

					oEVC4 = oEVC.appendChild(oXML.createElement("EVC_4"))
					oEVC4.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEVC.appendChild(oXML.createTextNode(ccCRLF))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
					oEVC.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oEVC3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))
					oEVC4.TEXT = ALLTRIM(CAST(CAST(oEVC4.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '107' && CONCEPTO DIAN VACACIONES COMPENSADAS
				IF VARTYPE(oEVA) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVA = oITE.appendChild(oXML.createElement("EVA"))

					oEVA.appendChild(oXML.createTextNode(ccCRLF))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
					oEVA.appendChild(oXML.createTextNode(ccTAB))

					oEVA1 = oEVA.appendChild(oXML.createElement("EVA_1"))
					oEVA1.TEXT = "0" &&& ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

					oEVA.appendChild(oXML.createTextNode(ccCRLF))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
					oEVA.appendChild(oXML.createTextNode(ccTAB))

					oEVA2 = oEVA.appendChild(oXML.createElement("EVA_2"))
					oEVA2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEVA.appendChild(oXML.createTextNode(ccCRLF))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
					oEVA.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oEVA2.TEXT = ALLTRIM(CAST(CAST(oEVA2.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '108' && CONCEPTO DIAN PRIMA SALARIAL
				IF VARTYPE(oEPR) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEPR = oITE.appendChild(oXML.createElement("EPR"))

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))

					oEPR1 = oEPR.appendChild(oXML.createElement("EPR_1"))
					oEPR1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))

					oEPR2 = oEPR.appendChild(oXML.createElement("EPR_2"))
					oEPR2.TEXT = "0"

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oEPR1.TEXT = ALLTRIM(CAST(CAST(oEPR1.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '109' && CONCEPTO DIAN PRIMA NO SALARIAL
				IF VARTYPE(oEPR) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEPR = oITE.appendChild(oXML.createElement("EPR"))

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))

					oEPR1 = oEPR.appendChild(oXML.createElement("EPR_1"))
					oEPR1.TEXT = "0"

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))

					oEPR2 = oEPR.appendChild(oXML.createElement("EPR_2"))
					oEPR2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEPR.appendChild(oXML.createTextNode(ccCRLF))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
					oEPR.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oEPR2.TEXT = ALLTRIM(CAST(CAST(oEPR2.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '110' && CONCEPTO DIAN CESANTIAS
				IF VARTYPE(oECE) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oECE = oITE.appendChild(oXML.createElement("ECE"))

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE1 = oECE.appendChild(oXML.createElement("ECE_1"))
					oECE1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE2 = oECE.appendChild(oXML.createElement("ECE_2"))
					oECE2.TEXT = "1.00"

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE3 = oECE.appendChild(oXML.createElement("ECE_3"))
					oECE3.TEXT = "0.00"

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oECE1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '111' && CONCEPTO DIAN INTERESES SOBRE CESANTIAS
				IF VARTYPE(oECE) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oECE = oITE.appendChild(oXML.createElement("ECE"))

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE1 = oECE.appendChild(oXML.createElement("ECE_1"))
					oECE1.TEXT = "0.00"

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE2 = oECE.appendChild(oXML.createElement("ECE_2"))
					oECE2.TEXT = "1.00"

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))

					oECE3 = oECE.appendChild(oXML.createElement("ECE_3"))
					oECE3.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oECE.appendChild(oXML.createTextNode(ccCRLF))
					oECE.appendChild(oXML.createTextNode(ccTAB))
					oECE.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oECE3.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '112' && CONCEPTO DIAN INCAPACIDADES
				IF VARTYPE(oEIN) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEIN = oITE.appendChild(oXML.createElement("EIN"))

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))

					oEIN1 = oEIN.appendChild(oXML.createElement("EIN_1"))
					oEIN1.TEXT = ""

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))

					oEIN2 = oEIN.appendChild(oXML.createElement("EIN_2"))
					oEIN2.TEXT = ""

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))

					oEIN3 = oEIN.appendChild(oXML.createElement("EIN_3"))
					oEIN3.TEXT = ALLTRIM(CAST(CAST(ABS(S_NROHORAS) AS INT) AS CHARACTER (50)))

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))

					oEIN4 = oEIN.appendChild(oXML.createElement("EIN_4"))
					oEIN4.TEXT = "3"

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))

					oEIN5 = oEIN.appendChild(oXML.createElement("EIN_5"))
					oEIN5.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oEIN.appendChild(oXML.createTextNode(ccCRLF))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
					oEIN.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oEIN5.TEXT = ALLTRIM(CAST(CAST(oEIN5.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '113' && CONCEPTO DIAN LICENCIAS MATERNIDAD - PATERNIDAD
				IF VARTYPE(oELI) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oELI = oITE.appendChild(oXML.createElement("ELI"))

					oELI.appendChild(oXML.createTextNode(ccCRLF))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))

					oELI1 = oELI.appendChild(oXML.createElement("ELI_1"))
					oELI1.TEXT = ""

					oELI.appendChild(oXML.createTextNode(ccCRLF))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))

					oELI2 = oELI.appendChild(oXML.createElement("ELI_2"))
					oELI2.TEXT = ""

					oELI.appendChild(oXML.createTextNode(ccCRLF))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))

					oELI3 = oELI.appendChild(oXML.createElement("ELI_3"))
					oELI3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

					oELI.appendChild(oXML.createTextNode(ccCRLF))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))

					oELI4 = oELI.appendChild(oXML.createElement("ELI_4"))
					oELI4.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oELI.appendChild(oXML.createTextNode(ccCRLF))
					oELI.appendChild(oXML.createTextNode(ccTAB))
					oELI.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oELI4.TEXT = ALLTRIM(CAST(CAST(oELI4.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '114' && CONCEPTO DIAN LICENCIAS REMUNERADAS
				IF VARTYPE(oELR) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oELR = oITE.appendChild(oXML.createElement("ELR"))

					oELR.appendChild(oXML.createTextNode(ccCRLF))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))

					oELR1 = oELR.appendChild(oXML.createElement("ELR_1"))
					oELR1.TEXT = ""

					oELR.appendChild(oXML.createTextNode(ccCRLF))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))

					oELR2 = oELR.appendChild(oXML.createElement("ELR_2"))
					oELR2.TEXT = ""

					oELR.appendChild(oXML.createTextNode(ccCRLF))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))

					oELR3 = oELR.appendChild(oXML.createElement("ELR_3"))
					oELR3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

					oELR.appendChild(oXML.createTextNode(ccCRLF))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))

					oELR4 = oELR.appendChild(oXML.createElement("ELR_4"))
					oELR4.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oELR.appendChild(oXML.createTextNode(ccCRLF))
					oELR.appendChild(oXML.createTextNode(ccTAB))
					oELR.appendChild(oXML.createTextNode(ccTAB))
				ELSE
					oELR4.TEXT = ALLTRIM(CAST(CAST(oELR4.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '115' && CONCEPTO DIAN BONIFICACIONES SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEBN = oITE.appendChild(oXML.createElement("EBN"))

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

				oEBN1 = oEBN.appendChild(oXML.createElement("EBN_1"))
				oEBN1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

				oEBN2 = oEBN.appendChild(oXML.createElement("EBN_2"))
				oEBN2.TEXT = ""

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

			CASE S_CONCEPDIAN = '116' && CONCEPTO DIAN BONIFICACIONES NO SALARIALES

				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEBN = oITE.appendChild(oXML.createElement("EBN"))

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

				oEBN1 = oEBN.appendChild(oXML.createElement("EBN_1"))
				oEBN1.TEXT = ""

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

				oEBN2 = oEBN.appendChild(oXML.createElement("EBN_2"))
				oEBN2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

				oEBN.appendChild(oXML.createTextNode(ccCRLF))
				oEBN.appendChild(oXML.createTextNode(ccTAB))
				oEBN.appendChild(oXML.createTextNode(ccTAB))

*!*				CASE S_CONCEPDIAN = '136' && CONCEPTO DIAN LICENCIAS NO REMUNERADAS
*!*					IF VARTYPE(oELN) = 'U'
*!*						oITE.appendChild(oXML.createTextNode(ccCRLF))
*!*						oITE.appendChild(oXML.createTextNode(ccTAB))
*!*						oITE.appendChild(oXML.createTextNode(ccTAB))

*!*						oELN = oITE.appendChild(oXML.createElement("ELN"))

*!*						oELN.appendChild(oXML.createTextNode(ccCRLF))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))

*!*						oELN1 = oELN.appendChild(oXML.createElement("ELN_1"))
*!*						oELN1.TEXT = ""

*!*						oELN.appendChild(oXML.createTextNode(ccCRLF))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))

*!*						oELN2 = oELN.appendChild(oXML.createElement("ELN_2"))
*!*						oELN2.TEXT = ""

*!*						oELN.appendChild(oXML.createTextNode(ccCRLF))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))

*!*						oELN3 = oELN.appendChild(oXML.createElement("ELN_3"))
*!*						oELN3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

*!*						oELN.appendChild(oXML.createTextNode(ccCRLF))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*						oELN.appendChild(oXML.createTextNode(ccTAB))
*!*					ENDIF

			CASE S_CONCEPDIAN = '117' && CONCEPTO DIAN AUXILIOS SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEAX = oITE.appendChild(oXML.createElement("EAX"))

				oEAX.appendChild(oXML.createTextNode(ccCRLF))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))

				oEAX1 = oEAX.appendChild(oXML.createElement("EAX_1"))
				oEAX1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

				oEAX.appendChild(oXML.createTextNode(ccCRLF))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))

				oEAX2 = oEAX.appendChild(oXML.createElement("EAX_2"))
				oEAX2.TEXT = ""

			CASE S_CONCEPDIAN = '118' && CONCEPTO DIAN AUXILIOS NO SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEAX = oITE.appendChild(oXML.createElement("EAX"))

				oEAX.appendChild(oXML.createTextNode(ccCRLF))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))

				oEAX1 = oEAX.appendChild(oXML.createElement("EAX_1"))
				oEAX1.TEXT = ""

				oEAX.appendChild(oXML.createTextNode(ccCRLF))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))
				oEAX.appendChild(oXML.createTextNode(ccTAB))

				oEAX2 = oEAX.appendChild(oXML.createElement("EAX_2"))
				oEAX2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '119' && CONCEPTO DIAN OTROS CONCEPTOS SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEOT = oITE.appendChild(oXML.createElement("EOT"))

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT1 = oEOT.appendChild(oXML.createElement("EOT_1"))
				oEOT1.TEXT = ALLTRIM(S_OFIMA_DESCRIPCION)

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT2 = oEOT.appendChild(oXML.createElement("EOT_2"))
				oEOT2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT3 = oEOT.appendChild(oXML.createElement("EOT_3"))
				oEOT3.TEXT = ""

			CASE S_CONCEPDIAN = '120' && CONCEPTO DIAN OTROS CONCEPTOS NO SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEOT = oITE.appendChild(oXML.createElement("EOT"))

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT1 = oEOT.appendChild(oXML.createElement("EOT_1"))
				oEOT1.TEXT = ALLTRIM(S_OFIMA_DESCRIPCION)

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT2 = oEOT.appendChild(oXML.createElement("EOT_2"))
				oEOT2.TEXT = ""

				oEOT.appendChild(oXML.createTextNode(ccCRLF))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))
				oEOT.appendChild(oXML.createTextNode(ccTAB))

				oEOT3 = oEOT.appendChild(oXML.createElement("EOT_3"))
				oEOT3.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '121' && CONCEPTO DIAN COMPENSACIONES ORDINARIAS
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oECM = oITE.appendChild(oXML.createElement("ECM"))

				oECM.appendChild(oXML.createTextNode(ccCRLF))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))

				oECM1 = oECM.appendChild(oXML.createElement("ECM_1"))
				oECM1.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

				oECM.appendChild(oXML.createTextNode(ccCRLF))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))

				oECM2 = oECM.appendChild(oXML.createElement("ECM_2"))
				oECM2.TEXT = ""

			CASE S_CONCEPDIAN = '122' && CONCEPTO DIAN COMPENSACIONES EXTRAORDINARIAS
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oECM = oITE.appendChild(oXML.createElement("ECM"))

				oECM.appendChild(oXML.createTextNode(ccCRLF))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))

				oECM1 = oECM.appendChild(oXML.createElement("ECM_1"))
				oECM1.TEXT = ""

				oECM.appendChild(oXML.createTextNode(ccCRLF))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))
				oECM.appendChild(oXML.createTextNode(ccTAB))

				oECM2 = oECM.appendChild(oXML.createElement("ECM_2"))
				oECM2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
				
				
			CASE S_CONCEPDIAN = '123' && CONCEPTO DIAN BONOS PAGO SALARIALES
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEBO = oITE.appendChild(oXML.createElement("EBO"))

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO1 = oEBO.appendChild(oXML.createElement("EBO_1"))
					oEBO1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO2 = oEBO.appendChild(oXML.createElement("EBO_2"))
					oEBO2.TEXT = ""

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO3 = oEBO.appendChild(oXML.createElement("EBO_3"))
					oEBO3.TEXT = ""

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO4 = oEBO.appendChild(oXML.createElement("EBO_4"))
					oEBO4.TEXT = ""

			CASE S_CONCEPDIAN = '124' && CONCEPTO DIAN BONOS PAGO NO SALARIALES
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEBO = oITE.appendChild(oXML.createElement("EBO"))

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO1 = oEBO.appendChild(oXML.createElement("EBO_1"))
					oEBO1.TEXT = ""

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO2 = oEBO.appendChild(oXML.createElement("EBO_2"))
					oEBO2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO3 = oEBO.appendChild(oXML.createElement("EBO_3"))
					oEBO3.TEXT = ""

					oEBO.appendChild(oXML.createTextNode(ccCRLF))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))
					oEBO.appendChild(oXML.createTextNode(ccTAB))

					oEBO4 = oEBO.appendChild(oXML.createElement("EBO_4"))
					oEBO4.TEXT = ""
					
			CASE S_CONCEPDIAN = '125' && CONCEPTO DIAN BONOS ALIMENTACION SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEBO = oITE.appendChild(oXML.createElement("EBO"))

				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO1 = oEBO.appendChild(oXML.createElement("EBO_1"))
				oEBO1.TEXT = ""

				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO2 = oEBO.appendChild(oXML.createElement("EBO_2"))
				oEBO2.TEXT = ""
				
				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO3 = oEBO.appendChild(oXML.createElement("EBO_3"))
				oEBO3.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
				
				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO4 = oEBO.appendChild(oXML.createElement("EBO_4"))
				oEBO4.TEXT = ""
				
			CASE S_CONCEPDIAN = '126' && CONCEPTO DIAN BONOS ALIMENTACION NO SALARIALES
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oEBO = oITE.appendChild(oXML.createElement("EBO"))

				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO1 = oEBO.appendChild(oXML.createElement("EBO_1"))
				oEBO1.TEXT = ""

				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO2 = oEBO.appendChild(oXML.createElement("EBO_2"))
				oEBO2.TEXT = ""
				
				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO3 = oEBO.appendChild(oXML.createElement("EBO_3"))
				oEBO3.TEXT = ""
				
				oEBO.appendChild(oXML.createTextNode(ccCRLF))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))
				oEBO.appendChild(oXML.createTextNode(ccTAB))

				oEBO4 = oEBO.appendChild(oXML.createElement("EBO_4"))
				oEBO4.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
				
			CASE S_CONCEPDIAN = '127' && CONCEPTO DIAN COMISION
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oECO = oITE.appendChild(oXML.createElement("ECO"))

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO1 = oECO.appendChild(oXML.createElement("ECO_1"))
				oECO1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO2 = oECO.appendChild(oXML.createElement("ECO_2"))
				oECO2.TEXT = ""

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO3 = oECO.appendChild(oXML.createElement("ECO_3"))
				oECO3.TEXT = ""


			CASE S_CONCEPDIAN = '128' && CONCEPTO DIAN PAGO TERCEROS
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oECO = oITE.appendChild(oXML.createElement("ECO"))

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO1 = oECO.appendChild(oXML.createElement("ECO_1"))
				oECO1.TEXT = ""

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO2 = oECO.appendChild(oXML.createElement("ECO_2"))
				oECO2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO3 = oECO.appendChild(oXML.createElement("ECO_3"))
				oECO3.TEXT = ""

			CASE S_CONCEPDIAN = '129' && CONCEPTO DIAN ANTICIPO
				oITE.appendChild(oXML.createTextNode(ccCRLF))
				oITE.appendChild(oXML.createTextNode(ccTAB))
				oITE.appendChild(oXML.createTextNode(ccTAB))

				oECO = oITE.appendChild(oXML.createElement("ECO"))

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO1 = oECO.appendChild(oXML.createElement("ECO_1"))
				oECO1.TEXT = ""

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO2 = oECO.appendChild(oXML.createElement("ECO_2"))
				oECO2.TEXT = ""

				oECO.appendChild(oXML.createTextNode(ccCRLF))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))
				oECO.appendChild(oXML.createTextNode(ccTAB))

				oECO3 = oECO.appendChild(oXML.createElement("ECO_3"))
				oECO3.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))
				
			CASE S_CONCEPDIAN = '130' && CONCEPTO DIAN DOTACIONES
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = "0.00"
				ELSE
					oEVO1.TEXT = ALLTRIM(CAST(CAST(oEVO1.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF
			

			CASE S_CONCEPDIAN = '131' && CONCEPTO DIAN APOYO SOSTENIMIENTO
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = ""

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = ALLTRIM(CAST(S_VALOR AS CHARACTER (50)))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = "0.00"
				ELSE
					oEVO2.TEXT = ALLTRIM(CAST(CAST(oEVO2.TEXT AS NUMERIC (10,2)) + S_VALOR AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '132' && CONCEPTO DIAN APOYO SOSTENIMIENTO
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = "0.00"
				ELSE
					oEVO3.TEXT = ALLTRIM(CAST(CAST(oEVO3.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
				
			CASE S_CONCEPDIAN = '133' && CONCEPTO DIAN BONIFICACION RETIRO
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = "0.00"
				ELSE
					oEVO4.TEXT = ALLTRIM(CAST(CAST(oEVO4.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
				
			CASE S_CONCEPDIAN = '134' && CONCEPTO DIAN INDEMNIZACION
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = "0.00"
				ELSE
					oEVO5.TEXT = ALLTRIM(CAST(CAST(oEVO5.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
				
			
			CASE S_CONCEPDIAN = '135' && CONCEPTO DIAN REINTEGRO
				IF VARTYPE(oEVO) = 'U'
					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oEVO = oITE.appendChild(oXML.createElement("EVO"))

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO1 = oEVO.appendChild(oXML.createElement("EVO_1"))
					oEVO1.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO2 = oEVO.appendChild(oXML.createElement("EVO_2"))
					oEVO2.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO3 = oEVO.appendChild(oXML.createElement("EVO_3"))
					oEVO3.TEXT = "0.00"

					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO4 = oEVO.appendChild(oXML.createElement("EVO_4"))
					oEVO4.TEXT = "0.00"
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO5 = oEVO.appendChild(oXML.createElement("EVO_5"))
					oEVO5.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
					
					
					oEVO.appendChild(oXML.createTextNode(ccCRLF))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))
					oEVO.appendChild(oXML.createTextNode(ccTAB))

					oEVO6 = oEVO.appendChild(oXML.createElement("EVO_6"))
					oEVO6.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
				ELSE
					oEVO6.TEXT = ALLTRIM(CAST(CAST(oEVO6.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF


			CASE S_CONCEPDIAN = '203' && CONCEPTO DIAN SOLIDARIDAD
				IF VARTYPE(oSSP) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSSP = oITS.appendChild(oXML.createElement("SSP"))

					oSSP.appendChild(oXML.createTextNode(ccCRLF))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))

					oSSP1 = oSSP.appendChild(oXML.createElement("SSP_1"))
					oSSP1.TEXT = ""

					oSSP.appendChild(oXML.createTextNode(ccCRLF))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))

					oSSP2 = oSSP.appendChild(oXML.createElement("SSP_2"))
					oSSP2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSSP.appendChild(oXML.createTextNode(ccCRLF))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))

					oSSP3 = oSSP.appendChild(oXML.createElement("SSP_3"))
					oSSP3.TEXT = ""

					oSSP.appendChild(oXML.createTextNode(ccCRLF))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))
					oSSP.appendChild(oXML.createTextNode(ccTAB))

					oSSP4 = oSSP.appendChild(oXML.createElement("SSP_4"))
					oSSP4.TEXT = ""
				ELSE
					oSSP2.TEXT = ALLTRIM(CAST(CAST(oSSP2.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
				
			CASE S_CONCEPDIAN = '204' && CONCEPTO DIAN SINDICATO
			
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSIN = oITS.appendChild(oXML.createElement("SIN"))

					oSIN.appendChild(oXML.createTextNode(ccCRLF))
					oSIN.appendChild(oXML.createTextNode(ccTAB))
					oSIN.appendChild(oXML.createTextNode(ccTAB))
					oSIN.appendChild(oXML.createTextNode(ccTAB))

					oSIN1 = oSIN.appendChild(oXML.createElement("SIN_1"))
					oSIN1.TEXT = "1"

					oSIN.appendChild(oXML.createTextNode(ccCRLF))
					oSIN.appendChild(oXML.createTextNode(ccTAB))
					oSIN.appendChild(oXML.createTextNode(ccTAB))
					oSIN.appendChild(oXML.createTextNode(ccTAB))

					oSIN2 = oSIN.appendChild(oXML.createElement("SIN_2"))
					oSIN2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
					
			CASE S_CONCEPDIAN = '205' && CONCEPTO DIAN SANCIONES PUBLICAS
			
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSAN = oITS.appendChild(oXML.createElement("SAN"))

					oSAN.appendChild(oXML.createTextNode(ccCRLF))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))

					oSAN1 = oSAN.appendChild(oXML.createElement("SAN_1"))
					oSAN1.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSAN.appendChild(oXML.createTextNode(ccCRLF))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))

					oSAN2 = oSAN.appendChild(oXML.createElement("SAN_2"))
					oSAN2.TEXT = ""
					
			CASE S_CONCEPDIAN = '206' && CONCEPTO DIAN SANCIONES PRIVADAS
			
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSAN = oITS.appendChild(oXML.createElement("SAN"))

					oSAN.appendChild(oXML.createTextNode(ccCRLF))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))

					oSAN1 = oSAN.appendChild(oXML.createElement("SAN_1"))
					oSAN1.TEXT = ""

					oSAN.appendChild(oXML.createTextNode(ccCRLF))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))
					oSAN.appendChild(oXML.createTextNode(ccTAB))

					oSAN2 = oSAN.appendChild(oXML.createElement("SAN_2"))
					oSAN2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
					
			CASE S_CONCEPDIAN = '207' && CONCEPTO DIAN LIBRANZAS
				oITS.appendChild(oXML.createTextNode(ccCRLF))
				oITS.appendChild(oXML.createTextNode(ccTAB))
				oITS.appendChild(oXML.createTextNode(ccTAB))

				oSLI = oITS.appendChild(oXML.createElement("SLI"))

				oSLI.appendChild(oXML.createTextNode(ccCRLF))
				oSLI.appendChild(oXML.createTextNode(ccTAB))
				oSLI.appendChild(oXML.createTextNode(ccTAB))
				oSLI.appendChild(oXML.createTextNode(ccTAB))

				oSLI1 = oSLI.appendChild(oXML.createElement("SLI_1"))
				oSLI1.TEXT = ALLTRIM(S_OFIMA_DESCRIPCION)

				oSLI.appendChild(oXML.createTextNode(ccCRLF))
				oSLI.appendChild(oXML.createTextNode(ccTAB))
				oSLI.appendChild(oXML.createTextNode(ccTAB))
				oSLI.appendChild(oXML.createTextNode(ccTAB))

				oSLI2 = oSLI.appendChild(oXML.createElement("SLI_2"))
				oSLI2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '208' && CONCEPTO DIAN PAGO A TERCEROS
				oITS.appendChild(oXML.createTextNode(ccCRLF))
				oITS.appendChild(oXML.createTextNode(ccTAB))
				oITS.appendChild(oXML.createTextNode(ccTAB))

				oSOT = oITS.appendChild(oXML.createElement("SOT"))

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT1 = oSOT.appendChild(oXML.createElement("SOT_1"))
				oSOT1.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT2 = oSOT.appendChild(oXML.createElement("SOT_2"))
				oSOT2.TEXT = ""

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT3 = oSOT.appendChild(oXML.createElement("SOT_3"))
				oSOT3.TEXT = ""
				
			CASE S_CONCEPDIAN = '209' && CONCEPTO DIAN ANTICIPOS
				oITS.appendChild(oXML.createTextNode(ccCRLF))
				oITS.appendChild(oXML.createTextNode(ccTAB))
				oITS.appendChild(oXML.createTextNode(ccTAB))

				oSOT = oITS.appendChild(oXML.createElement("SOT"))

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT1 = oSOT.appendChild(oXML.createElement("SOT_1"))
				oSOT1.TEXT = ""
				
				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT2 = oSOT.appendChild(oXML.createElement("SOT_2"))
				oSOT2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT3 = oSOT.appendChild(oXML.createElement("SOT_3"))
				oSOT3.TEXT = ""

			CASE S_CONCEPDIAN = '210' && CONCEPTO DIAN OTRAS DEDUCCIONES
				IF S_NUMCONCEP = 'LICNO'

					oITE.appendChild(oXML.createTextNode(ccCRLF))
					oITE.appendChild(oXML.createTextNode(ccTAB))
					oITE.appendChild(oXML.createTextNode(ccTAB))

					oELN = oITE.appendChild(oXML.createElement("ELN"))

					oELN.appendChild(oXML.createTextNode(ccCRLF))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))

					oELN1 = oELN.appendChild(oXML.createElement("ELN_1"))
					oELN1.TEXT = ""

					oELN.appendChild(oXML.createTextNode(ccCRLF))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))

					oELN2 = oELN.appendChild(oXML.createElement("ELN_2"))
					oELN2.TEXT = ""

					oELN.appendChild(oXML.createTextNode(ccCRLF))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))

					oELN3 = oELN.appendChild(oXML.createElement("ELN_3"))
					oELN3.TEXT = ALLTRIM(CAST(CAST(S_NROHORAS AS INT) AS CHARACTER (50)))

					oELN.appendChild(oXML.createTextNode(ccCRLF))
					oELN.appendChild(oXML.createTextNode(ccTAB))
					oELN.appendChild(oXML.createTextNode(ccTAB))

				ENDIF

				oITS.appendChild(oXML.createTextNode(ccCRLF))
				oITS.appendChild(oXML.createTextNode(ccTAB))
				oITS.appendChild(oXML.createTextNode(ccTAB))

				oSOT = oITS.appendChild(oXML.createElement("SOT"))

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT1 = oSOT.appendChild(oXML.createElement("SOT_1"))
				oSOT1.TEXT = ""

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT2 = oSOT.appendChild(oXML.createElement("SOT_2"))
				oSOT2.TEXT = ""

				oSOT.appendChild(oXML.createTextNode(ccCRLF))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))
				oSOT.appendChild(oXML.createTextNode(ccTAB))

				oSOT3 = oSOT.appendChild(oXML.createElement("SOT_3"))
				oSOT3.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

			CASE S_CONCEPDIAN = '211' && CONCEPTO DIAN PENSIÓN VOLUNTARIA
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA1.TEXT)
						oSVA1.TEXT = "0.00"
					ENDIF
					oSVA1.TEXT = ALLTRIM(CAST(CAST(oSVA1.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '212' && CONCEPTO DIAN RETENCIÓN EN LA FUENTE
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA2.TEXT)
						oSVA1.TEXT = "0.00"
					ENDIF
					oSVA2.TEXT = ALLTRIM(CAST(CAST(oSVA2.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '213' && CONCEPTO DIAN AFC
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA3.TEXT)
						oSVA3.TEXT = "0.00"
					ENDIF
					oSVA3.TEXT = ALLTRIM(CAST(CAST(oSVA3.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '215' && CONCEPTO DIAN EMBARGO FISCAL
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA5.TEXT)
						oSVA5.TEXT = "0.00"
					ENDIF
					oSVA5.TEXT = ALLTRIM(CAST(CAST(oSVA5.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '216' && CONCEPTO DIAN PLAN COMPLEMENTARIOS
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA6.TEXT)
						oSVA6.TEXT = "0.00"
					ENDIF
					oSVA6.TEXT = ALLTRIM(CAST(CAST(oSVA6.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
			
			CASE S_CONCEPDIAN = '217' && CONCEPTO DIAN EDUCACION
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA7.TEXT)
						oSVA7.TEXT = "0.00"
					ENDIF
					oSVA7.TEXT = ALLTRIM(CAST(CAST(oSVA7.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF
				
			CASE S_CONCEPDIAN = '218' && CONCEPTO DIAN REINTEGRO
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ""
				ELSE
					IF EMPTY(oSVA8.TEXT)
						oSVA8.TEXT = "0.00"
					ENDIF
					oSVA8.TEXT = ALLTRIM(CAST(CAST(oSVA8.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			CASE S_CONCEPDIAN = '219' && CONCEPTO DIAN DEUDA
				IF VARTYPE(oSVA) = 'U'
					oITS.appendChild(oXML.createTextNode(ccCRLF))
					oITS.appendChild(oXML.createTextNode(ccTAB))
					oITS.appendChild(oXML.createTextNode(ccTAB))

					oSVA = oITS.appendChild(oXML.createElement("SVA"))

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA1 = oSVA.appendChild(oXML.createElement("SVA_1"))
					oSVA1.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA2 = oSVA.appendChild(oXML.createElement("SVA_2"))
					oSVA2.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA3 = oSVA.appendChild(oXML.createElement("SVA_3"))
					oSVA3.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA4 = oSVA.appendChild(oXML.createElement("SVA_4"))
					oSVA4.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA5 = oSVA.appendChild(oXML.createElement("SVA_5"))
					oSVA5.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA6 = oSVA.appendChild(oXML.createElement("SVA_6"))
					oSVA6.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA7 = oSVA.appendChild(oXML.createElement("SVA_7"))
					oSVA7.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA8 = oSVA.appendChild(oXML.createElement("SVA_8"))
					oSVA8.TEXT = ""

					oSVA.appendChild(oXML.createTextNode(ccCRLF))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))
					oSVA.appendChild(oXML.createTextNode(ccTAB))

					oSVA9 = oSVA.appendChild(oXML.createElement("SVA_9"))
					oSVA9.TEXT = ALLTRIM(CAST(ABS(S_VALOR) AS CHARACTER (50)))
				ELSE
					IF EMPTY(oSVA9.TEXT)
						oSVA9.TEXT = "0.00"
					ENDIF
					oSVA9.TEXT = ALLTRIM(CAST(CAST(oSVA9.TEXT AS NUMERIC (10,2)) + ABS(S_VALOR) AS CHARACTER (50)))
				ENDIF

			ENDCASE
		ENDIF
	ENDSCAN

	oNomina.appendChild(oXML.createTextNode(ccCRLF))

	oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccTAB))

&&oNomina.appendChild(oXML.createTextNode(ccCRLF))
	oTOT = oNomina.appendChild(oXML.createElement("TOT"))

	SELECT C_EMPLEADO && Almacenando Devengados, Deducciones y Totales en el XML, etiquetas finales.

	oTOT.appendChild(oXML.createTextNode(ccCRLF))
	oTOT.appendChild(oXML.createTextNode(ccTAB))
	oTOT.appendChild(oXML.createTextNode(ccTAB))

	oTOT1 = oTOT.appendChild(oXML.createElement("TOT_1"))
	oTOT1.TEXT = "2"

	oTOT.appendChild(oXML.createTextNode(ccCRLF))
	oTOT.appendChild(oXML.createTextNode(ccTAB))
	oTOT.appendChild(oXML.createTextNode(ccTAB))

	oTOT2 = oTOT.appendChild(oXML.createElement("TOT_2"))
	oTOT2.TEXT = ALLTRIM(CAST(S_DEVENGADOS AS CHARACTER (50)))

	oTOT.appendChild(oXML.createTextNode(ccCRLF))
	oTOT.appendChild(oXML.createTextNode(ccTAB))
	oTOT.appendChild(oXML.createTextNode(ccTAB))

	oTOT3 = oTOT.appendChild(oXML.createElement("TOT_3"))
	oTOT3.TEXT = ALLTRIM(CAST(S_DEDUCCIONES AS CHARACTER (50)))

	oTOT.appendChild(oXML.createTextNode(ccCRLF))
	oTOT.appendChild(oXML.createTextNode(ccTAB))
	oTOT.appendChild(oXML.createTextNode(ccTAB))

	oTOT4 = oTOT.appendChild(oXML.createElement("TOT_4"))
	oTOT4.TEXT = ALLTRIM(CAST(S_TOTAL AS CHARACTER (50)))


	oTOT.appendChild(oXML.createTextNode(ccCRLF))
	oTOT.appendChild(oXML.createTextNode(ccTAB))

	oITE.appendChild(oXML.createTextNode(ccCRLF))
	oNomina.appendChild(oXML.createTextNode(ccCRLF))


	oXML.SAVE(pathSalida)
	returnBase64 = STRCONV(FILETOSTR(pathSalida), 13)
*_CLIPTEXT = returnBase64 -- Guarda el base64 en el portapapeles para poder comprobar si quedó bien generado.

CATCH TO IOEXCEPTION
	MESSAGEBOX("Error: " + IOEXCEPTION.MESSAGE + CHR(13) + CHR(13) + "Código de error: " + TRANSFORM(IOEXCEPTION.LINENO) + CHR(13) + CHR(13) + "Código que produjo el error: " + MESSAGE(1), 48)
ENDTRY

RETURN returnBase64
ENDFUNC

*---------------------------------------------------

FUNCTION generarUUID && Función que genera una cadena UUID/GUID, etiqueta NONCE del Header de seguridad de conexión para el WebService. Utilizado para la unicidad de cada petición.
LOCAL UUID
STORE "" TO UUID

LOCAL lcRetval, lcStruc_GUID, lcGUID, lnSize
DECLARE INTEGER CoCreateGuid IN "ole32.dll" STRING @lcGUIDStruc
DECLARE INTEGER StringFromGUID2 IN "ole32.dll" STRING cGUIDStruc, STRING @cGUID, LONG nSize
lcStruc_GUID = REPLICATE(" ",16)
lcGUID = REPLICATE(" ",80)
lnSize = LEN(lcGUID) / 2
IF CoCreateGuid(@lcStruc_GUID) <> 0
	RETURN ""
ENDIF
IF StringFromGUID2(lcStruc_GUID,@lcGUID,lnSize) = 0
	RETURN ""
ENDIF
RETURN STREXTR(STRCONV(lcGUID,6),"{","}") && >= VFP7
*RETURN SUBSTR(STRCONV(lcGUID,6),2,36) && < VFP7 Si se tiene un VFP en versión 7 o posterior, se debe utilizar esta línea para el formateo del UUID.

RETURN UUID

ENDFUNC

*---------------------------------------------------

FUNCTION FECHAACTUAL && Retorna la fecha actual formateada, también utilizada para una etiqueta del XML y Header de seguridad del Request.
LPARAMETERS OPCION && 1 FECHA Y HORA CON Z - 2 HORA.
IF OPCION = 1
	RETURN TTOC(DATETIME(), 3) + ".0" + SUBSTR(TIME(DATETIME()), 10, 2) + "-05:00"
ELSE
	RETURN TRANSFORM(TIME()) + "-05:00"
ENDIF
ENDFUNC

*---------------------------------------------------

FUNCTION generarHeaderSeguridad && Función que genera y retorna el Header de seguridad para la conexión al Web Service (header del request).
LPARAMETERS USUARIO, CONTRASENA && Recibe el usuario y la contraseña, credenciales otorgadas por el proveedor tecnológico Carvajal.

STORE "" TO XMLheader
UUIDBASE64 = STRCONV(generarUUID(), 13)

TEXT TO XMLheader TEXTMERGE PRETEXT 7 NOSHOW
				<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:inv="http://www.ebussines.com.co/foundation/il/contracts/document/model" xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
					<soapenv:Header>
					<wsse:Security>
						<wsse:UsernameToken wsu:Id="UsernameToken-1">
							<wsse:Username><<ALLTRIM(USUARIO)>></wsse:Username>
							<wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText"><<ALLTRIM(CONTRASENA)>></wsse:Password>
							<wsse:Nonce EncodingType="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-soap-message-security-1.0#Base64Binary"><<UUIDBASE64>></wsse:Nonce>
							<wsu:Created><<fechaActual(1)>></wsu:Created>
						</wsse:UsernameToken>
					</wsse:Security>
				   </soapenv:Header>
				   <<ccCRLF>>
ENDTEXT

RETURN XMLheader

ENDFUNC

*---------------------------------------------------

FUNCTION crearRequestDownload && Función que crea el request para consumir el método DocumentStatus el cual recibe un Transaction ID y retorna el estado de un documento
&& de nómina existente en el CEN Financiero.

LPARAMETERS USUARIO, CONTRASENA, COMPANYID, ACCOUNTID, TIPODCTO, NRODCTO

LOCAL XMLrequest, UUIDBASE64
STORE "" TO XMLrequest
&&MESSAGEBOX("TRANSACID RECIBIDO:" + TRANSACID)

XMLrequest = generarHeaderSeguridad(USUARIO, CONTRASENA)

TEXT TO XMLrequest ADDITIVE TEXTMERGE PRETEXT 7 NOSHOW

				<soapenv:Body>
					<inv:DownloadRequest>
			         <inv:companyId><<ALLTRIM(COMPANYID)>></inv:companyId>
			         <inv:accountId><<ALLTRIM(ACCOUNTID)>></inv:accountId>
			         <inv:documentType><<"NM">></inv:documentType>
			         <inv:documentNumber><<ALLTRIM(TIPODCTO)+ALLTRIM(NRODCTO)>></inv:documentNumber>
			         <inv:resourceType>DIAN_RESULT</inv:resourceType>
			         <inv:service>PAYROLL</inv:service>
					</inv:DownloadRequest>
				</soapenv:Body>
				</soapenv:Envelope>

ENDTEXT
_CLIPTEXT = XMLrequest
&&MESSAGEBOX("COPIAR")
RETURN XMLrequest

ENDFUNC

*---------------------------------------------------

FUNCTION crearRequestDocumentStatus && Función que crea el request para consumir el método DocumentStatus el cual recibe un Transaction ID y retorna el estado de un documento
&& de nómina existente en el CEN Financiero.

LPARAMETERS USUARIO, CONTRASENA, COMPANYID, ACCOUNTID, TRANSACID

LOCAL XMLrequest, UUIDBASE64
STORE "" TO XMLrequest
&&MESSAGEBOX("TRANSACID RECIBIDO:" + TRANSACID)

XMLrequest = generarHeaderSeguridad(USUARIO, CONTRASENA)

TEXT TO XMLrequest ADDITIVE TEXTMERGE PRETEXT 7 NOSHOW

				<soapenv:Body>
					<inv:DocumentStatusRequest>
						<inv:transactionId><<TRANSACID>></inv:transactionId>
						<inv:companyId><<ALLTRIM(COMPANYID)>></inv:companyId>
						<inv:accountId><<ALLTRIM(ACCOUNTID)>></inv:accountId>
						<inv:service>PAYROLL</inv:service>
					</inv:DocumentStatusRequest>
				</soapenv:Body>
				</soapenv:Envelope>

ENDTEXT
_CLIPTEXT = XMLrequest
&&MESSAGEBOX("COPIAR")
RETURN XMLrequest

ENDFUNC

*---------------------------------------------------

FUNCTION crearRequestUpload && Función que crea y retorna el request para consumir el método UPLOAD del WS. Usado para subir un documento de nómina individual o de ajuste.
LPARAMETERS USUARIO, CONTRASENA, COMPANYID, ACCOUNTID, MEDIOBASE64, NOMBREARCHIVO

LOCAL XMLrequest, UUIDBASE64
STORE "" TO XMLrequest

XMLrequest = generarHeaderSeguridad(USUARIO, CONTRASENA)

TEXT TO XMLrequest ADDITIVE TEXTMERGE PRETEXT 7 NOSHOW

				<soapenv:Body>
					<inv:UploadRequest>
						<inv:fileName><<ALLTRIM(NOMBREARCHIVO)>></inv:fileName>
						<inv:fileData><<MEDIOBASE64>></inv:fileData>
						<inv:companyId><<ALLTRIM(COMPANYID)>></inv:companyId>
						<inv:accountId><<ALLTRIM(ACCOUNTID)>></inv:accountId>
						<inv:service>PAYROLL</inv:service>
					</inv:UploadRequest>
				</soapenv:Body>
				</soapenv:Envelope>

ENDTEXT

_CLIPTEXT = XMLrequest

RETURN XMLrequest

ENDFUNC

*---------------------------------------------------

FUNCTION almacenarTransacID && Almacena en la BD, tabla MTLIQNOMNE campo NECUNE, el TRANSACTIONID de un documento de nómina recibido por Carvajal.
LPARAMETERS TIPODCTO, NRODCTO, TRANSACTIONID, ON

SQLUPDATE = "UPDATE MTLIQNOMNE SET NETRANSACID = '" + TRANSACTIONID + "' WHERE TIPODCTO = '" + TIPODCTO + "' AND NRODCTO = '" + NRODCTO + "'"
IF SQLEXEC(ON, SQLUPDATE) = -1
	MESSAGEBOX("Error al almacenar el Transaction ID en MTLIQNOMNE.")
ENDIF

ENDFUNC

*---------------------------------------------------

FUNCTION almacenarCuneDIAN && Almacena en la BD, tabla MTLIQNOMNE campo NECUNE, el TRANSACTIONID de un documento de nómina recibido por Carvajal.
LPARAMETERS TIPODCTO, NRODCTO, DIANCUNESHA256, ON

SQLUPDATE = "UPDATE MTLIQNOMNE SET NECUNE = '" + DIANCUNESHA256 + "' WHERE TIPODCTO = '" + TIPODCTO + "' AND NRODCTO = '" + NRODCTO + "'"
IF SQLEXEC(ON, SQLUPDATE) = -1
	MESSAGEBOX("Error al almacenar el CUNE en MTLIQNOMNE.")
ENDIF

ENDFUNC

*---------------------------------------------------

FUNCTION capturarError
LPARAMETERS msjError, pathSalida, TIPODCTO, NRODCTO, METODO

STORE "" TO _error

IF DIRECTORY(CURRENTPATH + "NOMINA_ELECTRONICA\") = .F.
	MKDIR CURRENTPATH + "\NOMINA_ELECTRONICA"
ENDIF

pathSalida = pathSalida + "NOMINA_ELECTRONICA\LOG-ERRORES-NE.txt"

_error = "Fecha: " + FECHAACTUAL(1) + CHR(13)
_error = _error + "Tipo documento: " + TIPODCTO + CHR(13)
_error = _error + "Numero de documento: " + NRODCTO + CHR(13)
_error = _error + "Mensaje de error: " + msjError + CHR(13)
_error = _error + "Metodo: " + ALLTRIM(STR(METODO)) + CHR(13) + CHR(13)

STRTOFILE(_error, pathSalida, .T.)

ENDFUNC


*---------------------------------------------------

FUNCTION validarEnvio && Retorna el transaction id o ERROR si algo falló
LPARAMETERS XMLresponse, pathSalida, TIPODCTO, NRODCTO, METODO && METODO = 1 ES UPLOAD, OPCIÓN = 2 ES DOCUMENTSTATUS, OPCIÓN = 3 ES DOWNLOAD

STORE "" TO msjError
STORE "" TO filtro
STORE "" TO TRANSACID
STORE "" TO DIAN_RESPONSE
STORE "" TO XmlDocumentKey

msjError = STREXTRACT(XMLresponse, "<errorMessage>","</errorMessage>")

IF !EMPTY(msjError)
	capturarError(msjError, pathSalida, TIPODCTO, NRODCTO, METODO)
	RETURN "ERROR"
ENDIF

DO CASE
CASE METODO = 1 && Para el caso de la subida del XML, sólamente se puede generar el errorMessage. De lo contrario, se captura el Transaction ID.

	TRANSACID = STREXTRACT(XMLresponse, "<transactionId>","</transactionId>")
	RETURN TRANSACID

CASE METODO = 2 && Para el caso de la consulta del estado del documento, se puede generar el el error en 'errorMessage' cuando es error interno del servidor, se debe capturar
&& o también puede generar un 'De' que es error en algunos campos del documento. También se debe capturar.
&&MESSAGEBOX(XMLresponse)
	filtro = STREXTRACT(XMLresponse, "<De>", "</De>")
&&			MESSAGEBOX(filtro)

	IF filtro = "Documento Firmado"
&& Consultar el CUNE aca.
		RETURN "T"
	ENDIF

	IF !EMPTY(filtro)
		capturarError(msjError, pathSalida, TIPODCTO, NRODCTO, METODO)
		RETURN "ERROR"
	ENDIF

CASE METODO = 3 && DOWNLOAD
	filtro = STREXTRACT(XMLresponse, "<downloadData>", "</downloadData>")
	DIAN_RESPONSE = STRCONV(filtro, 14)
	filtro = ""
	filtro = STREXTRACT(DIAN_RESPONSE, "<b:ErrorMessage>", "</b:ErrorMessage>")
*			MESSAGEBOX("DIANRESPONSE:" + DIAN_RESPONSE)

	IF !EMPTY(filtro)
		filtro = ALLTRIM(filtro)
		capturarError(filtro, pathSalida, TIPODCTO, NRODCTO, METODO)
		RETURN "ERROR"
	ENDIF

	XmlDocumentKey = STREXTRACT(DIAN_RESPONSE, "<b:XmlDocumentKey>", "</b:XmlDocumentKey>")

	RETURN XmlDocumentKey
ENDCASE

ENDFUNC

*---------------------------------------------------


FUNCTION enviarRequest && Envia el request mediante protocolo HTTP por medio de un objeto MICROSOFT.XMLHTTP. Se debe usar este por ser
&& HTTPS el endpoint de carvajal.

LPARAMETERS XMLrequest, pathSalida, TIPODCTO, NRODCTO, METODO, DETALLEWEBSERVICE && METODO = 1 ES UPLOAD, METODO = 2 ES DOCUMENTSTATUS, METODO = 3 ES DOWNLOAD

LOCAL PRODUCCION AS STRING
LOCAL PRUEBAS AS STRING
STORE "https://feco-prod-servicescomunication.cen.biz/webservice/" TO PRODUCCION
STORE "https://feco-stage-nominaelect-apig.facturacarvajal.com/webservice/" TO PRUEBAS
STORE "" TO VALIDAR

TRY
	oHTTP = CREATEOBJECT("Microsoft.XMLHTTP")
	oHTTP.OPEN("POST", PRODUCCION, .F.)
	oHTTP.setRequestHeader("Content-Type", "text/xml")
	oHTTP.setRequestHeader("Content-Type", "application/xml")
	oHTTP.setRequestHeader("Content-Length", LEN(XMLrequest))
	oHTTP.SEND(XMLrequest)

	IF (DETALLEWEBSERVICE = 1)
		_CLIPTEXT = XMLrequest
		MESSAGEBOX(XMLrequest)
	ENDIF

CATCH TO IOEXCEPTION

	MESSAGEBOX("Error de conexion con el WS: " + IOEXCEPTION.MESSAGE)

ENDTRY

*

IF oHTTP.STATUS = 200
	IF (DETALLEWEBSERVICE = 1)
		_CLIPTEXT = oHTTP.responseText
		MESSAGEBOX(oHTTP.responseText)
	ENDIF
&& Capturar errores o llamar a la consulta del CUNE.
	VALIDAR = validarEnvio(oHTTP.responseText, pathSalida, TIPODCTO, NRODCTO, METODO)

	IF VALIDAR = "ERROR"
		RETURN ""
	ENDIF

	DO CASE
	CASE METODO = 1
		RETURN VALIDAR && Si se subió el dcto y se retorna el transactionid

	CASE METODO = 2
		RETURN "T" && Si se estaba consumiendo el método de DocumentStatus, y pasa sin errores, retorna T.

	CASE METODO = 3
		RETURN VALIDAR && RETORNA CUNE

	ENDCASE

ELSE
	MESSAGEBOX("Error de conexion con el Web Service." + CHR(13) + CHR(13) + oHTTP.responseText + CHR(13) + CHR(13) + "Status: " + ALLTRIM(STR(oHTTP.STATUS)) + CHR(13) + CHR(13) + "Compruebe su conexión a Internet o póngase en contacto con su proveedor tecnológico.")
ENDIF

RETURN ""

ENDFUNC
