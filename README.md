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

![Flujo deVida](https://user-images.githubusercontent.com/92558339/163233213-c6cc878e-fa7a-4a59-952c-7b6345b8d43e.png)

El siguiente cuadro muestra el flujo de vida natural de una factura enviada por el proveedor al servicio eFACT:

![imagen](https://user-images.githubusercontent.com/92558339/144376089-99c88b13-db66-4852-87b5-241a094f48db.png)

Durante el proceso de registro y aprobación de la factura se puede producir el rechazo de la factura , bien por problemas de forma o contenido en la factura. En ese momento la factura pasara a estado Rechazada

![imagen](https://user-images.githubusercontent.com/92558339/144376363-40f51289-9811-4bd0-8351-9d78aae217bf.png)

El flujo de anulación   (*) permite al proveedor solicitar la Anulación de la factura . Requiere la aprobación o rechazo por parte del receptor de esta anulación para que sea efectiva. Se recoge en los siguientes estados: 

![imagen](https://user-images.githubusercontent.com/92558339/144376650-5218c943-5bb0-4294-8046-ec862fbfa4d7.png)
 
# Documentación para integradores

En este apartado podéis encontrar la guía de integración via WS para los diferentes servicios web proporcionados por eFact:

[Integración WS para proveedores de facturas](/ws-proveedores/README.md)

[Integración WS para Registros Contables de Facturas](/ws-rcf/README.md)

# Enlaces de interés

A continuación podéis encontrar una serie de enlaces a sitios de interés:

[Portal de soporte de eFact](https://suport-efact-empreses.aoc.cat/hc/ca/)

[Relación de cambios en la documentación](/CHANGELOG.md)
