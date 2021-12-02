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

Las peticiones que no cumplan estos requisitos podrán ser rechazadas:

- El único protocolo admitido en los servicios web es TLS>=1.2.
- Por motivos de seguridad se requiere en la cabecera de las peticiones esté presente la definición Content-Length con el valor correcto en bytes.
- Se recomienda también la definición del charset sea en UTF-8. Ejemplo:"Content-Type: text/xml;charset=UTF-8".
- Se admiten solicitudes POST con un tamaño máximo de 8M.





#	Operaciones

1.	Enviar Factura [enviarFactura]

    Este servicio permite enviar facturas al servicio eFACT

[enviarFactura.zip](https://github.com/ConsorciAOC/eFact/files/7639552/enviarFactura.zip)

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
              oficinaContable	  : Código dir de la oficina contable destino
              identificadorEmisor : Identificador del emisor (NIF o CIF o NIE ...)
              numeroFactura	      : Número de la factura.
              serieFactura	      : Serie de la factura
              fechaRecepcion	  : Fecha de recepción de la factura. Fecha de Entrada en eFACT


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


[consultarFactura.zip](https://github.com/ConsorciAOC/eFact/files/7639553/consultarFactura.zip)


Petición

    Parámetros : 

                numeroRegistro  :Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT. Parámetro recibido en la                                   respuesta del método anterior [1] enviarFactura.numeroRegistro

    Petición RPC-Literal:

                <soapenv:Body>
                <web:consultarFactura>
                    <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                </web:consultarFactura>
                </soapenv:Body>

Respuesta

    Parámetros : 
                numeroRegistro :Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT. 
                tramitacion:Información del estado de tramitación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
                anulacion:Informacion del estado de anulación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
                codigo:Código del estado actual de la factura
                descripcion:Descripción del motivo del cambio de estado al actual
                codigoSeguimiento: Comentario asociado al estado
                registroAdministrativo:Numero de Registro Administrativo en eFACT 
    
    Respuesta RPC-Literal:
                    <SOAP-ENV:Body wsu:Id="pfx60b0f6ad-d92f-b2de-a3a8-4d78d13b77d1" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-                     utility-1.0.xsd">
                    <ns1:consultarFacturaResponse>
                        <return>
                            <resultado>
                                <codigo>0</codigo>
                                <descripcion>Correcto</descripcion>
                                <codigoSeguimiento/>
                                <registroAdministrativo >12345</registroAdministrativo>
                            </resultado>
                            <factura>
                                <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                                <tramitacion>
                                    <codigo>1200</codigo>
                                    <descripcion>La factura ha sido registrada en el registro electrónico REC</descripcion>
                                    <motivo/>
                                </tramitacion>
                                <anulacion>
                                    <codigo>4200</codigo>
                                    <descripcion>Solicitada anulación</descripcion>
                                    <motivo>prueba</motivo>
                                </anulacion>
                            </factura>
                        </return>
                </ns1:consultarFacturaResponse>
                </SOAP-ENV:Body>


3.	Operación Anular Factura [anularFactura]

Este método permite solicitar la anulación de una factura ya enviada.[anularFactura.zip](https://github.com/ConsorciAOC/eFact/files/7639560/anularFactura.zip)




Petición

    Parámetros : 

                numeroRegistro  : Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT.
                motivo          : Descripción del motivo por el que se hace la anulación de la factura
                
    Petición RPC-Literal:
    
                <soapenv:Body>
                    <web:anularFactura>
                        <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                         <motivo>DESCRIPCION DEL MOTIVO DE LA SOLICITUD DE ANULACION</motivo>
                    </web:anularFactura>
                </soapenv:Body>


Respuesta

    Parámetros : 
                numeroRegistro : Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT. 
                mensaje        : "Anulación solicitada correctamente". En caso de error o cambio de estado no válido, se devolverá la excepción correspondiente.
                
                
    Respuesta RPC-Literal:
               <SOAP-ENV:Body wsu:Id="pfxa7090bef-4c86-f3b5-fd4f-b2a425c84699" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
                <ns1:anularFacturaResponse>
                    <return>
                        <resultado>
                            <codigo>0</codigo>
                            <descripcion>Correcto</descripcion>
                            <codigoSeguimiento/>
                        </resultado>
                        <factura>
                            <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                            <mensaje>Anulación solicitada correctamente</mensaje>
                        </factura>
                    </return>
                </ns1:anularFacturaResponse>
            </SOAP-ENV:Body>

 4.	Operación Consultar Estados [consultarEstados]
 
Este método permite obtener el listado de estados asignados a cambios en la factura. 

Existen dos flujos, el ordinario y el de anulación. El flujo ordinario corresponde al ciclo de vida de la factura, y el flujo de anulación corresponde al ciclo de solicitud de anulación.

La respuesta contiene los datos más representativos de los distintos estados por los que puede pasar una factura.

Petición

    Parámetros : 

               No tiene parámetros de entrada
                
    Petición RPC-Literal:
    
                <soapenv:Body>
                    <web:consultarEstados/>
                </soapenv:Body>

Respuesta

    Parámetros : 
                nombre        : Nombre del estado. 
                codigo        : Código representativo y único del estado
                descripcion   : Descripción del estado
                
                
    Respuesta RPC-Literal:

            <SOAP-ENV:Body wsu:Id="pfx6ab483c6-1e52-135b-47f3-80cc1287a700" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
            <ns1:consultarEstadosResponse>
                    <return>
                        <resultado>
                            <codigo>0</codigo>
                            <descripcion>Correcto</descripcion>
                            <codigoSeguimiento/>
                        </resultado>
                        <estados>
                            <estado>
                                <nombre>Registrada</nombre>
                                <codigo>1200</codigo>
                                <descripcion>La factura ha sido registrada en el registro electrónico REC</descripcion>
                            </estado>
                            <estado>
                                <nombre>Contabilizada la obligación reconocida</nombre>
                                <codigo>2400</codigo>
                                <descripcion>Contabilizada la obligación reconocida</descripcion>
                            </estado>
                            <estado>
                                <nombre>Pagada</nombre>
                                <codigo>2500</codigo>
                                <descripcion>Factura pagada</descripcion>
                            </estado>
                            <estado>
                                <nombre>Rechazada</nombre>
                                <codigo>2600</codigo>
                                <descripcion>La Unidad rechaza la factura</descripcion>
                            </estado>
                            <estado>
                                <nombre>Anulada</nombre>
                                <codigo>3100</codigo>
                                <descripcion>La Unidad aprueba la propuesta de anulación</descripcion>
                            </estado>
                            <estado>
                                <nombre>No solicitada anulación</nombre>
                                <codigo>4100</codigo>
                                <descripcion>No solicitada anulación</descripcion>
                            </estado>
                            <estado>
                                <nombre>Solicitada anulación</nombre>
                                <codigo>4200</codigo>
                                <descripcion>Solicitada anulación</descripcion>
                            </estado>
                            <estado>
                                <nombre>Aceptada anulación</nombre>
                                <codigo>4300</codigo>
                                <descripcion>Aceptada anulación</descripcion>
                            </estado>
                            <estado>
                                <nombre>Rechazada anulación</nombre>
                                <codigo>4400</codigo>
                                <descripcion>Rechazada anulación</descripcion>
                            </estado>
                        </estados>
                    </return>
                </ns1:consultarEstadosResponse>
            </SOAP-ENV:Body>


5.	Operación Consultar Listado Facturas [consultarListadoFacturas]
 
Este servicio permite consultar el estado de varias facturas. Este método permite buscar las facturas con el código de registro indicado. 
Se puede solicitar un máximo de 500 facturas por petición.


Petición

    Parámetros : 

               listadoFacturas :Código de identificación único de entrada, identificador único de la factura dentro de la plataforma.
                
    Petición RPC-Literal:
                <soapenv:Body>
                      <web:consultarListadoFacturas>
                        <request>
                        <!--Zero or more repetitions:-->
                            <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
                            <numeroRegistro>NUMERO_REGISTRO_2</numeroRegistro>
                        </request>
                       </web:consultarListadoFacturas>
                </soapenv:Body>

Respuesta

    Parámetros : 
    
                numeroRegistro   : Código de identificación único de entrada identificador único de la factura dentro de la plataforma eFACT.  
                tramitacion      : Información del estado de tramitación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
                anulacion        : Información del estado de anulación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
                codigo           : Código del estado actual de la factura
                descripcion      : Descripción del motivo del cambio de estado al actual
                codigoSeguimiento: Comentario asociado al estado
                registroAdministrativo :Numero de Registro Administrativo en eFACT 
                
    Respuesta RPC-Literal:
                <SOAP-ENV:Body wsu:Id="pfx1afbd40b-e707-1354-a37f-e11b7b2523af" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
                    <ns1:consultarListadoFacturasResponse>
                        <return>
                            <resultado>
                                <codigo>0</codigo>
                                <descripcion>Correcto</descripcion>
                                <codigoSeguimiento/>
                                <registroAdministrativo>12345</registroAdministrativo>
                            </resultado>
                            <facturas>
                                <consultarListadoFactura>
                                    <codigo>0</codigo>
                                    <descripcion>Correcto</descripcion>
                                    <factura>
                                        <numeroRegistro>NUMERO_REGISTRO</numero Registro>
                                        <tramitacion>
                                            <codigo>1200</codigo>
                                            <descripcion>La factura ha sido registrada en el registro electrónico REC</descripcion>
                                            <motivo/>
                                        </tramitacion>
                                        <anulacion>
                                            <codigo>4200</codigo>
                                            <descripcion>Solicitada anulación</descripcion>
                                            <motivo>prueba</motivo>
                                        </anulacion>
                                    </factura>
                                </consultarListadoFactura>
                                <consultarListadoFactura>
                                    <codigo>303</codigo>
                                    <descripcion>No existe factura con el número de registro especificado</descripcion>
                                    <factura>
                                        <numeroRegistro>NUMERO_REGISTRO_2</nume roRegistro>
                                        <tramitacion/>
                                        <anulacion/>
                                    </factura>
                                </consultarListadoFactura>
                            </facturas>
                        </return>
                     </ns1:consultarListadoFacturasResponse>
                </SOAP-ENV:Body>

6.	Flujo completo para solicitud descargas de estados pendientes para un emisor

A través de los siguientes métodos es posible obtener el listado completo de los estados que estan pendientes de descargar para un emisor de facturas

    1 - solicitudDescargasEstados : Solicitud de posibles descargas: Este método será el primero a ejecutar y obligatorio, en la solicitud de descargas pendientes para un Emisor

Nos devuelve el XML con la lista completa de índices de descargas de estados que se encuentran en estado pendiente de descarga para el NIF que se incluye como parámetro de entrada


    2 - peticionDescargasEstados : Petición de descargas: Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes para un Emisor
A partir del listado de índices obtenido en el método anterior (solicitudDescargasEstados ) realizaremos la peticion tantas veces como índices de descargas se obtuvieran en el primer método (a la elección del interlocutor).
El parámetro opcionMarcado (S/N) determina si el índice se marca como descargado al finalizar la descarga o bien será necesario el método 3 (confirmacionDescargasEstados) para marcar como descargado los índices, puesto que el documento estará pendiente de descarga hasta lanzar este ultimo método

    3 - confirmacionDescargasEstados Confirmación de descargas: Este método será el tercero a ejecutar y obligatorio, en la petición de descargas pendientes si la opcion de Marcado ha sido N
Este método será el tercero a ejecutar y es opcional, en la petición de descargas pendientes para un interlocutor determinado 
Su objetivo es cambiar de estado en eFACT y por cada índice de descarga ya realizado, de “PendienteDescarga” a “Descargado”.
Para ello se puede utilizar el Método 2 en el mismo momento de la descarga y mediante el parámetro de entrada “Opción marcado”, o habiéndose cerrado las descargas de forma correcta, invocar al Método 3 para confirmarlas tras su recepción.
    
    

6.1. Operación Solicitud Descargas Estados [solicitudDescargasEstados]      
[solicitudDescargasEstados.zip](https://github.com/ConsorciAOC/eFact/files/7639564/solicitudDescargasEstados.zip)


[Método 1 :Solicitud de posibles descargas: Este método será el primero a ejecutar y obligatorio, en la solicitud de descargas pendientes para un Emisor]

Petición

    Parámetros : 

               identificadorEmisor : NIF Emisor de Facturas .
                
    Petición RPC-Literal:

            <soapenv:Body wsu:Id="id-F0E6B1B6B4815A5F1A1591264747069161=" xmlns:wsu"http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
                  <sspp:solicitudDescargasEstados>
                     <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
                  </sspp:solicitudDescargasEstados>
               </soapenv:Body>

Respuesta

    Parámetros : 
    
    
                resultado   :
                                01      -->  Ok  
                                02	    -->  Error

                observaciones      : Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:
                                    •	Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.
                                    •	No se pudieron obtener los documentos pendientes

                ficheroResultante        : XML con el índice de descargas, comprimido y en Base64  ![imagen](https://user-images.githubusercontent.com/92558339/144367219-95a97ed4-87e4-4d8e-9bcd-847da8aa3911.png)

                
                
    Respuesta RPC-Literal:
    
                <soapenv:Body wsu:Id="pfx1afbd40b-e707-1354-a37f-e11b7b2523af" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
            <sspp:solicitudDescargasEstadosResponse >
                     <return>
                        <resultado>01</resultado>
                        <observaciones/>
                        <ficheroResultante>UEsDBBQACAAIAGNfxFAAAAAAAAAAAAAAAAAjAAAAcGV0a…. </ficheroResultante>
                     </return>
            </sspp:solicitudDescargasEstadosResponse>
             </ soapenv:Body>

   

    
6.2. Operación Petición Descargas Estados [peticionDescargasEstados]      

[Método 2 : Petición de descargas: Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes]
Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes de estados 
Siempre será realizado tras la ejecución del primer Método y tantas veces como índices de descargas se obtuvieran en el primer método (a la elección del interlocutor).

Petición

    Parámetros : 

               identificadorEmisor  : NIF Emisor de Facturas .
               indiceDescarga       : Índice de descarga
               opcionMarcado        : Admite valor S o N: 
                            S -> Si marca el índice en eFACT, con lo que no es necesario el Método 3 del WS
                            N -> No marca el índice en eFACT, por lo que el documento seguirá pdte de descarga y habrá que ejecutar el Método 3.

                
    Petición RPC-Literal:
    
                <soapenv:Body wsu:Id="id-F0E6B1B6B4815A5F1A1591266492637167" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
            <sspp:peticionDescargasEstados>
                     <peticionDescargas>
                        <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
                        <indiceDescarga>INDICE_DESCARGA</indiceDescarga>
                        <opcionMarcado>N</opcionMarcado>
                     </peticionDescargas>
                  </sspp:peticionDescargasEstados>
            </soapenv:Body>

Respuesta

    Parámetros : 
    
    
                resultado   :
                                01      -->  Ok  
                                02	    -->  Error

                observaciones      : Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:
                                    •	Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.
                                    •	No se pudieron obtener los documentos pendientes

                ficheroResultante        : XML de Estado comprimido ZIP en y en Base64  ![imagen](https://user-images.githubusercontent.com/92558339/144369376-c844c078-4427-4ad6-b07d-ed01a2087ae6.png)


                
                
    Respuesta RPC-Literal:
    
                <soapenv:Body wsu:Id="pfx1afbd40b-e707-1354-a37f-e11b7b2523af" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
            <sspp:solicitudDescargasEstadosResponse >
                     <return>
                        <resultado>01</resultado>
                        <observaciones/>
                        <ficheroResultante>UEsDBBQACAAIAGNfxFAAAAAAAAAAAAAAAAAjAAAAcGV0a…. </ficheroResultante>
                     </return>
            </sspp:solicitudDescargasEstadosResponse>
             </ soapenv:Body>

    
6.3. Operación Confirmación Descargas Estados [confirmacionDescargasEstados]    

[Método 3 : Confirmación de descargas: Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes ]
Este método será el tercero a ejecutar y es opcional, en la petición de descargas pendientes para un interlocutor determinado 
Su objetivo es cambiar de estado en eFACT y por cada índice de descarga ya realizado, de “PendienteDescarga” a “Descargado”.
Para ello se puede utilizar el Método 2 en el mismo momento de la descarga y mediante el parámetro de entrada “Opción marcado”, o habiéndose cerrado las descargas de forma correcta, invocar al Método 3 para confirmarlas tras su recepción.

Petición

    Parámetros : 

               identificadorEmisor : NIF Emisor de Facturas .
               indiceDescarga      : Índice de descarga
                
    Petición RPC-Literal:

            <soapenv:Body wsu:Id="id-F0E6B1B6B4815A5F1A1591266492637167" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
            <sspp:confirmacionDescargasEstados>
                     <confirmacionDescargas>
                        <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
                        <indiceDescarga>INDICE_DESCARGA</indiceDescarga>
                     </confirmacionDescargas>
             </sspp:confirmacionDescargasEstados>
            </soapenv:Body>

Respuesta

    Parámetros : 
    
    
                resultado   :
                                01      -->  Ok  
                                02	    -->  Error

                observaciones      : Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:
                                    •	Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.
                                    •	No se pudieron obtener los documentos pendientes

                ficheroResultante        : Fichero comprimido y en Base64  !

                
                
    Respuesta RPC-Literal:

        <soapenv:Body wsu:Id="id-F0E6B1B6B4815A5F1A1591266492637167" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss- wssecurity-utility-1.0.xsd">
        <sspp:confirmacionDescargasEstadosResponse>
                 <return>
                    <resultado>01</resultado>
                    <observaciones xsi:nil= "true"/>
                    <ficheroResultante xsi:nil= "true"/>
                 </return>
              </sspp:confirmacionDescargasEstadosResponse>

        </ soapenv:Body >


    
# Casos de prueba para cada operación

CON.1		Prueba de conectividad utilizando servicios web a través de Internet	

![imagen](https://user-images.githubusercontent.com/92558339/144371349-93e8763f-afc3-467c-a4c8-26ec4ff2842d.png)


CNU.1		Prueba de consultas de unidades	

![imagen](https://user-images.githubusercontent.com/92558339/144371322-ac7503c9-2090-4223-b1a1-e5aca913e88b.png)


CNE.1		Pruebas para consultar los posibles estados de  una factura.	

![imagen](https://user-images.githubusercontent.com/92558339/144371447-3dd80daf-96dc-4122-875f-54121d54eeea.png)

ENF.1		Pruebas de envío factura firmada correcta al sistema.	

![imagen](https://user-images.githubusercontent.com/92558339/144371474-9918c065-81b0-42ba-b4f4-ed0c22dc13e4.png)


CNF.1		Pruebas para consultar el estado de una factura a partir de un identificador de registro existente	

![imagen](https://user-images.githubusercontent.com/92558339/144371514-9f19e0ac-a5e7-434f-ae9a-271076c6218f.png)


ANF.1		Pruebas asociadas a la anulación de una factura con id de registro existente y estado permitido.	

![imagen](https://user-images.githubusercontent.com/92558339/144371545-09889442-c996-42c4-a5c8-e68e9df4c032.png)


# Como darse de alta en el servicio

# Entornos


Se han diseñado los siguientes entornos disponibles para integradores de la plataforma:

  TEST	El entorno de TEST es un entorno de integración habilitado para pruebas de los sistemas de los proveedores.

  PROD	El entorno de PROD es un entorno real de la plataforma eFACT


Puede encontrar el wsdl de los servicios en las siguientes rutas:

  TEST	https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl 

  PROD	https://efact.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl




