# eFact

__eFACT es el servicio de factura electrónica de las administraciones públicas catalanas para la recepción de las facturas electrónicas (e-facturas) por parte de sus proveedores.__

En este sentido, el servicio eFACT ni genera ni hace la gestión contable de las facturas pues de eso se encargan el programa de creación de facturas electrónicas que utilice el proveedor y el programa de gestión contable que utilice la administración destinataria, respectivamente.

Este repositorio pretende ir recopilando poco a poco la nueva documentación relacionada con eFACT conforme ésta es vaya actualizando.

## Formato factura-e 

Conforme a la Ley 25/2013, de 27 de diciembre, de impulso de la factura electrónica y creación del registro  contable de facturas en el Sector Público, las facturas que se remitan a las Administraciones Públicas serán electrónicas y se ajustarán al formato estructurado de factura electrónica Facturae versión 3.2.x con firma electrónica XAdES.

Por tanto, el __formato admitido por la plataforma es Facturae, concretamente las versiones: 3.2, 3.2.1 y 3.2.2.__

Tiene más información sobre dicho formato en: 

https://www.facturae.gob.es/formato/Paginas/version-3-2.aspx

La firma XAdES de la factura permitida se recoge  y detalla en el siguiente link :

https://www.facturae.gob.es/formato/Paginas/politicas-firma-electronica.aspx

El certificado usado para la firma podrá ser el certificado del emisor de la factura o bien el certificado de un Tercero de confianza en modalidad de firma delegada. 


## Flujo de vida de una factura

![Flujo de Vida](/imgs/diagrama-estados.png)

El siguiente cuadro muestra el flujo de vida natural de una factura enviada por el proveedor al servicio eFACT:

|Nombre|Codigo Estado: <br> BOE A - 2014 - 10660|eFACT|Descripción|
|------|----------------------------------------|-----|-----------|
||1000|SENT|Entregada en el servicio eFACT|
|Registrada|1200|REGISTERED|La factura electrónica ha sido redibida en el punto general de entrada de facturas y ha sido registrada administrativamente, proporcionando un número de registro al proveedor|
|Registrada en RCF|1300|DELIVERED|El mensaje ha llegado a los sistemas informáticos de la AA.PP. receptora, ha pasado las validaciones requeridas por éstos y está a disposición del/de los usuario/s receptor/es. Si no se ha podido llevar a cabo la entrega, se tendrá que generar un mensaje de rechazo indicando los motivos técnicos que hayan impedido la correcta entrega (por ejemplo: "La factura no puede ser entregada, el código de encaminamiento es incorrecto"). La publicación de este estado, o el correspondiente rechazo es obligatorio|
|Contabilizada la obligación de pago|2400|RECOGNISED|La obligación de pago derivada de la factura ha sido reconocida|
|Pagada|2500|PAID|La obligación de pago derivada de la factura ha sido pagada|

Durante el proceso de registro y aprobación de la factura se puede producir el rechazo de la factura , bien por problemas de forma o contenido en la factura. En ese momento la factura pasara a estado Rechazada:

|Nombre|Codigo Estado: <br> BOE A - 2014 - 10660|eFACT|Descripción|
|------|----------------------------------------|-----|-----------|
|Rechazada|2600|REJECTED|El portal general de entrada de facturas no ha podido identificar al destinatario. La oficina contable o la unidad tramitadora han rechzado la factura, se debe indicar al proveedor el motivo del rechazo|

# Documentación para integradores

En este apartado podéis encontrar la guía de integración via WS para los diferentes servicios web proporcionados por eFact:

[Integración WS para proveedores de facturas](/ws-proveedores/README.md) (válido hasta 01/01/2026)

[Integración WS para Registros Contables de Facturas](/ws-rcf/README.md)

# Enlaces de interés

A continuación podéis encontrar una serie de enlaces a sitios de interés:

[Portal de soporte de eFact](https://suport-efact-empreses.aoc.cat/hc/ca/)

[Relación de cambios en la documentación](/CHANGELOG.md)
