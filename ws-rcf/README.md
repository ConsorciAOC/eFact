# Integració vía WS per Registres Comptables de Factures (RCF)

# Introducció
El presente documento pretende describir el API REST para la integración de las plataformas receptoras con el hub de eFACT, con el objetivo de sustituir a la actual integración por FTP.

# Mètode d'autenticació
## Conectividad
Se va a establecer un sistema de filtros de IPs por origen, de forma que sólo se permita el acceso al servicio desde IPs cuyo origen se encuentre dentro de los admitidos.

## Autenticacion y autorizacion
La autenticación se realizará mediante el uso de tokens JWT. Estos tokens contienen toda la información necesaria para realizar las tareas de autenticación y autorización del peticionario. 
Los campos requeridos en los tokens JWT serían los siguientes:

**•	iss:** este campo (Issuer) establece el emisor del token y en él habrá que informar el código o nombre de usuario que identifica al integrador. Este dato será asignado por el servicio de soporte en el proceso de alta o migración de la plataforma receptora.

**•	aud:** este campo (Audience) establece el servicio al que va dirigido lo token. De este modo se evita el uso indebido de tokens generados para otros servicios. Este campo tendrá que contener el literal ‘efact’.

**•	nbf**: este campo (Not Before) contiene la fecha, expresada en formato epoch con precisión de segundos, a partir de la que el token entrará en vigor. Este mecanismo se prevé para poder emitir tokens que empezarán a ser válidos en un instante futuro. Normalmente, este campo contendrá la fecha actual.

**•	iat:** este campo (Issued At) contiene la fecha de emisión, en formato epoch con precisión de segundos, del token.

**•	exp:** Este campo (Expires At) contiene la fecha de expiración, en formato epoch con precisión de segundos, del token. Es recomendable emitir los tokens con una vigencia de unos pocos segundos para evitar un uso indebido en caso de robo de estos. El tiempo máximo de expiración permitido es de 300 segundos (5 minutos).

Una vez rellenado el token, éste se tendrá que firmar con el algoritmo HMAC-SHA256, usando una clave secreta que será asignada por el servicio de soporte en el proceso de alta o migración de la plataforma receptora. Esta clave tendrá que ser custodiada de forma segura por parte del integrador y no tendrá que viajar nunca dentro de una petición o cabecera HTTP.

A continuación, se muestra un ejemplo de token JWT en claro, antes de codificar en Base64:

![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/c8b757cd-815f-4d07-950b-86e2c011a183)


Una vez generado el token, este se incluirá a la cabecera HTTP ‘Authorization’ de la siguiente manera:

Authorization: Bearer 
![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/5faf7b0f-22ac-4f6f-96ba-1cbc0a9c86ef)


En el ejemplo de cabecera HTTP con el token JWT se muestran los diferentes segmentos del token pintados de diferente color. Cómo se puede observar, los segmentos están separado por un punto.

# Operacions

El servicio devolverá alguno de los siguientes códigos de estado de respuesta HTTP:

- **200:** petición realizada satisfactoriamente.
- **400:** petición incorrecta (por ejemplo, error en los parámetros de entrada).
- **401:** petición no autorizada.
- **404:** recurso no encontrado.
- **500:** otros errores

## Listado Errores

En caso de error, el servicio devolverá un fichero de tipo “application/json” con el siguiente contenido:

```json
{
	"codiError": "9999",
	"descripcioError": "Descripción del error"
}
```

A continuación, se detallan los posibles errores que puede devolver el servicio:

### Errores de autenticación (HttpStatus 401):

- **1001:** No s'ha especificat un token d’autenticació
- **1002:** No s'ha especificat un usuari
- **1003:** Usuari no vàlid
- **1004:** No s'ha especificat una data d'expiració del token
- **1005:** No s'ha especificat una data de creació del token
- **1006:** No s'ha especificat una data d'activació del token
- **1007:** No s'ha especificat el camp audience del token
- **1008:** El token ha expirat
- **1009:** El token encara no pot ser utilitzat
- **1010:** El temps d'expiració del token és superior al permès
- **1011:** El camp audience especificat no és vàlid

