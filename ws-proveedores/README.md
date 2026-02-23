# Integració via API REST per Proveïdors

1. [Introducció](#introducció)
2. [Mètode d'autenticació](#mètode-dautenticació)
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
	1. [Factura](#factura)
 	2. [HistoricEstatsFactura](#historicestatsfactura)
  	3. [EstatsPendents](#estatspendents)
	4. [ReceptorsEfact](#receptorsefact)
	5. [ReceptorEfact](#receptorefact)
	6. [EstatPendent](#estatpendent)
	7. [Estat](#estat)
	8. [EstatDetallat](#estatdetallat)
	9. [RelacioDir3](#relaciodir3)
	10. [CentreAdministratiu](#centreadministratiu)
	11. [DadesRegistre](#dadesregistre)
	12. [DadesMotiuRebuig](#dadesmotiurebuig)
6. [Annexos](#annexos)
	1. [Codis de resposta del servei](#codis-de-resposta-del-servei)
	2. [Possibles estats d'una factura](#possibles-estats-duna-factura)
	3. [Tipus mime admesos](#tipus-mime-admesos)


# Introducció
Aquest document pretén descriure l'API REST per a la integració de les plataformes emissores amb el hub d'eFACT, amb l'objectiu de substituir la integració actual per FTP i al web service SOAP de proveïdors.

# Mètode d'autenticació
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
Aquesta operació permet l'enviament d'una factura i, opcionalment, dels seus documents adjunts al servei eFACT. A la mateixa petició s'informaran tant la factura com els seus possibles documents adjunts, tenint en compte les restriccions següents:
-	Només es pot informar un màxim de 5 documents adjunts.
-	La mida de la petició no pot ser superior a 10 MB.

**Path relatiu de l'operació:** /factura

### **Petició**
*POST [urlServicio]/factura*

A continuació, s'indiquen tots les possibles dades susceptibles de ser informats al JSON de petició d'enviament d'una factura:
- **correuElectronic:** adreça de correu electrònic en la qual rebre les notificacions amb els canvis d'estat de la factura. Aquesta dada és opcional.
- **face:** atribut de tipus booleà per a indicar si cal lliurar la factura a FACe. Aquesta opció d'enviament només podrà ser utilitzada per emissors que siguin entitats públiques catalanes registrades com a receptors a eFACT. Aquesta dada és opcional i, en cas de no venir informat, prendrà el valor *false*. 
- **factura:** informació de la factura a enviar.
	- **nom:** nom del fitxer de factura. L'extensió ha de ser `xml` o `xsig`.
	- **contingut:** contingut del fitxer de factura codificat en base64. El tipus mime de la factura ha de ser `application/xml`.
- **adjunts:** col·lecció amb les dades dels documents adjunts a enviar juntament amb la factura. Per cada document adjunt s'especificaran les dades següents:
	- **nom:** nom del fitxer adjunt.
	- **mime:** tipus mime del fitxer adjunt. Veure l'annex [Tipus mime admesos](#tipus-mime-admesos).
 	- **contingut:** contingut del fitxer adjunt codificat en base64.

Exemple de petició d'enviament de factura:

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
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[Factura](#factura)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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
Aquesta operació permet obtenir les dades de la factura corresponent a l'identificador de factura especificat com a paràmetre. Si la factura té documents adjunts associats, també es tornen les seves dades.

**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de la factura que es vol consultar.

*GET [urlServicio]/factura/159732145*

### **Resposta**
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[Factura](#factura)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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
Aquesta operació permet obtenir tots els estats pel que ha passat la factura corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /historicEstatsFactura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de la factura que es vol consultar.

*GET [urlServicio]/historicEstatsFactura/159732145*

### **Resposta**
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[HistoricEstatsFactura](#historicestatsfactura)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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
Si la petició s'ha dut a terme amb èxit (codi HTTP "200"), es tornarà un fitxer de tipus `application/pdf` corresponent al rebut electrònic de la factura especificada com a paràmetre.


## Consulta d'estats pendents de descàrrega
Aquesta operació permet obtenir els canvis d'estat pendents de descàrrega per a la plataforma associada a l'usuari que realitza la petició. Retorna un màxim de 500 estats. Si hi hagués més estats a tornar del màxim establert, s'indicarà en l'atribut de tipus booleà *mesEstats* de la resposta. De manera opcional es permetrà filtrar pel NIF de l'entitat emissora.

**Path relatiu de l'operació:** /estats-pendents

### **Petició**

paràmetre|descripció|
---------|----------|
**nifProveidor** | Paràmetre opcional. NIF de l'entitat emissora, associada a la plataforma corresponent a l'usuari que realitza la petició, per a la qual obtenir els canvis d'estat pendents de descàrrega.

*GET [urlServicio]/estats-pendents*

*GET [urlServicio]/estats-pendents?nifProveidor=XXXXXXXX*

### **Resposta**
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[EstatsPendents](#estatspendents)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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

Quant a la data de pagament d'una factura (*dataPagament*), per als casos en els quals no es disposi d'una data de pagament concreta informada pel receptor, es considerarà com a data de pagament la data d'estat (atribut *data*) de l'estat "pagada" (PAID) corresponent.


## Eliminació d'estats pendents de descàrrega
Aquesta operació permet actualitzar l'estat especificat com "descarregat", de manera que ja no es tingui en compte per l'operació de consulta d'estats pendents de descàrrega (*GET [urlServicio]/estats-pendents*).

**Path relatiu de l'operació:** /estats-pendents/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id** | Identificador de l'estat que es vol marcar com descarregat.

*DELETE [urlServicio]/estats-pendents/1602*
   
### **Resposta**
Si la petició s'ha dut a terme amb èxit, simplement es torna el codi HTTP "200".


## Consulta de les entitats adherides a eFACT
Aquesta operació permet obtenir les dades principals de les entitats adherides al servei eFACT.

**Path relatiu de l'operació:** /receptors

### **Petició**

*GET [urlServicio]/receptors*
   
### **Resposta**
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[ReceptorsEfact](#receptorsefact)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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
Aquesta operació permet obtenir la informació detallada (adreça, dir3, etc.) per a l'entitat del servei eFACT especificada.

**Path relatiu de l'operació:** /receptors

### **Petició**

paràmetre|descripció|
---------|----------|
**nif** | NIF de l'entitat del servei eFACT per a la qual realitzar la consulta.

*GET [urlServicio]/receptors/:nif*
   
### **Resposta**
Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un objecte *[ReceptorEfact](#receptorefact)* (fitxer de tipus `application/json`). A continuació, es mostra un exemple de resposta:

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

## Factura
A continuació, es descriuen tots els possibles atributs d'un objecte *Factura*:
- **correuElectronic:** adreça de correu electrònic informada per l'emissor a l'operació d'enviament de la factura, en la qual es rebran les notificacions amb els canvis d'estat de la factura.
- **face:** atribut de tipus booleà, informat per l'emissor a l'operació d'enviament de la factura, en el qual s'indica si la factura s'ha lliurat la factura a FACe.
- **id:** identificador assignat a la factura al servei eFACT.
- **dataRecepcio:** data de recepció de la factura al servei eFACT. Format: *YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM*.
- **numero:** número de la factura.
- **serie:** número de sèrie de la factura.
- **dataExpedicio:** data d'expedició de la factura. Format: *YYYY-MM-DD*.
- **import:** import total de la factura.
- **proveidor:** dades del proveïdor de la factura. Aquest bloc conté les dades següents:
	- **nif:** NIF del proveïdor de la factura.
	- **nom:** nom del proveïdor de la factura.
- **receptor:** dades de l'entitat receptora de la factura. Aquest bloc conté les dades següents:
	- **nif:** NIF de l'entitat receptora de la factura.
	- **nom:** nom de l'entitat receptora de la factura.
	- **dir3:** objecte *[RelacioDir3](#relaciodir3)* amb les dades de l'oficina comptable, l'òrgan gestor i la unitat tramitadora als quals va dirigida la factura.
- **registre:** objecte *[DadesRegistre](#dadesregistre)* amb les dades de registre de la factura. Aquest bloc només es tornarà en el cas que es tracti d'una factura ja registrada al sistema, és a dir, si està o ha passat per l'estat REGISTERED.
- **numeroRegistreRCF:** número de registre comptable de la factura (RCF). Aquesta dada només es tornarà quan es tracti d'una factura ja "registrada al RCF", és a dir, si està o ha passat per l'estat ANNOTATED, i el receptor ha informat correctament aquesta dada.
- **motiuRebuig:** objecte *[DadesMotiuRebuig](#dadesmotiurebuig)* amb les dades de rebuig de la factura. Aquest bloc només es tornarà quan es tracti d'una factura "rebutjada" (REJECTED).
- **dataPagament:** data en què s'ha pagat la factura. Format: *YYYY-MM-DD*. Aquesta dada només es tornarà quan es tracti d'una factura "pagada" (PAID).
- **estat:** objecte *[Estat](#estat)* amb les dades principals de l'estat actual de la factura.
- **adjunts:** col·lecció amb les dades dels documents adjunts enviats juntament amb la factura. Per cada document adjunt s'informen les dades següents:
	- **nom:** nom informat per l'emissor per al document adjunt.
	- **mime:** tipus mime informat per l'emissor per al document adjunt.

## HistoricEstatsFactura
A continuació, es descriuen tots els possibles atributs d'un objecte *HitoricEstatsFactura*:

- **id:** identificador de la factura al servei eFACT.
- **estats:** col·lecció d'objectes *[EstatDetallat](#estatdetallat)*, un per cada estat de la factura.

## EstatsPendents
A continuació, es descriuen tots els possibles atributs d'un objecte *EstatsPendents*:
- **mesEstats:** atribut booleà que indica si hi ha més estats pendents de descàrrega, a part dels retornats a l'operació actual.
- **estats:** col·lecció d'objectes *[EstatPendent](#estatpendent)*, un per cada estat pendent de descàrrega.

## ReceptorsEfact
A continuació, es descriuen tots els possibles atributs d'un objecte *ReceptorsEfact*:
- **receptors:** col·lecció d'objectes amb les dades principals de les entitats receptores del servei eFACT. A cada objecte de la col·lecció s'informen les dades següents:
	- **nif:** NIF de l'entitat receptora.
	- **nom:** nom de l'entitat receptora.

## ReceptorEfact
A continuació, es descriuen tots els possibles atributs d'un objecte *ReceptorEfact*:
- **nif:** NIF de l'entitat receptora.
- **nom:** nom de l'entitat receptora.
- **direccio:** adreça fiscal de l'entitat receptora. Aquest bloc conté les dades següents:
	- **carrer:** carrer de l'entitat receptora.
	- **localitat:** localitat de l'entitat receptora.
	- **provincia:** província de l'entitat receptora.
	- **codiPostal:** codi postal de l'entitat receptora.
- **dir3:** col·lecció d'objectes *[RelacioDir3](#relaciodir3)*, un per cada terna DIR3 associada a l'entitat receptora al directori comú de FACe.

## EstatPendent
A continuació, es descriuen tots els possibles atributs d'un objecte *EstatPendent*:
- **id:** identificador de l'estat.
- **idFactura:** identificador de la factura a la qual està associada l'estat.
- **estat:** objecte *[EstatDetallat](#estatdetallat)* amb les dades de l'estat.

## Estat
A continuació, es descriuen tots els possibles atributs d'un objecte *Estat*:
- **codi:** codi de l'estat (codi tradicional hub).
- **codiNumeric:** codi numèric FACe corresponent a l'estat (BOE A-2014-10660).
- **data:** data de l'estat. Format: *YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM*.

## EstatDetallat
A continuació, es descriuen tots els possibles atributs d'un objecte *EstatDetallat*:
- **codi:** codi de l'estat (codi tradicional hub).
- **codiNumeric:** codi numèric FACe corresponent a l'estat (BOE A-2014-10660).
- **data:** data de l'estat. Format: *YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM*.
- **registre:** objecte *[DadesRegistre](#dadesregistre)* amb les dades de registre de la factura a la qual està associada aquest estat. Aquest bloc només es tornarà quan es tracti d'un estat de factura "registrada" (REGISTERED).
- **numeroRegistreRCF:** número de registre comptable de la factura (RCF). Aquesta dada només es mostrarà quan es tracti d'un estat de factura "registrada a RCF" (ANNOTATED) i el receptor l'hagi informat correctament.
- **motiuRebuig:** objecte *[DadesMotiuRebuig](#dadesmotiurebuig)* amb les dades de rebuig de la factura. Aquest bloc només es tornarà quan es tracti d'un estat de factura "rebutjada" (REJECTED).
- **dataPagament:** data en la qual s'ha pagat la factura. Format: *YYYY-MM-DD*. Aquesta dada només es mostrarà quan es tracti d'un estat de factura "pagada" (PAID). Per als casos en els quals no es disposi d'una data de pagament concreta informada pel receptor, es considerarà com a data de pagament la mateixa data de l'estat (atribut data).

## RelacioDir3
A continuació, es descriuen tots els possibles atributs d'un objecte *RelacioDir3*:
- **oficinaComptable:** objecte *[CentreAdministratiu](#centreadministratiu)* amb les dades de l'oficina comptable.
- **organGestor:** objecte *[CentreAdministratiu](#centreadministratiu)* amb les dades de l'òrgan gestor.
- **unitatTramitadora:** objecte *[CentreAdministratiu](#centreadministratiu)* amb les dades de la unitat tramitadora.

## CentreAdministratiu
A continuació, es descriuen tots els possibles atributs d'un objecte *CentreAdministratiu*:
- **codi:** codi del centre administratiu.
- **nom:** nom o descripció del centre administratiu.

## DadesRegistre
A continuació, es descriuen tots els possibles atributs d'un objecte *DadesRegistre*:
- **numero:** número de registre.
- **data:** data de registre. Format: *YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM*.

## DadesMotiuRebuig
A continuació, es descriuen tots els possibles atributs d'un objecte *DadesMotiuRebuig*:
- **codi:** codi del motiu de rebuig.
- **descripcio:** descripció del motiu de rebuig.


# Annexos

## Codis de resposta del servei
El servei tornarà algun dels codis d'estat de resposta HTTP següents:
- **200:** petició realitzada satisfactòriament.
- **400:** petició incorrecta (per exemple, error als paràmetres d'entrada).
- **401:** petició no autoritzada.
- **404:** recurs no trobat.
- **500:** altres errors

En cas d'error, el servei tornarà un fitxer de tipus `application/json`, amb el contingut següent:

```json
{
	"codiError": 9999,
	"descripcioError": "descripció error" 
}
```

A continuació, es detallen els possibles errors que pot tornar el servei:

### Errors d'autenticació (HttpStatus 401):
- **1001:** No s'ha especificat un token d'autenticació
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
- **1012:** La clau secreta no és vàlida

### Errors de recurs no trobat (HttpStatus 404):
- **2001:** No s'ha trobat la factura especificada
- **2002:** No s'ha trobat el document adjunt especificat
- **2003:** No s'ha trobat un històric d'estats per a la factura

### Errors de validació (HttpStatus: 400):
- **3001:** S'ha excedit el número màxim d'annexos permès
- **3002:** S'ha excedit la mida màxima permesa per a la petició
- **3003:** L'extensió del fitxer de factura a enviar no està suportada
- **3004:** El receptor de la factura no està donat d'alta a eFACT
- **3005:** La signatura de factura a enviar no és vàlida
- **3006:** No s'ha especificat el nom de fitxer d'algun adjunt
- **3007:** No s'ha especificat el tipus mime d'algun adjunt
- **3008:** No s'ha especificat el contingut d'algun adjunt
- **3009:** S'han informat adjunts amb tipus mime no suportats
- **3010:** S'han informat adjunts l'extensió dels quals no coincideix amb el tipus mime especificat
- **3011:** No ha estat possible obtenir la factura a enviar
- **3012:** La factura especificada no té número de registre
- **3013:** No s'ha especificat el nom del fitxer de factura
- **3014:** No s'ha especificat el contingut del fitxer de factura codificat en base64
- **3015:** S'han informat adjunts amb extensions no suportades
- **3016:** El format de la factura no és correcte
- **3017:** No ha estat possible determinar la versió de Facturae de la factura
- **3018:** La versió de Facturae de la factura no està suportada pel receptor
- **3019:** No es permeten lots de factures
- **3020:** El format del NIF de l'emissor no és vàlid
- **3021:** El format del NIF del receptor no és vàlid
- **3022:** El valor informat al camp correuElectronic supera els 512 caràcters permesos
- **3023:** El valor informat al camp correuElectronic no té el format esperat
- **3024:** La factura a enviar no està signada o bé el node de signatura no és correcte
- **3025:** L'emissor de la factura no té habilitat l'enviament a FACe

### Errors genèrics (HttpStatus 500):
- **9001:** S'ha produït un error intern de connexió amb la base de dades
- **9002:** S'ha produït un error en la generació del rebut electrònic de la factura
- **9999:** S'ha produït un error inesperat en l'execució de l'operació sol·licitada


## Possibles estats d'una factura
A continuació, es detallen els estats pels quals pot passar una factura al servei, en l'ordre coherent d'ocurrència:

|Codi d'estat eFACT|Codi d'estat númeric<br>(BOE A-2014-10660)|Descripció|
|------------------|------------------------------------------|----------|
|SENT|1000|Factura lliurada al servei eFACT.<br>Aquest estat el genera el servei eFACT de manera automàtica.|
|REGISTERED|1200|La factura ha estat registrada administrativament, proporcionant un número i data de registre, tant al proveïdor com a l'entitat.<br>Aquest estat el genera el servei eFACT de manera automàtica.|
|DELIVERED|1200|La factura ha estat lliurada a l'entitat destinatària.<br>Aquest estat és opcional i el genera l'entitat receptora de la factura.|
|ANNOTATED|1300|La factura ha estat verificada i registrada al registre comptable de factures (RCF), generant un número de registre comptable que cal proporcionar al proveïdor.<br>Aquest estat és obligatori i l'ha de generar l'entitat receptora de la factura.|
|RECEIVED|1300|La factura ha estat rebuda a la unitat de destinació.<br>Aquest estat és opcional i el genera l'entitat receptora de la factura.|
|ACCEPTED|1300|La factura ha estat conformada.<br>Aquest estat és opcional i el genera l'entitat receptora de la factura.|
|RECOGNISED|2400|L'obligació de pagament derivada de la factura ha estat reconeguda i comptabilitzada.<br>Aquest estat és obligatori i l'ha de generar l'entitat receptora de la factura.|
|PAID|2500|La factura ha estat pagada.<br>Aquest estat és obligatori i l'ha de generar l'entitat receptora de la factura.|
|REJECTED|2600|La factura ha estat rebutjada. S'ha d'indicar al proveïdor el motiu del rebuig.<br>En cas que calgui rebutjar una factura, aquest estat és obligatori i l'ha de generar:<br>- L'entitat receptora de la factura, sempre que la factura li hagi estat lliurada pel servei eFACT.<br>- El servei eFACT, en qualsevol altre cas (error de validació de signatura/certificat, error de registre, etc.).|

Una factura es pot rebutjar (estat REJECTED) en qualsevol moment, excepte si està en estat PAID.

Els estats PAID i REJECTED són estats finals i, un cop actualitzada a un d'aquests estats, s'entén que la gestió de la factura ja ha finalitzat.


## Tipus mime admesos
A continuació, s'indiquen els tipus mime admesos per a documents adjunts:
- **application/pdf** (pdf)
- **application/msword** (doc o docx)
- **application/vnd.ms-excel** (xls o xlsx)
- **application/vnd.oasis.opendocument.text** (odt)
- **application/vnd.oasis.opendocument.spreadsheet** (ods)
- **text/plain** (txt)
