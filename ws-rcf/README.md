# Integració vía WS per Registres Comptables de Factures (RCF)

# Introducció
Aquest document pretén descriure l'API REST per a la integració de les plataformes receptores amb el hub d'eFACT, amb l'objectiu de substituir la integració actual per FTP.

# Mètode d'autenticació
## Connectivitat
S'establirà un sistema de filtres d'IPs per origen, de manera que només es permeti l'accés al servei des d'IPs l'origen dels quals estigui dins dels admesos.

## Autenticació i autorització

L'autenticació es farà mitjançant l'ús de tokens JWT. Aquests tokens contenen tota la informació necessària per fer les tasques d'autenticació i autorització del peticionari.Los campos requeridos en los tokens JWT serían los siguientes:

**•	iss:** aquest camp (Issuer) estableix l'emissor del token i cal informar-hi el codi o nom d'usuari que identifica l'integrador. Aquesta dada serà assignada pel servei de suport al procés d'alta o migració de la plataforma receptora.

**•	aud:** aquest camp (Audience) estableix el servei al qual va dirigit el token. D'aquesta manera s'evita l'ús indegut de tokens generats per a altres serveis. Aquest camp haurà de contenir el literal “efact”.

**•	nbf**: aquest camp (Not Before) conté la data, expressada en format epoch amb precisió de segons, a partir de la qual el token entrarà en vigor. Aquest mecanisme es preveu per poder emetre tokens que començaran a ser vàlids en un instant futur. Normalment aquest camp contindrà la data actual.

**•	iat:** aquest camp (Issued At) conté la data d'emissió, en format epoch amb precisió de segons, del token.

**•	exp:** Aquest camp (Expires At) conté la data d'expiració, en format epoch amb precisió de segons, del token. És recomanable emetre els tokens amb una vigència d'uns pocs segons per evitar-ne un ús indegut en cas de robatori. El temps màxim d'expiració permès és de 300 segons (5 minuts).

Un cop emplenat el token, aquest s'haurà de signar amb l'algorisme HMAC-SHA256, usant una clau secreta que serà assignada pel servei de suport en el procés d'alta o migració de la plataforma receptora. Aquesta clau haurà de ser custodiada de manera segura per part de l'integrador i no haurà de viatjar mai dins d'una petició o capçalera HTTP.

A continuació, es mostra un exemple de token JWT en clar, abans de codificar a Base64:

![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/c5f67b58-363b-43e1-a1fe-6d15d778f1fa)


Un cop generat el token, aquest s'inclourà a la capçalera HTTP 'Authorization' de la següent manera:

Authorization: Bearer 
![imagen](https://github.com/ConsorciAOC/eFact/assets/92558339/5faf7b0f-22ac-4f6f-96ba-1cbc0a9c86ef)


A l'exemple de capçalera HTTP amb el token JWT es mostren els diferents segments del token pintats de diferent color. Com es pot observar, els segments estan separats per un punt.


# Operacions

El servei tornarà algun dels codis d'estat de resposta HTTP següents:

**• 200:** petició realitzada satisfactòriament.

**• 400:** petició incorrecta (per exemple, error als paràmetres d'entrada).

**• 401:** petició no autoritzada.

**• 404:** recurs no trobat.

**• 500:** altres errors

## Llistat Errors

En cas d'error, el servei tornarà un fitxer de tipus “application/json” amb el contingut següent:


{

   "codiError": "9999",
   
   " descripcioError": "descripcio error"
    
}

A continuació, es detallen els possibles errors que pot tornar el servei:

### Errors d'autenticació (HttpStatus 401):

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


### Errors de recurs no trobat (HttpStatus 404):

**o	2001:** No s'ha trobat la factura especificada

**o	2002:** No s'ha trobat el document adjunt especificat

**o	2003:** No s'ha trobat un historico d'estats per a la factura


### Errors de validació (HttpStatus: 400):

**o	3001:** L'estat proporcionat no és vàlid

**o	3002:** Per a actualitzar l'estat de la factura a ANNOTATED, és necessari especificar un número de registre

**o	3003:** Per a actualitzar l'estat de la factura a REJECTED, és necessari especificar un motiu de rebuig

**o	3004:** Per a actualitzar l'estat de la factura a PAID, és necessari especificar una data de pagament

**o	3005:** La factura especificada no té número de registre

**o	3006:** La data de pagament especificada no compleix el format esperat

**o	3007:** No és possible actualitzar la factura especificada


### Errors genèrics (HttpStatus 500):

**o	9001:** S'ha produït un error intern de connexió amb la base de dades

**o	9002:** S'ha produït un error en la generació del rebut electrònic de la factura

**o	9999:** S'ha produït un error inesperat en l'execució de l'operació sol·licitada



## 1. Consulta de factures pendents

Aquesta operació permet obtenir les factures pendents de descarregar per a la plataforma associada a lusuari que realitza la petició.
Retorna un màxim de 500 factures. De manera opcional, es permetrà filtrar pel NIF de l'entitat i/o pel codi d'oficina comptable.

**Path relatiu de l'operació:** /factures-pendents

### **Petició**

paràmetre|descripció|
---------|-----------|
**nif:**| Paràmetre opcional. NIF de l'entitat associada a la plataforma corresponent a l'usuari que fa la petició per al qual es volen obtenir les factures pendents de descàrrega.
**oficinaComptable:** |Paràmetre opcional. Codi doficina comptable associat a la plataforma corresponent a lusuari que realitza la petició per al qual es volen obtenir les factures pendents de descàrrega.

Exemple petició:

   GET [urlServicio]/factures-pendents?nif=XXXXXXXX
   
   GET [urlServicio]/factures-pendents?oficinaComptable=ZZZZZZZZZ
   
   GET [urlServicio]/factures-pendents?nif=XXXXXXXX&oficinaComptable=ZZZZZZZZZ

### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP “200”) es tornarà un fitxer de tipus “application/json” amb el contingut següent (Exemple resposta):
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
       

## 2. Consulta de dades d'una factura

Aquesta operació permet obtenir les dades de la factura corresponent a l'identificador de factura especificat com a paràmetre. Si la factura té documents adjunts associats, també es retornen les vostres dades.

**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura que es vol consultar.

Exemple petició:

   GET [urlServicio]/factura/12345
   
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP “200”) es tornarà un fitxer de tipus “application/json” amb el contingut següent:

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
         
## 3. Obtenció del fitxer d'una factura

Aquesta operació permet obtenir el fitxer de la factura corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/facturae

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura a descarregar.

Exemple petició:

   GET [urlServicio]/factura/12345/facturae
   
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP “200”), es retornarà un fitxer de tipus “application/xml” corresponent a la factura sol·licitada.

