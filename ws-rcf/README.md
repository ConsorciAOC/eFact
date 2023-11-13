# Integració vía WS per Registres Comptables de Factures (RCF)

# Introducció
El presente documento pretende describir el API REST para la integración de las plataformas receptoras con el hub de eFACT, con el objetivo de sustituir a la actual integración por FTP.

# Mètode d'autenticació
## Conectividad
Se va a establecer un sistema de filtros de IPs por origen, de forma que sólo se permita el acceso al servicio desde IPs cuyo origen se encuentre dentro de los admitidos.

## Autenticacion y autorizacion
La autenticación se realizará mediante el uso de tokens JWT. Estos tokens contienen toda la información necesaria para realizar las tareas de autenticación y autorización del peticionario. 
Los campos requeridos en los tokens JWT serían los siguientes:
•	iss: este campo (Issuer) establece el emisor del token y en él habrá que informar el código o nombre de usuario que identifica al integrador. Este dato será asignado por el servicio de soporte en el proceso de alta o migración de la plataforma receptora.

**•	aud:** este campo (Audience) establece el servicio al que va dirigido lo token. De este modo se evita el uso indebido de tokens generados para otros servicios. Este campo tendrá que contener el literal ‘efact’.

**•	nbf**: este campo (Not Before) contiene la fecha, expresada en formato epoch con precisión de segundos, a partir de la que el token entrará en vigor. Este mecanismo se prevé para poder emitir tokens que empezarán a ser válidos en un instante futuro. Normalmente, este campo contendrá la fecha actual.

**•	iat:** este campo (Issued At) contiene la fecha de emisión, en formato epoch con precisión de segundos, del token.

**•	exp:** Este campo (Expires At) contiene la fecha de expiración, en formato epoch con precisión de segundos, del token. Es recomendable emitir los tokens con una vigencia de unos pocos segundos para evitar un uso indebido en caso de robo de estos. El tiempo máximo de expiración permitido es de 300 segundos (5 minutos).

Una vez rellenado el token, éste se tendrá que firmar con el algoritmo HMAC-SHA256, usando una clave secreta que será asignada por el servicio de soporte en el proceso de alta o migración de la plataforma receptora. Esta clave tendrá que ser custodiada de forma segura por parte del integrador y no tendrá que viajar nunca dentro de una petición o cabecera HTTP.

A continuación, se muestra un ejemplo de token JWT en claro, antes de codificar en Base64:
![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/4d59a27d-6e1a-454d-9c24-ef788895f90f)

Una vez generado el token, este se incluirá a la cabecera HTTP ‘Authorization’ de la siguiente manera:

Authorization: Bearer 
![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/47bb7a04-9607-4942-b539-45b81043d10d)

En el ejemplo de cabecera HTTP con el token JWT se muestran los diferentes segmentos del token pintados de diferente color. Cómo se puede observar, los segmentos están separado por un punto.
# Operacions

El servicio devolverá alguno de los siguientes códigos de estado de respuesta HTTP:

**•	200:** petición realizada satisfactoriamente.

**•	400:** petición incorrecta (por ejemplo, error en los parámetros de entrada).

**•	401:** petición no autorizada.

**•	404:** recurso no encontrado.

**•	500:** otros errores


## Listado Errores

En caso de error, el servicio devolverá un fichero de tipo “application/json” con el siguiente contenido:

`{
   "codiError": "9999",
   
   " descripcioError": "Descripción del error"
}`

A continuación, se detallan los posibles errores que puede devolver el servicio:

### Errores de autenticación (HttpStatus 401):

**o	1001:** No s'ha especificat un token d’autenticació

**o	1002:** No s'ha especificat un usuari

**o	1003:** Usuari no vàlid

**o	1004:** No s'ha especificat una data d'espiració del token

**o	1005:** No s'ha especificat una data de creació del token

**o	1006:** No s'ha especificat una data d'activació del token

**o	1007:** No s'ha especificat el camp audience del token

**o	1008:** El token ha expirat

**o	1009:** El token encara no pot ser utilitzat

**o	1010:** El temps d'expiració del token és superior al permès

**o	1011:** El camp audience especificat no és vàlid


### Errores de recurso no encontrado (HttpStatus 404):

**o	2001:** No s'ha trobat la factura especificada

**o	2002:** No s'ha trobat el document adjunt especificat

**o	2003:** No s'ha trobat un historico d'estats per a la factura


### Errores de validación (HttpStatus: 400):

**o	3001:** L'estat proporcionat no és vàlid

**o	3002:** Per a actualitzar l'estat de la factura a ANNOTATED, és necessari especificar un número de registre

**o	3003:** Per a actualitzar l'estat de la factura a REJECTED, és necessari especificar un motiu de rebuig

**o	3004:** Per a actualitzar l'estat de la factura a PAID, és necessari especificar una data de pagament

**o	3005:** La factura especificada no té número de registre

**o	3006:** La data de pagament especificada no compleix el format esperat

**o	3007:** No és possible actualitzar la factura especificada


### Errores genéricos (HttpStatus 500):

**o	9001:** S'ha produït un error intern de connexió amb la base de dades

**o	9002:** S'ha produït un error en la generació del rebut electrònic de la factura

**o	9999:** S'ha produït un error inesperat en l'execució de l'operació sol·licitada


## 1. Consulta de facturas pendientes

Esta operación permite obtener las facturas pendientes de descargar para la plataforma asociada al usuario que realiza la petición. 
Devuelve un máximo de 500 facturas. De forma opcional se permitirá filtrar por el NIF de la entidad y/o por el código de oficina contable.

**Path relativo de la operación: ** /factures-pendents

###**Petición**

parámetro|descripción| 
---------|-----------|
**nif:**| Parámetros opcional.NIF de la entidad asociada a la plataforma correspondiente al usuario que realiza la petición para el que se quieren obtener las facturas pendientes de descarga.
**oficinaComptable:** |Parámetros opcional.código de oficina contable asociado a la plataforma correspondiente al usuario que realiza la petición para el que se quieren obtener las facturas pendientes de descarga.

Ejemplo petición:

   GET [urlServicio]/factures-pendents?nif=XXXXXXXX
   
   GET [urlServicio]/factures-pendents?oficinaComptable=ZZZZZZZZZ
   
   GET [urlServicio]/factures-pendents?nif=XXXXXXXX&oficinaComptable=ZZZZZZZZZ

###**Respuesta**

Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/json” con el siguiente contenido (Ejemplo respuesta):


   `{
     "mesFactures": false,  
  
     "factures": [
   
        {
        
            "id": "159732145",
            
            "nif": "ESQ1111112G",
            
            "oficinaComptable": "A987654321",
            
            "organGestor": "A987654321",
            
            "unitatTramitadora": "A987654321"
            
        },
        
        {
        
            "id": "159732146",
            
            "nif": "ESQ1111112G",
            
            "oficinaComptable": "A987654321",
            
            "organGestor": "A987654321",
            
            "unitatTramitadora": "A987654321"
            
        }
        
                  ]
     
       }`



# Como donar-se d'alta al servei

# Entorns
A continuación, se indica la URL base del servicio en función del entorno:

•	TEST: https://efact-pre.aoc.cat/rcf 

•	PRO:  https://efact.aoc.cat/rcf
