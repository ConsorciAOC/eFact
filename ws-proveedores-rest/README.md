# Integración vía WS para Proveedores

1. [Introducció](#introducció)
2. [Mètode d'autenticació](#mètode-dautenticació)
	1. [Connectivitat](#connectivitat)
	2. [Autenticació i autorització](#autenticació-i-autorització)
3. [Entorns](#entorns)
4. [Operacions](#operacions)
	1. [Enviament d'una factura](#enviament-duna-factura)
 	2. [Consulta de les dades d'una factura](#consulta-de-les-dades-duna-factura)
  	3. [Consulta de l'històric d'estats d'una factura](#consulta-de-lhistòric-destats-duna-factura)
   	4. [Obtenció del rebut electrònic de factura](#obtenció-del-rebut-electrònic-de-factura)
   	5. [Consulta d'estats pendents de descàrrega](#consulta-destats-pendents-de-descàrrega)
   	6. [Eliminació d'estats pendents de descàrrega](#eliminació-destats-pendents-de-descàrrega)
   	7. [Consulta de les entitats adherides a eFACT](#consulta-de-les-entitats-adherides-a-efact)
   	8. [Consulta del detall d'una entitat eFACT](#consulta-del-detall-duna-entitat-efact)
5. [Definició dels objectes de resposta](#definició-dels-objectes-de-resposta)


# Introducció
Aquest document pretén descriure l'API REST per a la integració de les plataformes emissores amb el hub d'eFACT, amb l'objectiu de substituir la integració actual per FTP i al web service SOAP de proveïdors.

# Mètode d'autenticació
## Connectivitat
S'establirà un sistema de filtres d'IPs per origen geogràfic, de manera que només es permeti l'accés al servei des d'IPs l'origen geogràfic de les quals estigui dins dels admesos. Per exemple, Rússia, Iran o Xina serien orígens geogràfics no permesos.

## Autenticació i autorització

L'autenticació es farà mitjançant l'ús de tokens JWT. Aquests tokens contenen tota la informació necessària per fer les tasques d'autenticació i autorització del peticionari. Els camps necessaris als tokens JWT seran els següents:

- **iss:** aquest camp (Issuer) estableix l'emissor del token i cal informar-hi el codi o nom d'usuari que identifica l'integrador. Aquesta dada serà assignada pel servei de suport al procés d'alta o migració de la plataforma emissora.

- **aud:** aquest camp (Audience) estableix el servei al qual va dirigit el token. D'aquesta manera s'evita l'ús indegut de tokens generats per a altres serveis. Aquest camp haurà de contenir el literal "efact".

- **nbf**: aquest camp (Not Before) conté la data, expressada en format *epoch* amb precisió de segons, a partir de la qual el token entrarà en vigor. Aquest mecanisme es preveu per poder emetre tokens que començaran a ser vàlids en un instant futur. Normalment aquest camp contindrà la data actual.

- **iat:** aquest camp (Issued At) conté la data d'emissió, en format *epoch* amb precisió de segons, del token.

- **exp:** Aquest camp (Expires At) conté la data d'expiració, en format *epoch* amb precisió de segons, del token. És recomanable emetre els tokens amb una vigència d'uns pocs segons per evitar-ne un ús indegut en cas de robatori. El temps màxim d'expiració permès és de 300 segons (5 minuts).

Un cop emplenat el token, aquest s'haurà de signar amb l'algorisme HMAC-SHA256, utilitzant una clau secreta que serà assignada pel servei de suport en el procés d'alta o migració de la plataforma emissora. Aquesta clau haurà de ser custodiada de manera segura per part de l'integrador i no haurà de viatjar mai dins d'una petició o capçalera HTTP.

A continuació, es mostra un exemple de token JWT en clar, abans de codificar a Base64:

![A continuació hi ha un desplegable amb l'explicació de descripció de la imatge](https://github.com/ConsorciAOC/eFact/assets/92558339/c5f67b58-363b-43e1-a1fe-6d15d778f1fa)

<details>
  <summary><i>Explicació textual de la imatge</i></summary>

### SEGMENT I
L'algorisme de signatura emprat per protegir el token, així com el tipus de token. Només es suportarà l'algoritme HS256 i el tipus de token JWT.

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

### SEGMENT II
Dades d'autenticació i autorització comentades anteriorment.

```json
{
    "iss": "hub_0012",
    "aud": "efact",
    "iat": 1516239022,
    "nbf": 1516239022,
    "exp": 1516239042
}
```

### SEGMENT III
Signatura del token, necessària per verificar que no ha estat alterat, així com la seva autenticitat.

```
3oGx4zGJy7YfYTIiBiPNBPCUGrC8oNlKQaGHSO4D5M0
```

### Notes

Els segments van delimitats per punts, de manera que trobem el primer segment, un punt, el segon segment, un punt i el tercer segment.

</details>

Un cop generat el token, aquest s'inclourà a la capçalera HTTP 'Authorization' de la següent manera:

![A continuació hi ha un desplegable amb l'explicació de descripció de la imatge](https://github.com/ConsorciAOC/eFact/assets/92558339/5faf7b0f-22ac-4f6f-96ba-1cbc0a9c86ef)

<details>
  <summary><i>Explicació textual de la imatge</i></summary>

La cadena que hi ha a continuació és una concatenació dels tres segments un cop codificat a Base64

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJBUFawMDAxIiwic3ViIjoiOTgyMTkyMDAwMiIsInVzZXIiOiIxMTExMTEwMTExMUgiLCJvYW1pIjoiSm9zZXZA9zZXAgSm9zZXAgSm9zZXAiOjE1MTYyMzkwMjIsIm5iZiI6MTUxMTUxOTAyMiwiZXhwIjoxNTE2MjM5MDQyfQ.3oGx4zGJy7YfYTIiBiPNBPCUGrC8oNlKQaGHSO4D5M0
```


</details>

A l'exemple de capçalera HTTP amb el token JWT es mostren els diferents segments del token pintats de diferent color. Com es pot observar, els segments estan separats per un punt.


# Entorns

A continuació, s'indica l'URL base del servei segons l'entorn:
- **TEST**: https://efact-pre.aoc.cat/proveidors
- **PRO**:  https://efact.aoc.cat/proveidors


# Operacions

## Enviament d'una factura
Esta operación permite el envío de una factura y, opcionalmente, de sus documentos adjuntos al servicio e-FACT. En la propia petición se informarán tanto la factura como sus posibles documentos adjuntos, teniendo en cuenta las siguientes restricciones:
-	Sólo se puede informar un máximo 5 documentos adjuntos.
-	El tamaño de la petición no puede ser superior a 10 MB.

**Path relatiu de l'operació:** /factura

### **Petició**

*POST [urlServicio]/factura*

A continuación, se indican todos los posibles datos susceptibles de ser informados en el JSON de petición de envío de una factura:
- **correuElectronic:** dirección de correo electrónico en la que recibir las notificaciones con los cambios de estado de la factura. Este dato es opcional.
- **face:** atributo de tipo booleano para indicar si hay que entregar la factura a FACe. Esta opción de envío sólo podrá ser utilizada por emisores que sea entidades públicas catalanas registradas como receptores en e-FACT. Este dato es opcional y, en caso de no informarse, tomará el valor false. 
- **factura:** información de la factura a enviar.
	- **nom:** nombre del fichero de factura.
	- **contingut:** contenido del fichero de factura codificado en base64. El tipo mime de la factura debe ser application/xml.
- **adjunts:** colección con los datos de los documentos adjuntos a enviar junto con la factura. Por cada documento adjunto se especificarán los siguientes datos:
	- **nom:** nombre del fichero adjunto.
	- **mime:** tipo mime del fichero adjunto. En el anexo 4.2 se indican los tipos mime admitidos.
 	- **contingut:** contenido del fichero adjunto codificado en base64.

Ejemplo de petición de envío de factura: 

```json
{
	"correuElectronic": "a@a.com",
	"face": false,
	"factura": {
		"nom”: "F24-00182.xml",
		"contingut": "PD94bWwgdmVi9MLTAwVMHls8sDADSQBAyc2lvlOkZhY3R1cRQ1Cl5DrgkmFlPg…"
	},
	"adjunts": [
		{
			"nom": "adjunt1.pdf",
			"mime": "application/pdf",
			"contingut": "UEsDBBQAAAAIALuJNVCiVMHls8sDADSQBAALAAAAYXR0YWNoMS5wZGbMWQVQG8w…"
		}
	]
}
```

### **Resposta**

Si la petición se ha llevado a cabo con éxito (código HTTP "200") se devolverá un objeto Factura (fichero de tipo `application/json`), cuyo contenido se detalla en el apartado 4.1 de este documento. A continuación, se muestra un ejemplo de respuesta:

```json
{
	"correuElectronic": "a@a.com",
	"face": false,
	"id": "159732145",
	"dataRecepcio": "2024-11-05T12:03:54+01:00",
	"numero": "F2411-0018",
	"serie": "A",
	"dataExpedicio": "2024-11-05",
	"import": 1542.75,
	"proveidor": {
		"nif": "ESR0599999J",
		"nom": "Proveïdor de pruebas"
	},
	"receptor": {
		"nif": "ESQ1111112G",
		"nom": "ENS FORMACIO B",
		"dir3": {
			"oficinaComptable": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
			},
			"organGestor": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
			},
			"unitatTramitadora": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
			}
		}
	},
	"estat": {
		"codi": "SENT",
		"codiNumneric": "1000",
		"data": "2024-11-05T12:03:54+01:00"
	},
	"adjunts": [
		{
			"nom": "adjunt1.pdf",
			"mime": "application/pdf"
		}
	]
}
```


## Consulta de les dades d'una factura

Esta operación permite obtener los datos de la factura correspondiente al identificador de factura especificado como parámetro. Si la factura tiene documentos adjuntos asociados, también se devuelven sus datos.

**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de la factura que es vol consultar.

*GET [urlServicio]/factura/159732145*

### **Resposta**

```json
{
	"correuElectronic": "a@a.com",
	"face": false,
	"id": "159732145", 
	"dataRecepcio": "2024-11-05T12:03:54+01:00",
	"numero": "F2411-0018",
	"serie": "A",
	"dataExpedicio": "2024-11-05",
	"import": 1542.75,
	"proveidor": {
		"nif": "ESR0599999J",
		"nom": "Proveïdor de pruebas"
	},
	"receptor": {
		"nif": "ESQ1111112G",
		"nom": "ENS FORMACIO B",
		"dir3": {
			"oficinaComptable": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
         	},
			"organGestor": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
			},
			"unitatTramitadora": {
				"codi": "A987654321",
				"nom": "ENS FORMACIO B"
			}
		}
	},
	"registre": {
		"numero": "E2023000064",
		"data": "2024-11-05T12:04:02.000+01:00"
	},
	"motiuRebuig": {
		"codi": "DF01",
		"descripcio": "Descipcio Motiu"
	},
	"estat": {
		"codi": "REJECTED",
		"codiNumneric": "2600",
		"data": "2024-11-18T10:03:54+01:00"
	},
	"adjunts": [
		{
			"nom": "1.pdf",
			"mime": "application/pdf"
		}
	]
}
```


## Consulta de l'històric d'estats d'una factura

Esta operación permite obtener todos los estados por lo que ha pasado la factura correspondiente al identificador de factura especificado como parámetro.

**Path relatiu de l'operació:** /historicEstatsFactura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de la factura que es vol consultar.

*GET [urlServicio]/historicEstatsFactura/159732145*

Si la petición se ha llevado a cabo con éxito (código HTTP "200") se devolverá un objeto *HistoricEstatsFactura* (fichero de tipo `application/json`), cuyo contenido se detalla en el apartado 4.2 de este documento. A continuación, se muestra un ejemplo de respuesta:

```json
{
	"id": "159732145",
	"estats": [
		{
			"codi": "SENT",
			"codiNumeric": "1000",
			"data": "2024-11-05T12:03:54+01:00"            
		},
		{
			"codi": "REGISTERED",
			"codiNumeric": "1200",
			"data": "2024-11-05T12:04:35.000+01:00",
			"registre": {
				"numero": "E2023000064",
				"data": "2024-11-05T12:04:02.000+01:00"
			}
		}
	]
}
```


## Obtenció del rebut electrònic de factura

Aquesta operació permet obtenir el rebut electrònic corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/rebut

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de la factura de la qual es vol obtenir el rebut electrònic.

*GET [urlServei]/factura/12345/rebut*
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP "200"), es retornarà un fitxer de tipus `application/pdf` corresponent al rebut electrònic de la factura especificada com a paràmetre.


## Consulta d'estats pendents de descàrrega

Esta operación permite obtener los cambios estado pendientes de descarga para la plataforma asociada al usuario que realiza la petición. Devuelve un máximo de 500 estados. Si hubiese más estados a devolver del máximo estipulado, se informará en el atributo de tipo booleano mesEstats de la respuesta. De forma opcional se permitirá filtrar por el NIF de la entidad emisora.

**Path relatiu de l'operació:** /estats-pendents

### **Petició**

paràmetre|descripció|
---------|----------|
**nifProveidor** | Parámetro opcional. NIF de la entidad emisora, asociada a la plataforma correspondiente al usuario que realiza la petición, para la que se quieren obtener los cambios de estado pendientes de descarga.

*GET [urlServicio]/estats-pendents*

*GET [urlServicio]/estats-pendents?nifProveidor=XXXXXXXX*

### **Resposta**

Si la petición se ha llevado a cabo con éxito (código HTTP "200") se devolverá un objeto *EstatsPendents* (fichero de tipo `application/json`), cuyo contenido se detalla en el apartado 4.3 de este documento. A continuación, se muestra un ejemplo de respuesta:

```json
{
	"mesEstats": false,
	"estats": [
		{
			"id": "1597",
			"idFactura": "159732145",
			"estat": {
				"codi": "REGISTERED",
				"codiNumneric": "1200",
				"data": "2024-11-05T12:04:35.000+01:00",
				"registre": {
					"numero": "E2023000064",
					"data": "2024-11-05T12:04:02.000+01:00"
				}
			}
		}
	]
}
```

En cuanto a la fecha de pago de una factura (dataPagament), para los casos en los que no se disponga de una fecha de pago concreta informada por el receptor, se considerará como fecha de pago la propia fecha de estado (atributo data) del estado "pagada" (PAID) correspondiente.


## Eliminació d'estats pendents de descàrrega

Esta operación permite actualizar como descargado el estado especificado, de forma que ya no sea tenido en cuenta por la operación de consulta de estados pendientes de descarga (*GET [urlServicio]/estats-pendents*).

**Path relatiu de l'operació:** /estats-pendents/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de l'estat que es vol marcar com descarregat.

*DELETE [urlServicio]/estats-pendents/1602*
   
### **Resposta**

Si la petición se ha llevado a cabo con éxito, simplemente se devuelve el código HTTP "200".


## Consulta de les entitats adherides a eFACT

Esta operación permite obtener los datos principales de las entidades adheridas al servicio eFACT.

**Path relatiu de l'operació:** /receptors

### **Petició**

*GET [urlServicio]/receptors*
   
### **Resposta**

Si la petición se ha llevado a cabo con éxito (código HTTP "200") se devolverá un objeto *ReceptorsEfact* (fichero de tipo `application/json`), cuyo contenido se detalla en el apartado 4.4 de este documento. A continuación, se muestra un ejemplo de respuesta:

```json
{
	"receptors": [
		{
			"nif": "ESQ1111111I",
			"nom": "ENS FORMACIO A",
		},
		{
			"nif": "ESQ1111112G",
			"nom": "ENS FORMACIO B",
		},
		…
	]
}
```


## Consulta del detall d'una entitat eFACT

Esta operación permite obtener la información detallada (dirección, dir3, etc.) para la entidad del servicio eFACT especificada.

**Path relatiu de l'operació:** /receptors

### **Petició**

paràmetre|descripció|
---------|----------|
**nif** | NIF de l'entitat del servei eFACT per a la qual realitzar la consulta.

*GET [urlServicio]/receptors/:nif*
   
### **Resposta**

Si la petición se ha llevado a cabo con éxito (código HTTP "200") se devolverá un objeto *ReceptorEfact* (fichero de tipo `application/json`), cuyo contenido se detalla en el apartado 4.5 de este documento. A continuación, se muestra un ejemplo de respuesta:

```json
{
	"nif": "ESQ1111111I",
	"nom": "ENS FORMACIO A",
	"direccio": {
		"carrer": "Passatge de la Concepció, 11",
		"localitat": "Barcelona",
		"provincia": "Barcelona",
		"codiPostal": "08008"
	}
	"dir3": [
		{
			"oficinaComptable": {
				"codi": "A00000000",
				"nom": "ENS FORMACIO A"
			},
			"organGestor": {
				"codi": "A00000000",
				"nom": "ENS FORMACIO A"
			},
			"unitatTramitadora": {
				"codi": "A00000000",
				"nom": "ENS FORMACIO A"
			}
		}
	]
}
```


# Definició dels objectes de resposta
