# GraphQL API Vulnerabilities

> **Obiettivo**: riconoscere rapidamente dove e come falliscono le API GraphQL (design/implementazione), come **pensarle** e **testarle** in modo etico, e come **prevenirle**.  
> **Per te che fai pentest/report**: c’è anche una mini–checklist e un template di segnalazione.

***

## TL;DR

*   Le vulnerabilità GraphQL nascono spesso da **introspection attiva**, **autorizzazioni mancanti per campo/resolver**, **endpoint/methods permissivi** (GET, form-encoded), **suggestions** (schema leak), **rate‑limiting aggirabile con aliases**, **validazioni contenuto/mime non enforce** → **CSRF**.
*   **Pensiero chiave**: ogni resolver è un potenziale **punto di autorizzazione**; le **query possono essere profonde/aliasate**; **tutto passa dallo stesso endpoint** → controlli **coerenti** e **server‑side** obbligatori.

***

## Cos’è (in due righe)

*   **GraphQL** è un **linguaggio di query** per API che consente al client di chiedere **esattamente** i campi necessari (Queries), **modificare** dati (Mutations) e **ricevere eventi** (Subscriptions).
*   L’API espone **uno (o pochi) endpoint** e una **schema** (tipi/campi/argomenti) che funge da **contratto** frontend↔backend.

***

## Componenti chiave (rapido)

*   **Schema**: tipi (object/scalar/enum/union/interface), campi, argomenti, relazioni.
*   **Queries** (read), **Mutations** (write/modify), **Subscriptions** (push/WebSocket).
*   **Fields** & **Arguments**: cosa chiedi e con quali parametri.
*   **Variables**: separi struttura e valori dinamici.
*   **Aliases**: più istanze dello stesso field/type in una sola richiesta.
*   **Fragments**: riuso di subset di campi.
*   **Introspection**: query built‑in che rivelano lo **schema** → potentissima per attacker.

***

## Trovare l’endpoint GraphQL

*   **Universal query**: se invii `query{__typename}` a un endpoint GraphQL valido, ottieni una risposta con `{"data":{"__typename":"Query"}}` (o simile).
*   **Path comuni**:
    *   `/graphql`, `/api`, `/api/graphql`, `/graphql/api`, `/graphql/graphql`, (talvolta) `/v1/graphql`
*   **HTTP methods**: in prod **dovrebbe** accettare **solo** `POST application/json`, ma spesso rispondono anche a **GET** o `POST x-www-form-urlencoded` → indizio di **CSRF** potenziale.
*   **Messaggi d’errore**: “query not present” su richieste non-GraphQL è un buon segnale.

> **LAB**: *Finding a hidden GraphQL endpoint (PRACTITIONER)*

**Esempio universale (richiesta POST JSON)**:

```http
POST /graphql
Content-Type: application/json

{"query":"query{__typename}"}
```

***

## Test iniziale (osservazione passiva)

*   Naviga la webapp con il **browser di Burp** e guarda l’**HTTP history**: spesso vedrai richieste GraphQL della UI (GraphiQL/Playground inclusi, se esposti).
*   Prendi una query reale come base e **sperimenta** in Repeater variando fields/args.

***

## Vulnerabilità tipiche (con esempi + lab)

### 1) Argomenti non sanificati / IDOR (Object-level authorization)

**Idea**: se un resolver accede agli oggetti **direttamente per ID** (o altro argomento) senza **autorizzazione per oggetto**, è IDOR.  
**Esempio pratico**: la lista `products` mostra solo `listed=true`, ma `product(id:3)` restituisce un delistato.  
**Cosa testare**: cambiare ID/slug/UUID; provare risorse “mancanti” o “saltate”.  
**LAB**: *Accessing private GraphQL posts (APPRENTICE)*, *Accidental exposure of private GraphQL fields (PRACTITIONER)*

> **Suggerimento**: l’autorizzazione va **per campo e per oggetto** (non solo a livello di route).

***

### 2) Schema discovery (Introspection & Suggestions)

**Introspection**: se attiva in prod, può rivelare **tipi, campi, mutation, descrizioni** (info disclosure + mappa attacco).  
**Probe minima**:

```json
{"query":"{__schema{queryType{name}}}"}
```

**Full introspection** (spesso serve rimuovere `onOperation/onFragment/onField`):

