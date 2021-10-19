# Integración vía WS para Proveedores

# Introducción

El objetivo de este documento es facilitar la labor de integración para los sistemas automatizados de proveedores dentro de la plataforma de facturación electrónica e.FACT a través de servicios Web. 

La integración permite tanto a proveedores para la gestión de sus propias facturas como a plataformas de facturación que ofrecen servicios como terceros para la gestión de facturas y estados con el servicio eFACT.

Las facturas enviadas deben ir firmadas con un certificado independiente del certificado usado en el servicio Web

# Método de autenticación

Respecto al certificado y autenticación, los certificados admitidos son los admitidos por la plataforma @firma del MINHAP y  cuya lista completa de los prestadores aceptados por esta plataforma se encuentran publicados en el siguiente enlace https://administracionelectronica.gob.es/PAe/aFirma-Anexo-PSC

La autenticación se realiza mediante la firma de la petición SOAP, ya que tanto las peticiones como las respuestas deben ir firmadas según el estándar OASIS WSSecurity 1.0 X509 Token Profile:

• http://en.wikipedia.org/wiki/WS-Security
• http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-tokenprofile-1.0.pdf

El certificado de firma para las peticiones, no es necesario proporcionarlo previamente

El único proto colo admitido en los servicios web es TLS>=1.2.


#	Operaciones

1.	Enviar Factura [enviarFactura]

    Este servicio permite enviar facturas al servicio eFACT
   
Petición - Parámetros de la información del proveedor

            La petición al servicio se estructura en 3 apartados:

           (a)  Información del proveedor 

                correo  :  correo destinatario de las distintas notificaciones asociadas a la factura.

          (b)  	Fichero factura 

                factura : Contenido codificado en base64 del documento .xsig de la factura, el fichero debe tener la extensión válida ".xsig".
                nombre  : Nombre del documento de la factura.
                mime    : Mime type del documento, en este caso debe ser "application/xml"

          (c) 	Ficheros Anexos . Los anexos son optativos, existe un máximo de 5 anexos o hasta 8MB 

                anexo   : Contenido codificado en base64 del documento anexo
                nombre  : Nombre del documento anexo.
                mime    : Mime type del documento. mimes admitidos
                          •	pdf application/pdf
                          •	doc	application/msword
                          •	docx	application/msword
                          •	xls	application/vnd.ms-excel
                          •	xlsx	application/vnd.ms-excel
                          •	odt	application/vnd.oasis.opendocument.text
                          •	ods	application/vnd.oasis.opendocument.spreadsheet
                          •	txt	text/plain

        Petición - RPC-Literal:

                <soapenv:Body>
                    <web:enviarFactura>
                      <request>
                         <!--You may enter the following 3 items in any order-->
                          <correo>desarrollo.efact@desarrolloefact.es</correo>
                          <factura>
                              <!--You may enter the following 3 items in any order-->
                              <factura>PD94bWwgdmVyc2lv...lOkZhY3R1cmFlPg==</factura>
                              <nombre>FC23.xsig</nombre>
                              <mime>application/xml</mime>
                          </factura>
                          <anexos>
                              <!--Zero or more repetitions:-->
                              <anexo>
                                <!--You may enter the following 3 items in any order-->
                                <anexo>PD94bWwgdmVyc</anexo>
                                <nombre>anexo.txt</nombre>
                                <mime>text/plain</mime>
                              </anexo>
                          </anexos>
                      </request>
                    </web:enviarFactura>
                  </soapenv:Body>

Respuesta :	La respuesta contiene los datos más representativos de la factura que ha sido enviada:

         Parámetros:

              numeroRegistro      : Código de identificación único de entrada, identificador único de la factura 
                                    dentro de la plataforma eFACT. 
              organoGestor        : Código dir del Organo Gestor destino.
              unidadTramitadora   :	Código dir de la unidad tramitadora destino.
              oficinaContable	    : Código dir de la oficina contable destino
              identificadorEmisor : Identificador del emisor (NIF o CIF o NIE ...)
              numeroFactura	      : Número de la factura.
              serieFactura	      : Serie de la factura
              fechaRecepcion	    : Fecha de recepción de la factura. Fecha de Entrada en eFACT


         Respuesta RPC-Literal

          <SOAP-ENV:Body wsu:Id="pfxedd7d608-0ac5-5cd1-3b59-f89cbdf1ee0d" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
                <ns1:enviarFacturaResponse>
                  <return>
                    <resultado>
                      <codigo>0</codigo>
                      <descripcion>Correcto</descripcion>
                      <codigoSeguimiento/>
                    </resultado>
                    <factura>
                      <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                      <organoGestor>P00000010</organoGestor>
                      <unidadTramitadora>P00000010</unidadTramitadora>
                      <oficinaContable>P00000010</oficinaContable>
                      <identificadorEmisor>12345678Z</identificadorEmisor>
                      <numeroFactura>NUMERO</numeroFactura>
                      <serieFactura>SERIE</serieFactura>
                      <fechaRecepcion>2021-09-17 13:17:48</fechaRecepcion>
                    </factura>
                  </return>
                </ns1:enviarFacturaResponse>
          
2.	Operación Consultar Factura [consultarFactura]
Este método permite consultar el estado de una factura. Esta petición buscará la factura con el código de registro indicado.

2.1	Petición




# Casos de prueba para cada operación

# Como darse de alta en el servicio

# Entornos


Se han diseñado los siguientes entornos disponibles para integradores de la plataforma:

  TEST	El entorno de TEST es un entorno de integración habilitado para pruebas de los sistemas de los proveedores.

  PROD	El entorno de PROD es un entorno real de la plataforma eFACT


Puede encontrar el wsdl de los servicios en las siguientes rutas:

  TEST	https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl 

  PROD	https://efact.eacat.cat/bustia/services/EFactWebServiceProxyService.wsdl



