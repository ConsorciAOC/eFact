# eFact

e.FACT es el servicio de factura electrónica de las administraciones públicas catalanas para la recepción de las facturas electrónicas (e-facturas) por parte de sus proveedores. En este sentido, el servicio e.FACT ni genera ni hace la gestión contable de las facturas pues de eso se encargan el programa de creación de facturas electrónicas que utilice el proveedor y el programa de gestión contable que utilice la administración destinataria, respectivamente .
Este repositorio pretende ir recopilando poco a poco la nueva documentación relacionada con eFACT conforme ésta es vaya actualizando.


## Formato factura-e 

Conforme a la Ley 25/2013, de 27 de diciembre, de impulso de la factura electrónica y creación del registro  contable de facturas en el Sector Público, las facturas que se remitan a las Administraciones Públicas serán electrónicas y se ajustarán al formato estructurado de factura electrónica Facturae versión 3.2.x con firma electrónica XAdES. 
Por tanto, el formato admitido por la plataforma es Facturae, concretamente las versiones: 3.2, 3.2.1 y 3.2.2.
Tiene más información sobre dicho formato en: 

https://www.facturae.gob.es/formato/Paginas/version-3-2.aspx

La firma XAdES de la factura permitida se recoge  y detalla en el siguiente link : 

https://www.facturae.gob.es/formato/Paginas/politicas-firma-electronica.aspx

El certificado usado para la firma podrá ser el certificado del emisor de la factura o bien el certificado de un Tercero de confianza en modalidad de firma delegada. 


## Flujo de vida de una factura

El siguiente cuadro muestra el flujo de vida natural de una factura enviada por el proveedor al servicio eFACT:



![imagen](https://user-images.githubusercontent.com/92558339/144375971-5bdaf038-fe4a-4685-a99d-b1c01ee09b7f.png)



 
Durante el proceso de registro y aprobación de la factura se puede producir el rechazo de la factura , bien por problemas de forma o contenido en la factura. En ese momento la factura pasara a estado Rechazada
Nombre	Estado Público- :BOE A - 2014 -10660	eFACT	Descripción 
Rechazada	2600	REJECTED	El portal general de entrada de facturas no ha podido identificar al destinatario La oficina contable o la unidad tramitadora han rechazado la factura, se debe indicar al proveedor el motivo del rechazo.
La validación semántica por parte de la oficina contable es negativa
La validación comercial por parte de la unidad tramitadora es negativa
El flujo de anulación   (*) permite al proveedor solicitar la Anulación de la factura . Requiere la aprobación o rechazo por parte del receptor de esta anulación para que sea efectiva. Se recoge en los siguientes estados: 
Nombre	Estado Público- :BOE A - 2014 -10660	eFACT	Descripción 
Solicitada Anulación	4200	eFACT	El proveedor solicita anulación de la factura electrónica informando también del motivo.
Aceptada Anulación	4300		La unidad tramitadora acepta la solicitud de anulación de la factura electrónica. Cambia automáticamente el estado de tramitación de una factura a Anulada-3100 en el flujo de tramitación
Rechazada Anulación	4400		La unidad tramitadora rechaza la solicitud de anulación de la factura electrónica.
El proveedor recibirá el estado Anulada

 

# Documentación para integradores

En este apartado podéis encontrar la guía de integración via WS para los diferentes servicios web proporcionados por eFact:

[Integración WS para proveedores de facturas](/ws-proveedores/README.md)

# Enlaces de interés

A continuación podéis encontrar una serie de enlaces a sitios de interés:

[Portal de soporte de eFact](https://www.aoc.cat/portal-suport/efact-empreses-base-coneixement/)

[Relación de cambios en la documentación](/CHANGELOG.md)