### Errores de recurso no encontrado (HttpStatus 404):

- **2001:** No s'ha trobat la factura especificada
- **2002:** No s'ha trobat el document adjunt especificat
- **2003:** No s'ha trobat un historico d'estats per a la factura

### Errores de validación (HttpStatus: 400):

- **3001:** L'estat proporcionat no és vàlid
- **3002:** Per a actualitzar l'estat de la factura a ANNOTATED, és necessari especificar un número de registre
- **3003:** Per a actualitzar l'estat de la factura a REJECTED, és necessari especificar un motiu de rebuig
- **3004:** Per a actualitzar l'estat de la factura a PAID, és necessari especificar una data de pagament
- **3005:** La factura especificada no té número de registre
- **3006:** La data de pagament especificada no compleix el format esperat
- **3007:** No és possible actualitzar la factura especificada

### Errores genéricos (HttpStatus 500):

- **9001:** S'ha produït un error intern de connexió amb la base de dades
- **9002:** S'ha produït un error en la generació del rebut electrònic de la factura
- **9999:** S'ha produït un error inesperat en l'execució de l'operació sol·licitada

## 1. Consulta de facturas pendientes

Esta operación permite obtener las facturas pendientes de descargar para la plataforma asociada al usuario que realiza la petición. 
Devuelve un máximo de 500 facturas. De forma opcional se permitirá filtrar por el NIF de la entidad y/o por el código de oficina contable.

**Path relativo de la operación:** /factures-pendents

### **Petición**

parámetro|descripción| 
---------|-----------|
**nif:**| Parámetro opcional. NIF de la entidad asociada a la plataforma correspondiente al usuario que realiza la petición para el que se quieren obtener las facturas pendientes de descarga.
**oficinaComptable:** |Parámetro opcional. Código DIR3 de oficina contable asociado a la plataforma correspondiente al usuario que realiza la petición para el que se quieren obtener las facturas pendientes de descarga.

**Ejemplo petición:**

   GET [urlServicio]/factures-pendents?nif=XXXXXXXX
   
   GET [urlServicio]/factures-pendents?oficinaComptable=ZZZZZZZZZ
   
   GET [urlServicio]/factures-pendents?nif=XXXXXXXX&oficinaComptable=ZZZZZZZZZ

### **Respuesta**

Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/json” con el siguiente contenido (Ejemplo respuesta):

```json
{
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
}
```

## 2. Consulta de los datos de una factura

Esta operación permite obtener los datos de la factura correspondiente al identificador de factura especificado como parámetro. Si la factura tiene documentos adjuntos asociados, también se devuelven sus datos.

**Path relativo de la operación:** /factura/:id

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura que se quiere consultar.

**Ejemplo petición:**

   GET [urlServicio]/factura/12345
   
   
### **Respuesta**

Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/json” con el siguiente contenido:

```json
{
	"numeroFactura": "F2310-001",
	"dataFactura": "2023-10-25",
	"importFactura": 1542.75,
	"nifProveidor": "ESR0599999J",
	"nomProveidor": "Entidad de pruebas",
	"nif": "ESQ1111112G",
	"nom": "ENS FORMACIO B",
	"numeroRegistre": "E2023000064",
	"dataRegistre": "2023-10-25T16:25:02.000+02:00",
	"estat": "REJECTED",
	"dataEstat": "2023-10-26T12:03:54+02:00",
	"codiMotiuRebuig": "2134",
	"descripcioMotiuRebuig": "Descipcio Motiu",
	"adjunts": [
		{
			"idAdjunt": "159731005",
			"nom": "1.pdf"
		}
	]
}
```

## 3. Obtención del fichero de una factura

Esta operación permite obtener el fichero de la factura correspondiente al identificador de factura especificado como parámetro.

