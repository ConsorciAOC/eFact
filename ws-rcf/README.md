# Integració via WS per Registres Comptables de Factures (RCF)

1. [Introducció](#introducció)
2. [Mètode d'autenticació](#mètode-dautenticació)
	1. [Connectivitat](#connectivitat)
	2. [Autenticació i autorització](#autenticació-i-autorització)
3. [Operacions](#operacions)
	1. [Consulta de factures pendents](#consulta-de-factures-pendents)
	2. [Consulta de dades d'una factura](#consulta-de-dades-duna-factura)
	3. [Obtenció del fitxer d'una factura](#obtenció-del-fitxer-duna-factura)
	4. [Obtenció del fitxer d'un document adjunt](#obtenció-del-fitxer-dun-document-adjunt)
	5. [Obtenció del rebut electrònic de factura](#obtenció-del-rebut-electrònic-de-factura)
	6. [Obtenció de l'històric d'estats d'una factura](#obtenció-de-lhistòric-destats-duna-factura)
	7. [Actualització d'estats de factura](#actualització-destats-de-factura)
	8. [Consulta d'adjunts pendents](#consulta-dadjunts-pendents)
	9. [Eliminació d'adjunts pendents de descàrrega](#eliminació-dadjunts-pendents-de-descàrrega)
	10. [Consulta de les entitats adherides a una plataforma](#consulta-de-les-entitats-adherides-a-una-plataforma)
 	11. [Codis de resposta del servei](#codis-de-resposta-del-servei)
	12. [Llistat d'errors](#llistat-derrors)
4. [Estats de factura](#estats-de-factura)
5. [Com donar-se d'alta al servei](#com-donar-se-dalta-al-servei)
6. [Entorns](#entorns)
7. [Preguntes freqüents](#preguntes-freqüents)


# Introducció
Aquest document pretén descriure l'API REST per a la integració de les plataformes receptores amb el hub d'eFACT, amb l'objectiu de substituir la integració actual per FTP.

# Mètode d'autenticació
## Connectivitat
S'establirà un sistema de filtres d'IPs per origen, de manera que només es permeti l'accés al servei des d'IPs l'origen de les quals estigui dins dels admesos.

## Autenticació i autorització

L'autenticació es farà mitjançant l'ús de tokens JWT. Aquests tokens contenen tota la informació necessària per fer les tasques d'autenticació i autorització del peticionari. Els camps necessaris als tokens JWT seran els següents:

**•	iss:** aquest camp (Issuer) estableix l'emissor del token i cal informar-hi el codi o nom d'usuari que identifica l'integrador. Aquesta dada serà assignada pel servei de suport al procés d'alta o migració de la plataforma receptora.

**•	aud:** aquest camp (Audience) estableix el servei al qual va dirigit el token. D'aquesta manera s'evita l'ús indegut de tokens generats per a altres serveis. Aquest camp haurà de contenir el literal "efact".

**•	nbf**: aquest camp (Not Before) conté la data, expressada en format *epoch* amb precisió de segons, a partir de la qual el token entrarà en vigor. Aquest mecanisme es preveu per poder emetre tokens que començaran a ser vàlids en un instant futur. Normalment aquest camp contindrà la data actual.

**•	iat:** aquest camp (Issued At) conté la data d'emissió, en format *epoch* amb precisió de segons, del token.

**•	exp:** Aquest camp (Expires At) conté la data d'expiració, en format *epoch* amb precisió de segons, del token. És recomanable emetre els tokens amb una vigència d'uns pocs segons per evitar-ne un ús indegut en cas de robatori. El temps màxim d'expiració permès és de 300 segons (5 minuts).

Un cop emplenat el token, aquest s'haurà de signar amb l'algorisme HMAC-SHA256, utilitzant una clau secreta que serà assignada pel servei de suport en el procés d'alta o migració de la plataforma receptora. Aquesta clau haurà de ser custodiada de manera segura per part de l'integrador i no haurà de viatjar mai dins d'una petició o capçalera HTTP.

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


# Operacions

## Consulta de factures pendents

Aquesta operació permet obtenir les factures pendents de descarregar per a la plataforma associada a l'usuari que realitza la petició.
Retorna un màxim de 500 factures. De manera opcional, es permetrà filtrar pel NIF de l'entitat i/o pel codi d'oficina comptable.

**NOTA:** un cop processades, és necessari marcar les factures com "descarregades". Per a això, s'ha d'actualitzar l'estat de les factures a DELIVERED o ANNOTATED (veure operació [Actualització d'estats d'una factura](#actualització-destats-de-factura)). Si no es fa això, les factures sempre quedaran com a pendents de descàrrega i es tornarien obtenir com a resultat de l'execució d'aquesta operació.

**Path relatiu de l'operació:** /factures-pendents

### **Petició**

paràmetre|descripció|
---------|----------|
**nif:** | Paràmetre opcional. NIF de l'entitat per a la qual es volen obtenir les factures pendents de descàrrega.
**oficinaComptable:** | Paràmetre opcional. Codi de l'oficina comptable per a la qual es volen obtenir les factures pendents de descàrrega.

**Exemple petició:**

   GET [urlServei]/factures-pendents?nif=XXXXXXXX
   
   GET [urlServei]/factures-pendents?oficinaComptable=ZZZZZZZZZ
   
   GET [urlServei]/factures-pendents?nif=XXXXXXXX&oficinaComptable=ZZZZZZZZZ

### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un missatge de tipus `application/json` amb el contingut següent:

- **mesFactures**: Com que aquesta operació torna un màxim de 500 factures, en aquest camp s'indica si hi ha més factures, a part de les tornades, pendents de descàrrega per als paràmetres especificats. 	Possibles valors: true o false.

- **factures**: Col·lecció amb les dades de les factures pendents de descàrrega tornades. Com a màxim es tornaran 500 factures. Per cada factura s'especificaran les dades següents :
  - **id:** Identificador de la factura al hub
  - **nif:** NIF de l'entitat receptora de la factura.
  - **oficinaComptable:** Codi dir3 de l'oficina comptable a la qual va dirigida la factura. 
  - **organGestor:** Codi dir3 de l'òrgan gestor al qual va dirigida la factura.
  - **unitatTramitadora:** Codi dir3 de la unitat tramitadora comptable a la qual va dirigida la factura.

**Exemple resposta:**
   

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

## Consulta de dades d'una factura

Aquesta operació permet obtenir les dades de la factura corresponent a l'identificador de factura especificat com a paràmetre. Si la factura té documents adjunts associats, també es retornen les dades d'aquests.

**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura que es vol consultar.

**Exemple petició:**

   GET [urlServei]/factura/12345
   
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un missatge de tipus `application/json` amb el contingut següent:

- **id:** Identificador de la factura al hub
- **numeroFactura:** Número de la factura.
- **dataFactura:** Data d'expedició de la factura. Aquesta data segueix el format YYYY-MM-DD.
- **importFactura:** import total de la factura.
- **nifProveedor:** NIF del proveïdor de la factura.
- **nomProveedor:** Nom del proveïdor de la factura.
- **nif:** NIF de l'entitat receptora de la factura.
- **nom:** Nom de l'entitat receptora de la factura.
- **numeroRegistre:** Número de registre de la factura.
- **dataRegistre:** Data de registre de la factura. Format: YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM.
- **numeroRegistreRCF:** Número de registre comptable de la factura. Només en cas que es tracti d'una factura "registrada a RCF" per a la qual s'hagi informat aquesta dada a l'estat corresponent (ANNOTATED).
- **numeroRegistreFACe:** Número de registre de la factura a FACe. Només si es tracta d'una factura descarregada de FACe.
- **dataRegistreFACe:** Data de registre de la factura a FACe. Només si es tracta d'una factura descarregada de FACe. Format: YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM.
- **oficinaComptable:** Codi dir3 de l'oficina comptable a la qual va dirigida la factura.
- **organGestor:** Codi dir3 de l'òrgan gestor al qual va dirigida la factura.
- **unitatTramitadora:** Codi dir3 de la unitat tramitadora comptable a la qual va dirigida la factura.
- **estat:** Estat actual de la factura al hub.
- **dataEstat:** Data de l'estat actual de la factura. Format: YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM.
- **codiMotiuRebuig:** Codi del motiu de rebuig de la factura. Només en cas que es tracti d'una factura rebutjada (REJECTED).
- **descripcioMotiuRebuig:** Descripció del motiu de rebuig de la factura. Només en cas que es tracti d'una factura rebutjada (REJECTED).
- **dataPagament:** Data en què s'ha pagat la factura. Format: YYYY-MM-DD. Només si es tracta d'una factura "pagada" (PAID).
- **adjunts:** Col·lecció amb les dades dels documents adjunts, associats a la factura, pendents de descàrrega. Per cada document adjunt s'especificaran les dades següents:
  - **idAdjunt:** Identificador del document adjunt.
  - **nom:** Nom informat per l'emissor per al document adjunt.

**Exemple resposta:**

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
         
## Obtenció del fitxer d'una factura

Aquesta operació permet obtenir el fitxer de la factura corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/facturae

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura a descarregar.

**Exemple petició:**

   GET [urlServei]/factura/12345/facturae
   
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP "200"), es retornarà un fitxer de tipus "application/xml" corresponent a la factura sol·licitada.

## Obtenció del fitxer d'un document adjunt

Aquesta operació permet obtenir el fitxer del document adjunt corresponent a l'identificador del document adjunt i l'identificador de factura especificats com a paràmetres.

**Path relatiu de l'operació:** /factura/:id/adjunts/:idAdjunt

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura a què està associat el document adjunt.
**idAdjunt:** | Identificador del document adjunt a descarregar

**Exemple petició:**

   GET [urlServei]/factura/12345/adjunts/12348
   
   
### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà el fitxer del document adjunt sol·licitat. A la capçalera Content-type s'especificarà el tipus mime corresponent.
 

## Obtenció del rebut electrònic de factura

Aquesta operació permet obtenir el rebut electrònic corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/rebut

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura de la qual voleu obtenir el vostre rebut electrònic.


**Exemple petició:**

   GET [urlServei]/factura/12345/rebut
   
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP "200"), es retornarà un fitxer de tipus "application/pdf" corresponent al rebut electrònic de la factura especificada com a paràmetre.

## Obtenció de l'històric d'estats d'una factura

Aquesta operació permet obtenir l'històric d'estats corresponent a l'identificador de factura especificat com a paràmetre.

En la secció [Estats de factura](#estats-de-factura) es descriuen els estats pels quals pot passar una factura i com ha de ser flux d'actualització d'estats d'una factura.

**Path relatiu de l'operació:** /factura/:id/estats

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura de la qual voleu obtenir el vostre històric d'estats.


**Exemple petició:**

   GET [urlServei]/factura/12345/estats
   
   
### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un fitxer de tipus `application/json` amb el contingut següent:
 
- **estats:** Col·lecció amb les dades de cadascun dels estats pels quals ha passat la factura especificada. Per cada estat s'especificaran les dades següents:
  - **estat:** Codi de l'estat. (codi eFACT estat)
  - **codiEstat:** Codi numèric FACE corresponent a l'estat.
  - **dataEstat:** Data de l'estat. Format: YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM.
  - **numeroRegistre:** Número de registre. Només en cas que es tracti de l'estat "registrada" (REGISTERED).
  - **dataRegistre:** Data de registre. Només en cas que es tracti de l'estat "registrada" (REGISTERED).Format: YYYY-MM-DD"T"HH24:MI:SS.FF3TZH:TZM.
  - **numeroRegistreRCF:** Número de registre comptable de la factura. Només en cas que es tracti de l'estat "registrada a RCF" (ANNOTATED).
  - **codiMotiuRebuig:** Codi del motiu de rebuig. Només en cas que es tracti de l'estat rebutjada (REJECTED).
  - **descripcioMotiuRebuig:** descripció del motiu de rebuig. Només en cas que es tracti de l'estat rebutjada (REJECTED).
  - **dataPagament:** Data en què s'ha pagat la factura. Només en cas que es tracti de l'estat "pagada" (PAID). Format: YYYY-MM-DD.

**Exemple resposta:**
```json
{
	"estats": 
	[
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

Pel que fa a la data de pagament d'una factura (dataPagament), per als casos de factures anteriors a la integració per aquest API REST en què no es disposi d'una data de pagament concreta informada pel receptor, es considerarà com a data de pagament la mateixa data d'estat (dataEstat) de l'estat "pagada" (PAID) corresponent.

## Actualització d'estats de factura

Aquesta operació té dues funcions:

- Actualitzar l'estat d'una factura.

- Quan una factura s'actualitza a l'estat DELIVERED o ANNOTATED, la factura es considera descarregada i, per tant, ja no serà tinguda en compte per l'operació de consulta de factures pendents de descàrrega (*GET [urlServei]/factures-pendents*).

Els estats de les factures només poden ser els següents: DELIVERED, ANNOTATED, RECEIVED, ACCEPTED, RECOGNISED, PAID i REJECTED. Si s'intenta actualitzar una factura a un estat diferent, es tornarà un error indicant que l'actualització d'estat especificada no està permesa.

L'estat REGISTERED el genera automàticament eFACT i indica que la factura ja ha entrat al registre administratiu de l'entitat. Aquest estat implica la següent informació sobre les dades del registre d'entrada:
- Número de registre d'entrada.
- Data i hora d'entrada al registre administratiu de l'entitat.

**Obligatòriament, les entitats han d'informar els estats següents: ANNOTATED, RECOGNISED, PAID i, en cas de rebuig, REJECTED.** La resta dels estats es consideren opcionals. A més, cal tenir en compte el següent:

- En actualitzar a l'estat **ANNOTATED** serà obligatori informar el número de registre comptable (*numeroRegistreRCF*), perquè l'entitat ha d'haver registrat la factura al Registre Comptable de Factures.

- En actualitzar a l'estat **PAID** serà obligatori informar-ne la data de pagament corresponent (dataPagament). Aquesta data s'ha d'especificar seguint el patró YYYY-MM-DD (exemple: 2023-09-15). Aquest estat correspon a un estat final i després d'aquest estat s'entén que la gestió de la factura ja ha finalitzat.

- En actualitzar a l'estat **REJECTED** serà obligatori informar del motiu de rebuig: *descripcioMotiuRebuig*. La dada *codiMotiuRebuig* és opcional i cada plataforma la pot informar lliurement seguint la seva pròpia codificació. Aquest estat correspon a un estat final i després d'aquest estat s'entén que la gestió de la factura ja ha finalitzat. Una factura es pot rebutjar en qualsevol moment, excepte si es troba en estat PAID.

La seqüència d'estats coherent d'una factura hauria de ser la següent:
1. REGISTERED (eFACT)
2. ANNOTATED
3. RECOGNISED
4. PAID

En la secció [Estats de factura](#estats-de-factura) es descriuen els estats pels quals pot passar una factura i com ha de ser flux d'actualització d'estats d'una factura.


**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|----------|
**id:** | Identificador de la factura de la qual voleu obtenir el vostre històric d'estats.


**Exemple petició:**

  PATCH [urlServei]/factura/12345

Tot seguit, s'indiquen totes les possibles dades susceptibles de ser informades.

```json
{
	"estat": "",
	"codiMotiuRebuig": "",
	"descripcioMotiuRebuig": "",
	"numeroRegistreRCF": "", 
	"dataPagament": ""
}
```

Tots els camps són opcionals, excepte el camp "estat" i els especificats anteriorment com a obligatoris per a un determinat estat:

-	**descripcioMotiu:** només és obligatori per a l'estat REJECTED.
-	**dataPagament:** només és obligatori per a l'estat PAID.


Exemple de fitxer JSON per actualitzar una factura rebutjada (REJECTED):

```json
{
	"estat": "REJECTED",
	"codiMotiuRebuig": "E01",
	"descripcioMotiuRebuig": "No s'ha especificat el número d'expedient"
}
```

Exemple de fitxer JSON per actualitzar una factura a registrada a RCF(ANNOTATED):

```json
{
	"estat": "ANNOTATED",
	"numeroRegistreRCF": "RCF-2023-89584"
}
```

Exemple de fitxer JSON per actualitzar una factura a reconeguda l'obligació del pagament (RECOGNISED):

```json
{
	"estat": "RECOGNISED"
}
```

Exemple de fitxer JSON per actualitzar una factura com a pagada (PAID):

```json
{
	"estat": "PAID",
	"dataPagament": "2023-11-30"
}
```
 
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit, es torna un JSON amb les dades de la factura actualitzada. L'estructura d'aquest JSON seria exactament la mateixa que la que s'especifica al punt 2 per a l'operació *"Consulta de dades d'una factura"*.

## Consulta d'adjunts pendents

Atès que es poden rebre documents adjunts en qualsevol moment posterior a l'enviament de les factures associades, aquesta operació permet obtenir els documents adjunts pendents de descarregar per a la plataforma associada a l'usuari que fa la petició. Retorna un màxim de 500 documents adjunts. De manera opcional, es permetrà filtrar pel NIF de l'entitat i/o pel codi d'oficina comptable.

**Path relatiu de l'operació:** /adjunts-pendents


### **Petició**

paràmetre|descripció|
---------|----------|
**nif:** | Paràmetre opcional. NIF de l'entitat associada a la plataforma corresponent a l'usuari que fa la petició per al qual es volen obtenir les documents adjunts pendents de descàrrega.
**oficinaComptable:** | Paràmetre opcional. Codi d'oficina comptable associat a la plataforma corresponent a l'usuari que realitza la petició per al qual es volen obtenir les documents adjunts pendents de descàrrega.

**Exemple petició:**

   GET [urlServei]/adjunts-pendents?nif=XXXXXXXX
   
   GET [urlServei]/adjunts-pendents?oficinaComptable=ZZZZZZZZZ
   
   GET [urlServei]/adjunts-pendents?nif=XXXXXXXX&oficinaComptable=ZZZZZZZZZ

### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un fitxer de tipus `application/json` amb el contingut següent:
 
- **mesAdjunts:** Com que aquesta operació torna un màxim de 500 documents adjunts, en aquest camp s'indica si hi ha més adjunts, a part dels retornats, pendents de descàrrega per als paràmetres especificats. Possibles valors: true o false.
- **adjunts:** Col·lecció amb les dades dels documents adjunts pendents de descàrrega tornats. Per cada document adjunt s'especificaran les dades següents:
  - **idAdjunt:** Identificador del document adjunt.
  - **idFactura:** Identificador de la factura a què pertany el document adjunt.
  - **nom:** Nom informat per l'emissor del document adjunt.

**Exemple resposta:**

```json
{
	"mesAdjunts": false,  
	"adjunts": [
		{
			"idAdjunt": "159731025",
			"idFactura": "159731027",
			"nom": "2.pdf"
		},
		{
			"idAdjunt": "159731026",
			"idFactura": "159731027",
			"nom": "1.pdf"
		},
		{
			"idAdjunt": "159732161",
			"idFactura": "159732145",
			"nom": "1.xlsx"
		},
	]
}
```


## Eliminació d'adjunts pendents de descàrrega

Aquesta operació permet actualitzar com a descarregat l'adjunt especificat, de manera que ja no sigui tingut en compte per l'operació de consulta d'adjunts pendents de descàrrega (GET [urlServei]/adjunts-pendents).

**Path relatiu de l'operació:** /adjunts-pendents/:idAdjunt

### **Petició**

paràmetre|descripció|
---------|----------|
**idAdjunt:** | Identificador del document adjunt que es vol marcar com a descarregat.


**Exemple petició:**

DELETE [urlServei]/adjunts-pendents/2755

   
### **Resposta**

Si la petició s'ha dut a terme amb èxit, simplement es torna el codi HTTP "200".

## Consulta de les entitats adherides a una plataforma

Aquesta operació permet obtenir les dades principals de les entitats adherides a la plataforma associada a l'usuari que realitza la petició.

**Path relatiu de l'operació:** /ens

### **Petició**

GET [urlServei]/ens

   
### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP "200") es tornarà un fitxer de tipus `application/json` amb el contingut següent:
 
- **ens:** Llistat d'entitats adherides a la plataforma.
  - **nif:** NIF de l'entitat
  - **nom:** Nom de l'entitat.
  - **ine10:** Codi INE10 de l'entitat.

**Exemple resposta:**

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

## Codis de resposta del servei

El servei tornarà algun dels codis d'estat de resposta HTTP següents:

- **200:** petició realitzada satisfactòriament.
- **400:** petició incorrecta (per exemple, error als paràmetres d'entrada).
- **401:** petició no autoritzada.
- **404:** recurs no trobat.
- **500:** altres errors

## Llistat d'errors

En cas d'error, el servei tornarà un fitxer de tipus `application/json`, amb el contingut següent:

```json
{
	"codiError": 9999,
	"descripcioError": "descripcio error" 
}
```

A continuació, es detallen els possibles errors que pot tornar el servei:

### Errors d'autenticació (HttpStatus 401):

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

### Errors de recurs no trobat (HttpStatus 404):

- **2001:** No s'ha trobat la factura especificada
- **2002:** No s'ha trobat el document adjunt especificat
- **2003:** No s'ha trobat un històric d'estats per a la factura

### Errors de validació (HttpStatus: 400):

- **3001:** L'estat proporcionat no és vàlid
- **3002:** Per a actualitzar l'estat de la factura a ANNOTATED, és necessari especificar un número de registre
- **3003:** Per a actualitzar l'estat de la factura a REJECTED, és necessari especificar un motiu de rebuig
- **3004:** Per a actualitzar l'estat de la factura a PAID, és necessari especificar una data de pagament
- **3005:** La factura especificada no té número de registre
- **3006:** La data de pagament especificada no compleix el format esperat
- **3007:** No és possible actualitzar la factura especificada

### Errors genèrics (HttpStatus 500):

- **9001:** S'ha produït un error intern de connexió amb la base de dades
- **9002:** S'ha produït un error en la generació del rebut electrònic de la factura
- **9999:** S'ha produït un error inesperat en l'execució de l'operació sol·licitada


# Estats de factura
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

# Com donar-se d'alta al servei


# Entorns

A continuació, s'indica l'URL base del servei segons l'entorn:
-	**TEST**: https://efact-pre.aoc.cat/rcf 
-	**PRO**:  https://efact.aoc.cat/rcf


# Preguntes freqüents

- **Un cop realitzada la migració al API REST, es continuarà tenint accés a la bústia FTP?**
No, una vegada realitza la migració, es deshabilitarà immediatament l'accés a la bústia FTP i la platafroma corresponent només podrà integrar amb el hub a través d'aquest API REST. Les factures que haguessin quedat sense descarregar a la bústia FTP, es traspassaran de forma automàtica a la interfície API REST perquè la platafroma corresponent se les pugui descarregar. Si aquestes factures tinguessin documents adjunts associats, la plataforma receptora podrà obtenir els identificadors dels documents adjunts executant l'operació _[Consulta de dades d'una factura](#consulta-de-dades-duna-factura)_, i podrà descarregar-los executant l'operació _[Obtenció del fitxer d'un document adjunt](#obtenció-del-fitxer-dun-document-adjunt)_, especificant l'identificador de document adjunt corresponent.


- **Com es podrà continuar la gestió del cicle de vida de les factures descarregades per FTP?**
Es continuarà podent consultar les dades de qualsevol factura passada i continuar amb la gestió del cicle de vida de les factures "pendents de finalitzar" a través de les operacions corresponents d'aquest API REST. Per a això, només cal saber l'identificador de la factura al hub. Actualment, a la integració per FTP, aquesta dada la proporciona el hub a la plataforma receptora de dues formes:
   - A la pròpia nomenclatura del fitxer de factura que es diposita a la carpeta _out_ de la bústia FTP. Aquesta dada va informada a la quarta posició: _[id_emissor]@[id_receptor]@[referència_emissor]@**[hubid]**_

   - Al node _DeliveryFeedback/StatusFeedback/HubFeedback/HubId_ del fitxer XML d'estat que es lliura a la plataforma receptora al costat de la factura a la carpeta _statout_.