## 4. Obtenció del fitxer d'un document adjunt

Aquesta operació permet obtenir el fitxer del document adjunt corresponent a l'identificador d'adjunt i l'identificador de factura especificats com a paràmetres.

**Path relatiu de l'operació:** /factura/:id/adjunts/:idAdjunt

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura a què està associat el document adjunt.
**idAdjunt:**| identificador del document adjunt a descarregar

Exemple petició:

   GET [urlServicio]/factura/12345/adjunts/12348
   
   
### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP “200”) es tornarà el fitxer del document adjunt sol·licitat. A la capçalera Content-type s'especificarà el tipus mime corresponent.
 

## 5. Obtenció del rebut electrònic de factura

Aquesta operació permet obtenir el rebut electrònic corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/rebut

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura de la qual voleu obtenir el vostre rebut electrònic.


Exemple petició:

   GET [urlServicio]/factura/12345/rebut
   
   
### **Resposta**

	Si la petició s'ha dut a terme amb èxit (codi HTTP “200”), es retornarà un fitxer de tipus “application/pdf” corresponent al rebut electrònic de la factura especificada com a paràmetre.

## 6. Obtenció de l'històric d'estats d'una factura

Aquesta operació permet obtenir l'històric d'estat corresponent a l'identificador de factura especificat com a paràmetre.

**Path relatiu de l'operació:** /factura/:id/estats

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura de la qual voleu obtenir el vostre històric d'estats.


Exemple petició:

   GET [urlServicio]/factura/12345/estats
   
   
### **Resposta**

 Si la petició s'ha dut a terme amb èxit (codi HTTP “200”) es tornarà un fitxer de tipus “application/json” amb el contingut següent:

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

Pel que fa a la data de pagament d'una factura (dataPagament), per als casos de factures anteriors a la integració per aquest API REST en què no es disposi d'una data de pagament concreta informada pel receptor, es considerarà com a data de pagament la mateixa data d'estat (dataEstat) de l'estat “pagada” (PAID) corresponent.

## 7. Actualització d'estats de factura

Aquesta operació té dues funcions:

• Actualitzar l'estat de la factura corresponent a l'identificador especificat com a paràmetre segons les dades especificades.

• A més, en cas que l'estat a què actualitzar la factura sigui DELIVERED o ANNOTATED, la factura corresponent a l'identificador especificat com a paràmetre s'actualitzarà com a descarregada, de manera que ja no sigui tinguda en compte per l'operació de consulta de factures pendents ( GET [urlServei]/factures-pendents).


Restriccions:

• L'estat REGISTERED no s'acceptarà. Només s'acceptaran els estats següents: DELIVERED, ANNOTATED, RECEIVED, ACCEPTED, RECOGNISED, PAID i REJECTED. Si s'informa un estat diferent, es tornarà un error indicant que l'actualització d'estat especificada no és permesa.

• En actualitzar a l'estat ANNOTATED serà obligatori informar el número de registre comptable (numeroRegistreRCF).

• En actualitzar a l'estat REJECTED serà obligatori informar del motiu de rebuig: descripcioMotiuRebuig. La dada codiMotiuRebuig és opcional i cada plataforma la pot informar lliurement seguint la seva pròpia codificació.

• En actualitzar a l'estat PAID serà obligatori informar-ne la data de pagament corresponent (dataPagament). Aquesta data s'ha d'especificar seguint el patró YYYY-MM-DD (exemple: 2023-09-15).

• Els estats que han d'informar les entitats de forma obligatòria són: ANNOTATED, RECOGNISED, PAID i, en cas de rebuig, REJECTED.

**Path relatiu de l'operació:** /factura/:id

### **Petició**

paràmetre|descripció|
---------|-----------|
**id:**| identificador de la factura de la qual voleu obtenir el vostre històric d'estats.


Exemple petició:

  PATCH [urlServicio]/factura/12345

Tot seguit, s'indiquen totes les possibles dades susceptibles de ser informades.

{

	"estat": "",
 
	"codiMotiuRebuig ": "",
 
	"descripcioMotiuRebuig ": "",
 
	"numeroRegistreRCF": "", 
 
	"dataPagament": ""
 
}


Tots els camps són opcionals, excepte el camp “estat” i els especificats anteriorment com a obligatoris per a un determinat estat:

•	"numeroRegistreRCF": només és obligatori per a l'estat ANNOTATED.

•	"descripcioMotiu": només és obligatori per a l'estat REJECTED.

•	"dataPagament". només és obligatori per a l'estat PAID.


Exemple de fitxer JSON per actualitzar una factura rebutjada (REJECTED):

{

 	"estat": "REJECTED",

 	"codiMotiuRebuig ": "E01",
	
 	"descripcioMotiuRebuig ": “No s'ha especificat el número d'expedient"
  
}


Exemple de fitxer JSON per actualitzar una factura a registrada a RCF(ANNOTATED):

{

	"estat": "ANNOTATED ",
 
	"numeroRegistreRCF": "RCF-2023-89584"
 
}


Exemple de fitxer JSON per actualitzar una factura a reconeguda l'obligació del pagament (RECOGNISED):

{

	"estat": "RECOGNISED "
 
}


Exemple de fitxer JSON per actualitzar una factura com a pagada (PAID):

{

	"estat": "PAID",
 
	"dataPagament": "2023-11-30"
 
}


  
   
### **Resposta**

Si la petició s'ha dut a terme amb èxit, es torna un JSON amb les dades de la factura actualitzada. L'estructura d'aquest JSON seria exactament la mateixa que la que especifica l'apartat 2.3 per a l'operació “Consulta les dades d'una factura”.


## 8. Esborrament d'adjunts pendents de descàrrega

Aquesta operació permet actualitzar com a descarregat l'adjunt especificat, de manera que ja no sigui tingut en compte per l'operació de consulta d'adjunts pendents de descàrrega (GET [urlServicio]/adjuntspendents).

**Path relatiu de l'operació:** /adjunts-pendents/:idAdjunt

### **Petició**

paràmetre|descripció|
---------|-----------|
**idAdjunt:**| identificador del document adjunt que es vol marcar com a descarregat..


Exemple petició:

DELETE [urlServicio]/adjunts-pendents/2755


   
### **Resposta**

Si la petició s'ha dut a terme amb èxit, simplement es torna el codi HTTP “200”.

## 9. Consulta de les entitats adherides a una plataforma

Aquesta operació permet obtenir les dades principals de les entitats adherides a la plataforma associada a lusuari que realitza la petició.

**Path relatiu de l'operació:** /ens

### **Petició**

GET [urlServicio]/ens

   
### **Resposta**

Si la petició s'ha dut a terme amb èxit (codi HTTP “200”) es tornarà un fitxer de tipus “application/json” amb el contingut següent:

	{
		"ens": [
     			  {
            			"nif": "ESQ1111112G",
            			"nom": "ENS FORMACIO B",
            			"ine10": "7996100002"
        		  }
    			]
	}


# Como donar-se d'alta al servei

# Entorns

A continuació, s'indica l'URL base del servei segons l'entorn:

•	TEST: https://efact-pre.aoc.cat/rcf 

•	PRO:  https://efact.aoc.cat/rcf
