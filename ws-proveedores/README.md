# Integración vía WS para Proveedores

1. [Introducción](#introducción)
2. [Método de autenticación](#método-de-autenticación)
3. [Operaciones](#operaciones)
   1. [Enviar Factura](#1-enviar-factura--enviarfactura)
   2. [Operación Consultar Factura](#2-operación-consultar-factura-consultarfactura)
   3. [Operación Consultar Estados](#3-operación-consultar-estados-consultarestados)
   4. [Operación Consultar Listado Facturas](#4-operación-consultar-listado-facturas-consultarlistadofacturas)
   5. [Flujo completo para solicitud descargas de estados pendientes para un emisor](#5-flujo-completo-para-solicitud-descargas-de-estados-pendientes-para-un-emisor)
      1. [Método 1: Solicitud de posibles descargas](#método-1-solicitud-de-posibles-descargas)
      2. [Método 2: Petición de descargas:](#método-2-petición-de-descargas)
      3. [Método 3: Confirmación de descargas](#método-3-confirmación-de-descargas)
4. [Casos de prueba para cada operación](#casos-de-prueba-para-cada-operación)
   1. [CONECTIVIDAD](#conectividad)
   2. [ENVIO FACTURA](#envio-factura)
   3. [CONSULTA ESTADOS](#consulta-estados)
   4. [ANULACION FACTURA](#anulacion-factura)
5. [Como darse de alta en el servicio](#como-darse-de-alta-en-el-servicio)
6. [Entornos](#entornos)


# Introducción

El objetivo de este documento es facilitar la labor de integración para los sistemas automatizados de proveedores dentro de la plataforma de facturación electrónica eFACT a través de servicios Web. 

La integración permite tanto a proveedores para la gestión de sus propias facturas como a plataformas de facturación que ofrecen servicios como terceros para la gestión de facturas y estados con el servicio eFACT.

Las facturas enviadas deben ir firmadas con un certificado independiente del certificado usado en el servicio Web.

# Método de autenticación

Respecto al certificado y autenticación, los certificados admitidos son los admitidos por la plataforma @firma del MINHAP y  cuya lista completa de los prestadores aceptados por esta plataforma se encuentran publicados en el siguiente [enlace](https://administracionelectronica.gob.es/PAe/aFirma-Anexo-PSC).

La autenticación se realiza mediante la firma de la petición SOAP, ya que tanto las peticiones como las respuestas deben ir firmadas según el estándar [WS-Security](http://en.wikipedia.org/wiki/WS-Security), en concreto el [OASIS WSSecurity 1.0 X509 Token Profile](https://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-x509-token-profile-1.0.pdf), y contenter un elemento [Timestamp](https://docs.oasis-open.org/wss/v1.1/wss-v1.1-spec-pr-SOAPMessageSecurity-01.pdf).

El certificado de firma para las peticiones se utilizará para identificar la plataforma integrada de cada proveedor y deberá ser proporcionado en el formulario de alta del servicio.

Las peticiones que no cumplan estos requisitos podrán ser rechazadas:

- El único protocolo admitido en los servicios web es TLS>=1.2.
- Por motivos de seguridad se requiere en la cabecera de las peticiones esté presente la definición Content-Length con el valor correcto en bytes.
- Se recomienda también la definición del charset sea en UTF-8. Ejemplo: `Content-Type: text/xml;charset=UTF-8`.
- Se admiten solicitudes `POST` con un tamaño máximo de 8M.

# Operaciones

## 1. Enviar Factura  [enviarFactura]

Este servicio permite enviar facturas al servicio eFACT

### Petición
-------------
Parámetros de la información del proveedor. La petición al servicio se estructura en 3 apartados:

**a) Información del proveedor**

parámetro|descripción| 
---------|-----------|
correo |  correo destinatario de las distintas notificaciones asociadas a la factura.

**b) Fichero factura**

parámetro|descripción| 
---------|-----------|
factura|  Contenido codificado en base64 del documento .xsig de la factura, el fichero debe tener la extensión válida ".xsig".
nombre|  Nombre del documento de la factura.
mime|  Mime type del documento, en este caso debe ser "application/xml" 

**c) Ficheros Anexos**
   
Los anexos son optativos, existe un máximo de 5 anexos o hasta 8MB 

parámetro|descripción| 
---------|-----------|
anexo| Contenido codificado en base64 del documento anexo
nombre |  Nombre del documento anexo.
mime|  Mime type del documento. 

__Restricciones en el nombre de los documentos anexos__
- El conjunto de caracteres permitidos para el nombre de un anexo es el formado por letras minúsculas y mayúsculas, números, guion medio (-), guion bajo (_), punto (.) y signo más (+). Todo carácter que no se encuentre dentro de este conjunto será eliminado.
- Los caracteres con algún signo ortográfico auxiliar (acentos, diéresis, etc.) se sustituirán por el carácter equivalente sin signo ortográfico.
- La letra ñ se sustituirá por la letra n y la cedilla (ç) por la letra c, respetando la forma original de mayúscula o minúscula.
- El carácter espacio en blanco se sustituirá el carácter por guion bajo.
- El nombre de los anexos podrá tener una longitud máxima de 70 caracteres, sin tener en cuenta la extensión. Los nombres que superen esta longitud se truncarán a 70 caracteres.

Para evitar posibles duplicidades en el nombre de los documentos anexos, se añadirá la posición del anexo precedida de un guion bajo como sufijo del nombre, justo antes de la extensión. Teniendo esto en cuenta, la longitud máxima del nombre final de un documento anexo, sin tener en cuenta la extensión, sería de 72 caracteres.

Por ejemplo, si junto a una factura se envían dos documentos anexos con nombres *"#InformeFacturación-Enero2024.xlsx"* y *"Justificante estándar operaciones (enero 2024).pdf"*, dichos anexos se renombrarán respectivamente a *"InformeFacturacion-Enero2024_1.xlsx"* y *"Justificante_estandar_operaciones_enero_2024_2.pdf"*.

__mimes admitidos__
- pdf: application/pdf
- doc: application/msword
- docx:	application/msword
- xls: application/vnd.ms-excel
- xlsx: application/vnd.ms-excel
- odt: application/vnd.oasis.opendocument.text
- ods: application/vnd.oasis.opendocument.spreadsheet
- txt: text/plain
   
#### RPC-Literal
```xml
<soapenv:Body>
    <web:enviarFactura>
        <request>
            <correo>desarrollo.efact@desarrolloefact.es</correo>
            <factura>
                <factura>PD94bWwgdmVyc2lv...lOkZhY3R1cmFlPg==</factura>
                <nombre>FC23.xsig</nombre>
                <mime>application/xml</mime>
            </factura>
            <anexos>
                <anexo>
                    <anexo>PD94bWwgdmVyc</anexo>
                    <nombre>anexo.txt</nombre>
                    <mime>text/plain</mime>
                </anexo>
            </anexos>
        </request>
    </web:enviarFactura>
</soapenv:Body>
```

### Respuesta 
------------

La respuesta contiene los datos más representativos de la factura que ha sido enviada:
 
parámetro|descripción| 
---------|-----------|
numeroRegistro| ódigo de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT.|
organoGestor|Código dir del Organo Gestor destino.|
unidadTramitadora|Código dir de la unidad tramitadora destino.
oficinaContable|Código dir de la oficina contable destino
identificadorEmisor|Identificador del emisor (NIF o CIF o NIE ...)
numeroFactura|Número de la factura.
serieFactura|Serie de la factura
fechaRecepcion|Fecha de recepción de la factura. Fecha de Entrada en eFACT

#### RPC-Literal 
```xml
<soapenv:Body>
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
</soapenv:Body>
```

###     Ejemplo peticion/respuesta:
-----------------------------------
             
[enviarFactura.zip](https://github.com/ConsorciAOC/eFact/files/7639552/enviarFactura.zip)

## 2.	Operación Consultar Factura [consultarFactura]

Este método permite consultar el estado de una factura. Esta petición buscará la factura con el código de registro indicado.

### Petición  
------------

parámetro|descripción| 
---------|-----------|
numeroRegistro|Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT. Parámetro recibido en la respuesta del método anterior (__enviarFactura.numeroRegistro__)
   
#### RPC-Literal 
```xml
<soapenv:Body>
    <web:consultarFactura>
        <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
    </web:consultarFactura>
</soapenv:Body>
```

### Respuesta  
-------------

parámetro|descripción| 
---------|-----------|
numeroRegistro|Código de identificación único de entrada, identificador único de la factura dentro de la plataforma eFACT. 
tramitacion|Información del estado de tramitación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
anulacion|Informacion del estado de anulación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
codigo|Código del estado actual de la factura
descripcion|Descripción del motivo del cambio de estado al actual
codigoSeguimiento|Comentario asociado al estado
registroAdministrativo|Numero de Registro Administrativo en eFACT 
   
#### RPC-Literal 
```xml                 
<soapenv:Body>
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
</soapenv:Body>
```

### Ejemplo peticion/respuesta:
-------------------------------

[consultarFactura.zip](https://github.com/ConsorciAOC/eFact/files/7639553/consultarFactura.zip)

# 3. Operación Consultar Estados [consultarEstados]
 
Este método permite obtener el listado de estados asignados a cambios en la factura. 

Existen dos flujos, el ordinario y el de anulación. El flujo ordinario corresponde al ciclo de vida de la factura, y el flujo de anulación corresponde al ciclo de solicitud de anulación.

La respuesta contiene los datos más representativos de los distintos estados por los que puede pasar una factura.

### Petición
------------

No tiene parámetros de entrada
#### RPC-Literal 
```xml 
<soapenv:Body>
    <web:consultarEstados/>
</soapenv:Body>
```

### Respuesta
-------------

parámetro|descripción| 
---------|-----------|
nombre|Nombre del estado. 
codigo|Código representativo y único del estado
descripcion|Descripción del estado
   
#### RPC-Literal 
```xml
<soapenv:Body>
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
</soapenv:Body>
```

# 4. Operación Consultar Listado Facturas [consultarListadoFacturas]
 
Este servicio permite consultar el estado de varias facturas. Este método permite buscar las facturas con el código de registro indicado. 
Se puede solicitar un máximo de 500 facturas por petición.

### Petición
------------
                
parámetro|descripción| 
---------|-----------|
listadoFacturas |Código de identificación único de entrada, identificador único de la factura dentro de la plataforma.

#### RPC-Literal
```xml
<soapenv:Body>
    <web:consultarListadoFacturas>
        <request>
            <!--Zero or more repetitions:-->
            <numeroRegistro>NUMERO_REGISTRO</numeroRegistro>
            <numeroRegistro>NUMERO_REGISTRO_2</numeroRegistro>
        </request>
    </web:consultarListadoFacturas>
</soapenv:Body>
```

### Respuesta
-------------

parámetro|descripción| 
---------|-----------| 
numeroRegistro| Código de identificación único de entrada identificador único de la factura dentro de la plataforma eFACT.  
tramitacion      | Información del estado de tramitación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
anulacion        | Información del estado de anulación. Contiene los elementos codigo_estado, descripcion_estado y motivo_estado
codigo           | Código del estado actual de la factura
descripcion      | Descripción del motivo del cambio de estado al actual
codigoSeguimiento| Comentario asociado al estado
registroAdministrativo |Numero de Registro Administrativo en eFACT 

#### RPC-Literal 
```xml
<soapenv:Body>
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
</soapenv:Body>
```

# 5. Flujo completo para solicitud descargas de estados pendientes para un emisor

A través de los siguientes métodos es posible obtener el listado completo de los estados que estan pendientes de descargar para un emisor de facturas

1. **solicitudDescargasEstados**: Solicitud de posibles descargas: Este método será el primero a ejecutar y obligatorio, en la solicitud de descargas pendientes para un Emisor

   Nos devuelve el XML con la lista completa de índices de descargas de estados que se encuentran en estado pendiente de descarga para el NIF que se incluye como parámetro de entrada

2. **peticionDescargasEstados**: Petición de descargas: Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes para un Emisor.

   A partir del listado de índices obtenido en el método anterior (solicitudDescargasEstados ) realizaremos la peticion tantas veces como índices de descargas se obtuvieran en el primer método (a la elección del interlocutor).

   El parámetro opcionMarcado (S/N) determina si el índice se marca como descargado al finalizar la descarga o bien será necesario el método 3 (confirmacionDescargasEstados) para marcar como descargado los índices, puesto que el documento estará pendiente de descarga hasta lanzar este ultimo método

3. **confirmacionDescargasEstados**: Confirmación de descargas: Este método será el tercero a ejecutar y obligatorio, en la petición de descargas pendientes si la opcion de Marcado ha sido N

   Este método será el tercero a ejecutar y es opcional, en la petición de descargas pendientes para un interlocutor determinado.

   Su objetivo es cambiar de estado en eFACT y por cada índice de descarga ya realizado, de “PendienteDescarga” a “Descargado”.

   Para ello se puede utilizar el Método 2 en el mismo momento de la descarga y mediante el parámetro de entrada “Opción marcado”, o habiéndose cerrado las descargas de forma correcta, invocar al Método 3 para confirmarlas tras su recepción.

## Método 1: Solicitud de posibles descargas

Este método será el primero a ejecutar y obligatorio, en la solicitud de descargas pendientes para un Emisor

### Petición
------------

parámetro|descripción| 
---------|-----------|
identificadorEmisor | NIF Emisor de Facturas .

#### RPC-Literal 
```xml
<soapenv:Body>
    <sspp:solicitudDescargasEstados>
        <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
    </sspp:solicitudDescargasEstados>
</soapenv:Body>
```

### Respuesta
-------------

parámetro|descripción| 
---------|-----------|
resultado    | 0 -->  Ok<br/> 1 -->  Error
observaciones|Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:<br/>1) Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.<br/>2) No se pudieron obtener los documentos pendientes
ficheroResultante | XML con el índice de descargas, comprimido en formato ZIP y en Base64

#### RPC-Literal 
```xml
<soapenv:Body>
    <sspp:solicitudDescargasEstadosResponse >
        <return>
            <resultado>0</resultado>
            <observaciones>OK</observaciones>
            <ficheroResultante>UEsDBBQACAAIAGNfxFAAAAAAAAAAAAAAAAAjAAAAcGV0a…. </ficheroResultante>
        </return>
    </sspp:solicitudDescargasEstadosResponse>
</soapenv:Body>
```

### Índice de descargas
------------------------------

El formato del índice de descargas es el siguiente:

parámetro|descripción| 
---------|-----------|
Document | Habrá un elemento de este tipo por cada factura que tenga cambios de estado pendientes de descargar
DocId    | Identificador del fichero de estados a descargar (deberá proporcionarse en la siguiente operación)
Supplier | Datos del emisor de la factura
Buyer    | Datos del receptor de la factura

#### XML 
```xml
<DocumentList>
  <Document>
    <DocId>11294</DocId>
    <DocType>003</DocType>
    <DocProcessingDate>2021-03-08</DocProcessingDate>
    <Supplier>
      <TaxNumber>IDENTIFICADOR_EMISOR</TaxNumber>
      <DeptCode></DeptCode>
    </Supplier>
    <Buyer>
      <TaxNumber>IDENTIFICADOR_RECEPTOR</TaxNumber>
      <DeptCode></DeptCode>
    </Buyer>
    <DocNumber>1</DocNumber>
    <DocDate>2021-03-08</DocDate>
  </Document>
  <Document>
      ...
  </Document>
</DocumentList>
```

### Ejemplo peticion/respuesta
------------------------------

[solicitudDescargasEstados.zip](https://github.com/ConsorciAOC/eFact/files/7639564/solicitudDescargasEstados.zip)

## Método 2: Petición de descargas: 

Este método será el segundo a ejecutar y obligatorio, en la petición de descargas pendientes de estados 

Siempre será realizado tras la ejecución del primer método y tantas veces como índices de descargas se obtuvieran en el primer método (a la elección del interlocutor).

### Petición
------------

parámetro|descripción| 
---------|-----------|
identificadorEmisor  | NIF Emisor de Facturas .
indiceDescarga       | Índice de descarga
opcionMarcado        | Admite valor S o N:<br/>S -> Si marca el índice en eFACT, con lo que no es necesario el Método 3 del WS.<br/>N -> No marca el índice en eFACT, por lo que el documento seguirá pdte de descarga y habrá que ejecutar el Método 3.

#### RPC-Literal 
```xml
<soapenv:Body>
    <sspp:peticionDescargasEstados>
        <peticionDescargas>
            <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
            <indiceDescarga>INDICE_DESCARGA</indiceDescarga>
            <opcionMarcado>N</opcionMarcado>
        </peticionDescargas>
    </sspp:peticionDescargasEstados>
</soapenv:Body>
```

### Respuesta
-------------

parámetro|descripción| 
---------|-----------|
resultado    | 0 --> Ok<br/>1 --> Error
observaciones|Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:<br/>1) Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.<br/>2) No se pudieron obtener los documentos pendientes
ficheroResultante | XML de Estado comprimido ZIP en y en Base64
 
#### RPC-Literal 
```xml
<soapenv:Body>
    <sspp:solicitudDescargasEstadosResponse >
        <return>
            <resultado>01</resultado>
            <observaciones/>
            <ficheroResultante>UEsDBBQACAAIAGNfxFAAAAAAAAAAAAAAAAAjAAAAcGV0a…. </ficheroResultante>
        </return>
    </sspp:solicitudDescargasEstadosResponse>
</soapenv:Body>
```

### Fichero de estados
----------------------

El formato del fichero de estados es el siguiente:

parámetro|descripción| 
---------|-----------|
InvoiceFeedback | Cambio de estado de la factura
InvoiceId      | Número de la factura
Supplier       | Datos del emisor de la factura
Buyer          | Datos del receptor de la factura
InvoiceDate    | Fecha de la factura
Feedback       | Datos del cambio de estado
Status         | Estado de la factura
StatusCode     | Código de estado de la factura
StatusDate     | Fecha de cambio de estado
Reason         | Razón del cambio de estado
Description    | Descripción del cambio de estado

#### XML 
```xml
<DeliveryFeedback>
  <StatusFeedback>
    <HubFeedback>
      <HubId>333508</HubId>
    </HubFeedback>
    <InvoiceFeedback>
      <InvoiceId>1</InvoiceId>
      <Supplier>
        <Cif>IDENTIFICADOR_EMISOR</Cif>
      </Supplier>
      <Buyer>
        <Cif>IDENTIFICADOR_RECEPTOR</Cif>
      </Buyer>
      <InvoiceDate>2021-03-08</InvoiceDate>
      <Feedback>
        <Status>REJECTED</Status>
        <StatusCode>2600</StatusCode>
        <StatusDate>2021-03-08T12:22:00</StatusDate>
        <Reason>
          <Code>HRE1</Code>
          <Description>Error al registrar document</Description>
        </Reason>
      </Feedback>
    </InvoiceFeedback>
  </StatusFeedback>
</DeliveryFeedback>
```

El esquema del mensaje XML de `DeliveryFeeback` lo podeis descargar de [aquí](xsds/DeliveryFeedback.xsd).

## Método 3: Confirmación de descargas

Este método será el tercero a ejecutar y es opcional, en la petición de descargas pendientes para un interlocutor determinado.

Su objetivo es cambiar de estado en eFACT y por cada índice de descarga ya realizado, de “PendienteDescarga” a “Descargado”.

Para ello se puede utilizar el Método 2 en el mismo momento de la descarga y mediante el parámetro de entrada “Opción marcado”, o habiéndose cerrado las descargas de forma correcta, invocar al Método 3 para confirmarlas tras su recepción.

### Petición
------------

parámetro|descripción| 
---------|-----------|
identificadorEmisor | NIF Emisor de Facturas
indiceDescarga      | Índice de descarga

#### RPC-Literal 
```xml

<soapenv:Body>
    <sspp:confirmacionDescargasEstados>
        <confirmacionDescargas>
            <identificadorEmisor>IDENTIFICADOR_EMISOR</identificadorEmisor>
            <indiceDescarga>INDICE_DESCARGA</indiceDescarga>
        </confirmacionDescargas>
    </sspp:confirmacionDescargasEstados>
</soapenv:Body>
```

### Respuesta
-------------

parámetro|descripción| 
---------|-----------|
resultado    | 0 --> Ok<br/>1 --> Error
observaciones|Los posibles mensajes de respuesta en caso de error para este método, son los siguientes:<br/>1) Los datos enviados en el campo TypeRequest no son soportados o no han sido enviados.<br/>2) No se pudieron obtener los documentos pendientes
ficheroResultante | Fichero comprimido y en Base64

#### RPC-Literal 
```xml
<soapenv:Body>
    <sspp:confirmacionDescargasEstadosResponse>
        <return>
            <resultado>01</resultado>
            <observaciones xsi:nil="true"/>
            <ficheroResultante xsi:nil="true"/>
        </return>
    </sspp:confirmacionDescargasEstadosResponse>
</soapenv:Body>
```

# Casos de prueba para cada operación

## CONECTIVIDAD

Prueba de conectividad utilizando servicios web a través de Internet

![imagen](https://user-images.githubusercontent.com/92558339/144372517-22bfb612-78f3-4a6d-8bdc-0e0066902a49.png)

## ENVIO FACTURA

Pruebas de envío factura firmada correcta al sistema.	

![imagen](https://user-images.githubusercontent.com/92558339/144371474-9918c065-81b0-42ba-b4f4-ed0c22dc13e4.png)

## CONSULTA ESTADOS		

Pruebas para consultar los posibles estados de  una factura.	

![imagen](https://user-images.githubusercontent.com/92558339/144371447-3dd80daf-96dc-4122-875f-54121d54eeea.png)

Pruebas para consultar el estado de una factura a partir de un identificador de registro existente	

![imagen](https://user-images.githubusercontent.com/92558339/144374184-db492fd9-8468-4f44-b6cc-6702455f3bb1.png)

## ANULACION FACTURA

Pruebas asociadas a la anulación de una factura con id de registro existente y estado permitido.

![imagen](https://user-images.githubusercontent.com/92558339/144371545-09889442-c996-42c4-a5c8-e68e9df4c032.png)

# Como darse de alta en el servicio

Para darse de alta se debe rellenar la solicitud de adhesion a eFACT que se puede encontrar en el portal de soporte del servicio [eFACT](https://suport-efact-empreses.aoc.cat/hc/ca/articles/4415512590481).

# Entornos

Se han diseñado los siguientes entornos disponibles para integradores de la plataforma:

- **TEST**:	El entorno de TEST es un entorno de integración habilitado para pruebas de los sistemas de los proveedores.
- **PROD**:	El entorno de PROD es el entorno real de la plataforma eFACT.

Puede encontrar la definición de los servicios en formato WSDL en las siguientes rutas:

- **TEST**: [https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl](https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl)
- **PROD**: [https://efact.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl](https://efact.aoc.cat/bustia/services/EFactWebServiceProxyService.wsdl)

La URL de consumo del servicio para cada entorno es la siguiente:

- **TEST**: [https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService](https://efact-pre.aoc.cat/bustia/services/EFactWebServiceProxyService)
- **PROD**: [https://efact.aoc.cat/bustia/services/EFactWebServiceProxyService](https://efact.aoc.cat/bustia/services/EFactWebServiceProxyService)

Los certificados con los que se firman las respuestas XML son los siguientes para cada entorno (expiran cada pocos años y hay que mantenerlos actualizados):

- **TEST**: [certificado-eFACT-TEST-2025.p7b](certificados/certificado-eFACT-TEST-2025.p7b) (expira el 26/2/2028)
- **PROD**: [certificado-eFACT-PROD-2025.p7b](certificados/certificado-eFACT-PROD-2025.p7b) (expira el 20/2/2028)