**Path relativo de la operación:** /factura/:id/facturae

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura a descargar.

**Ejemplo petición:**

   GET [urlServicio]/factura/12345/facturae
   
   
### **Respuesta**

Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/xml” correspondiente a la factura solicitada.

## 4. Obtención del fichero de un documento adjunto

Esta operación permite obtener el fichero del documento adjunto correspondiente al identificador de adjunto y al identificador de factura especificados como parámetros.

**Path relativo de la operación:** /factura/:id/adjunts/:idAdjunt

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura a la que está asociado el documento adjunto.
**idAdjunt:**| identificador del documento adjunto a descargar.

**Ejemplo petición:**

   GET [urlServicio]/factura/12345/adjunts/12348
   
   
### **Respuesta**

   Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá el fichero del documento adjunto solicitado. En la cabecera Content-type se especificará el tipo mime correspondiente.
             

## 5. Obtención del recibo electrónico de una factura

Esta operación permite obtener el recibo electrónico correspondiente al identificador de factura especificado como parámetro.

**Path relativo de la operación:** /factura/:id/rebut

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura de la que quiere obtener su recibo electrónico.


**Ejemplo petición:**

   GET [urlServicio]/factura/12345/rebut
   
   
### **Respuesta**

  Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/pdf” correspondiente al recibo electrónico de la factura especificada como parámetro.            

## 6. Obtención del histórico de estados de una factura

Esta operación permite obtener el histórico de estados correspondiente al identificador de factura especificado como parámetro.

**Path relativo de la operación:** /factura/:id/estats

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura de la que quiere obtener su histórico de estados.


**Ejemplo petición:**

   GET [urlServicio]/factura/12345/estats
   
   
### **Respuesta**

 Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/json” con el siguiente contenido:  

```json
{
	"estats": [
		{
			"estat": "REGISTERED",
			"codiEstat": "1200",
			"dataEstat": "2023-10-25T16:25:06+02:00",
			"numeroRegistre": "E2023000064",
			"dataRegistre": "2023-10-25T16:25:02.000+02:00"
		},
		{
			"estat": "REJECTED",
			"codiEstat": "2600",
			"dataEstat": "2023-10-26T12:03:05+02:00",
			"codiMotiuRebuig": "2134",
			"descripcioMotiuRebuig": "Descipcio Motiu"
		}
	]	
}
```

En cuanto a la fecha de pago de una factura (dataPagament), para los casos de facturas anteriores a la integración por este API REST en los que no se disponga de una fecha de pago concreta informada por el receptor, se considerará como fecha de pago la misma fecha de estado (dataEstat) del estado “pagada” (PAID) correspondiente.

## 7. Actualización de estados de una factura

Esta operación tiene dos funciones:

•	Actualizar el estado de la factura correspondiente al identificador especificado como parámetro según los datos especificados. 

•	Además, en caso de que el estado al que actualizar la factura sea DELIVERED o ANNOTATED, la factura correspondiente al identificador especificado como parámetro se actualizará como descargada, de forma que ya no sea tenida en cuenta por la operación de consulta de facturas pendientes (GET [urlServicio]/factures-pendents).


Restricciones:

•	No se aceptará el estado REGISTERED. Sólo se aceptarán los siguientes estados: DELIVERED, ANNOTATED, RECEIVED, ACCEPTED, RECOGNISED, PAID y REJECTED. Si se informa un estado diferente, se devolverá un error indicando que la actualización de estado especificada no está permitida.

•	Al actualizar al estado ANNOTATED será obligatorio informar el número de registro contable (numeroRegistreRCF).

•	Al actualizar al estado REJECTED será obligatorio informar del motivo de rechazo: descripcioMotiuRebuig. El dato codiMotiuRebuig es opcional y cada plataforma puede informarlo libremente siguiendo su propia codificación.