```graphql
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types { ...FullType }
    directives { name description args { ...InputValue } }
  }
}
fragment FullType on __Type {
  kind name description
  fields(includeDeprecated:true){
    name description args{...InputValue} type{...TypeRef}
    isDeprecated deprecationReason
  }
  inputFields{...InputValue}
  interfaces{...TypeRef}
  enumValues(includeDeprecated:true){
    name description isDeprecated deprecationReason
  }
  possibleTypes{...TypeRef}
}
fragment InputValue on __InputValue {
  name description type{...TypeRef} defaultValue
}
fragment TypeRef on __Type {
  kind name ofType{ kind name ofType{ kind name ofType{ kind name } } }
}
```

**Suggestions (Apollo)**: errori “Did you mean …?” possono **svelare nomi di campi/operazioni** anche con introspection spenta. Strumenti come **Clairvoyance** ricostruiscono lo schema dai suggerimenti.

**LAB**: *Accidental exposure of private GraphQL fields (PRACTITIONER)*

***

### 3) Bypass difese sull’introspection

**Regex naive**: se bloccano solo `__schema{`, prova **spazi/newline/virgole** (GraphQL ignora whitespace, regex mal fatta no).

```json
{"query":"query{__schema\n{queryType{name}}}"}
```

**Metodi alternativi**: se POST-JSON è filtrato, prova **GET** con query URL‑encoded, o `POST x-www-form-urlencoded`.

> **LAB**: *Finding a hidden GraphQL endpoint (PRACTITIONER)*

***

### 4) Bypass rate‑limiting con **Aliases**

**Idea**: il limiter conta per **richiesta HTTP**, non per **operazioni**; con aliases invii **molte operazioni in una sola richiesta** (brute‑force “compresso”).  
**Esempio concettuale**:

```graphql
query bulkCheck($code:Int){
  a1:isValidDiscount(code:$code){ valid }
  a2:isValidDiscount(code:$code){ valid }
  a3:isValidDiscount(code:$code){ valid }
}
```

**LAB**: *Bypassing GraphQL brute force protections (PRACTITIONER)*

***

### 5) CSRF su GraphQL

**Condizioni**: endpoint accetta **GET** o **POST form‑encoded** e **non valida** `Content-Type` / **non usa CSRF token**.  
**Impatto**: query/mutation eseguite **con le credenziali della vittima** (browser).  
**LAB**: *Performing CSRF exploits over GraphQL (PRACTITIONER)*

> Nota: `POST application/json` **con enforcement del Content-Type** + **CSRF token** riduce drasticamente il rischio.

***

### 6) Altri pattern ricorrenti

*   **Autorizzazione mancante su resolver annidati**: controllo fatto solo al top-level, ma **fields nested** rivelano info sensibili.
*   **Errori verbosi**: stack trace e message schemano tipi/campi.
*   **DoS / cost‑based**: query **profonde** (depth), molte **alias**, cicli di **fragments**, input enormi → impatto su CPU/memoria/DB.
*   **CORS e cookies**: policy permissive + session cookies = amplifica CSRF/abusi cross‑origin.

***

## Metodologia rapida di test (step mentali)

1.  **Individua endpoint**: path comuni + universal query; prova **POST JSON**, poi **GET** e `x-www-form-urlencoded`.
2.  **Osserva traffico reale**: cattura query della UI; identifica **tipi/campi/args** usati “in produzione”.
3.  **Schema discovery**: prova **introspection**; se off, verifica **suggestions** (errori), renaming, typo, casing.
4.  **Autorizzazioni**:
    *   **Object-level**: IDOR su queries/mutations con argomenti (ID/slug/UUID).
    *   **Field-level**: campi sensibili presenti? accessibili via nesting?
5.  **Robustezza richiesta**:
    *   **Metodi**: accetta GET/form?
    *   **Content-Type**: validato?
    *   **CSRF token** richiesto/validato?
6.  **Abuso di complessità**:
    *   **Depth/aliases/fragments**; limiti presenti? errori specifici?
    *   **Max bytes** e timeouts?
7.  **Rate-limit**:
    *   Per‑request vs per‑operazione; aliases battono limiter?
8.  **Error handling**:
    *   Messaggi rivelano schema/stack? informazioni eccessive?

***

## Checklist tascabile (pentest)

