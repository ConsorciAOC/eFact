# Integración vía WS para Proveedores

1. [Introducció](#introducció)
2. [Mètode d'autenticació](#mètode-dautenticació)
	1. [Connectivitat](#connectivitat)
	2. [Autenticació i autorització](#autenticació-i-autorització)
3. [Operacions](#operacions)


# Introducció
Aquest document pretén descriure l'API REST per a la integració de les plataformes emissores amb el hub d'eFACT, amb l'objectiu de substituir la integració actual per FTP i al web service SOAP de proveïdors.

# Mètode d'autenticació
## Connectivitat
S'establirà un sistema de filtres d'IPs per origen geogràfic, de manera que només es permeti l'accés al servei des d'IPs l'origen geogràfic de les quals estigui dins dels admesos. Per exemple, Rússia, Iran o Xina serien orígens geogràfics no permesos.

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