•	Al actualizar al estado PAID será obligatorio informar la fecha de pago correspondiente (dataPagament). Esta fecha debe especificarse siguiendo el patrón YYYY-MM-DD (ejemplo: 2023-09-15).

•	Lo estados que deben informar las entidades de forma obligatoria son: ANNOTATED, RECOGNISED, PAID y, en caso de rechazo, REJECTED.


**Path relativo de la operación:** /factura/:id

### **Petición**

parámetro|descripción| 
---------|-----------|
**id:**| identificador de la factura de la que quiere obtener su histórico de estados.


**Ejemplo petición:**

  PATCH [urlServicio]/factura/12345

A continuación, se indican todos los posibles datos susceptibles de ser informados. 

```json
{
	"estat": "",
	"codiMotiuRebuig ": "",
	"descripcioMotiuRebuig ": "",
	"numeroRegistreRCF": "", 
	"dataPagament": ""
}
```

Todos los campos son opcionales, excepto el campo “estat” y los especificados anteriormente como obligatorios para un determinado estado:

•	"numeroRegistreRCF": sólo es obligatorio para el estado ANNOTATED.

•	"descripcioMotiu": sólo es obligatorio para el estado REJECTED.

•	"dataPagament". sólo es obligatorio para el estado PAID.


Ejemplo de fichero JSON para actualizar una factura a rechazada (REJECTED):

```json
{
	"estat": "REJECTED",
	"codiMotiuRebuig ": "E01",
	"descripcioMotiuRebuig ": “No s'ha especificat el número d'expedient"
}
```

Ejemplo de fichero JSON para actualizar una factura a registrada en RCF (ANNOTATED):

```json
{
	"estat": "ANNOTATED ",
	"numeroRegistreRCF": "RCF-2023-89584"
}
```

Ejemplo de fichero JSON para actualizar una factura a reconocida la obligación del pago (RECOGNISED):

```json
{
	"estat": "RECOGNISED "
}
```

Ejemplo de fichero JSON para actualizar una factura como pagada (PAID):

```json
{
	"estat": "PAID",
	"dataPagament": "2023-11-30"
}
```
   
### **Respuesta**

Si la petición se ha llevado a cabo con éxito, se devuelve un JSON con los datos de la factura actualizada. La estructura de este JSON sería exactamente la misma que la especificada en el apartado 2.3 para la operación “Consulta de los datos de una factura”.


## 8. Borrado de adjuntos pendientes de descarga

Esta operación permite actualizar como descargado el adjunto especificado, de forma que ya no sea tenido en cuenta por la operación de consulta de adjuntos pendientes de descarga (GET [urlServicio]/adjuntspendents).

**Path relativo de la operación:** /adjunts-pendents/:idAdjunt

### **Petición**

parámetro|descripción| 
---------|-----------|
**idAdjunt:**| identificador del documento adjunto que se quiere marcar como descargado.


**Ejemplo petición:**

DELETE [urlServicio]/adjunts-pendents/2755


   
### **Respuesta**

Si la petición se ha llevado a cabo con éxito, simplemente se devuelve el código HTTP “200”.

## 9. Consulta de las entidades adheridas a una plataforma

Esta operación permite obtener los datos principales de las entidades adheridas a la plataforma asociada al usuario que realiza la petición.

**Path relativo de la operación:** /ens

### **Petición**

GET [urlServicio]/ens

   
### **Respuesta**

Si la petición se ha llevado a cabo con éxito (código HTTP “200”) se devolverá un fichero de tipo “application/json” con el siguiente contenido:

```json
{
	"ens": [
		{
			"nif": "ESQ1111112G",
			"nom": "ENS FORMACIO B",
			"ine10": "7996100002"
		}
	]
}
```

# Como donar-se d'alta al servei

# Entorns
A continuación, se indica la URL base del servicio en función del entorno:

•	TEST: https://efact-pre.aoc.cat/rcf 

•	PRO:  https://efact.aoc.cat/rcf