*   [ ] `query{__typename}` risponde? endpoint trovato su path comuni?
*   [ ] POST `application/json` **enforced**? Endpoint rifiuta **GET** e **form-encoded**?
*   [ ] **Introspection**: attiva? probata con whitespace/encoding/metodi alternativi?
*   [ ] **Suggestions** attive? nomi di campi rivelati da “Did you mean…?”
*   [ ] **IDOR**: argomenti diretti (id/slug) proteggono object‑level access?
*   [ ] **Field-level auth**: campi sensibili accessibili via nesting o errori?
*   [ ] **Aliases/Fragments**: limiti su count/depth/unique fields/bytes?
*   [ ] **Rate‑limit**: per‑operazione o solo per‑richiesta? aliases lo aggirano?
*   [ ] **CSRF**: Content-Type validato, token richiesto, Origin/Referer verificati?
*   [ ] **Logging/Errors**: messaggi/stack troppo verbosi? rivelano schema?

***

## Prevenzione (best practice per dev/tester)

*   **Introspection**: disabilita in prod se l’API non è pubblica; se pubblica, **revisiona lo schema** (no campi privati).
*   **Suggestions**: disabilita/mitiga (server e framework); evita leak “Did you mean…”.
*   **Autorizzazioni**:
    *   **Per oggetto e per campo (resolver)**, non solo a livello di route/operazione.
    *   Regole coerenti su **tutte** le entry (query/mutation/subscription).
*   **Input hardening**:
    *   Enforce **types/range/format**, size limits; rifiuta input “giganti”.
    *   **Validazione server‑side** sempre.
*   **CSRF**:
    *   Accetta **solo** `POST application/json`; **verifica Content-Type** a runtime.
    *   **CSRF token** robusto + controllo **Origin/Referer** dove applicabile.
*   **Rate-limit & complessità**:
    *   **Depth limit**, **cost analysis**, limiti su **unique fields/aliases/root fields**, **max bytes**, **timeout**.
    *   **Caching** e **throttling** per query ripetitive.
*   **Error handling**: messaggi puliti, niente stack/schema details in prod.
*   **CORS**: restrittivo; evita combinazioni pericolose con cookie di sessione.
*   **Secure Dev Lifecycle**: audit regolari dello schema, test negativi, strumenti SCA/DAST con moduli GraphQL.

***

## Mini‑template di report (esempio)

**Titolo**: Introspection attiva + campi privati esposti  
**Severità**: Medium → High (in base ai campi)  
**Descrizione**: L’endpoint GraphQL consente richieste di introspection in prod e rivela tipi/campi inclusi dati sensibili (es. `user.email`, `userId`).  
**Impatto**: Information disclosure, facilitazione di attacchi mirati (IDOR/brute force).  
**Evidenze**: richiesta `{"query":"{__schema{types{name}}}"}` → elenco tipi; errori con suggerimenti attivi.  
**Cause radice**: configurazione prod non hardenizzata; assenza di policy su schema pubblico.  
**Rimedi**: disabilitare introspection/suggestions in prod; revisione schema; test di regressione.

***

## Esempi sintetici utili (da copiare in Repeater)

**Universal query**

```graphql
query{__typename}
```

**Probe introspection minima**

```graphql
{ __schema { queryType { name } } }
```

**Introspection con newline per bypass regex naive**

```graphql
query{__schema
{queryType{name}}}
```

**GET URL‑encoded (quando permesso)**

    GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D

**Aliases (concettuale)**

```graphql
query bulk($id1:ID!,$id2:ID!){
  u1:getUser(id:$id1){ id }
  u2:getUser(id:$id2){ id }
}
```

***

## Labs di riferimento (solo nome, niente soluzioni)

*   *Accessing private GraphQL posts (APPRENTICE)*
*   *Accidental exposure of private GraphQL fields (PRACTITIONER)*
*   *Finding a hidden GraphQL endpoint (PRACTITIONER)*
*   *Bypassing GraphQL brute force protections (PRACTITIONER)*
*   *Performing CSRF exploits over GraphQL (PRACTITIONER)*

***

## Glossario rapido

*   **Resolver**: funzione backend che risponde a un field/operazione.
*   **Field-level authorization**: controllo di permessi **per campo** oltre che per risorsa.
*   **Depth/Cost limit**: limiti strutturali sulle query per prevenire DoS/abusi.
*   **Suggestions**: messaggi server che propongono nomi corretti (schema leak).

***

## Per approfondire (facoltativo)

*   PortSwigger Web Security Academy — *GraphQL*
*   Documentazione GraphQL ufficiale (schema, queries, mutations)
*   Guide Apollo su best practice di sicurezza e limiting di complessità

