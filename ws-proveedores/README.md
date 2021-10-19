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

El único protocolo admitido en los servicios web es TLS>=1.2.

#	Operaciones

# Casos de prueba para cada operación

# Como darse de alta en el servicio

# Entornos

*URL de los entornos (de los WSDL y del servicio propiamente)*

*Certificados públicos con los que se firmará la respuesta de cada entorno*
