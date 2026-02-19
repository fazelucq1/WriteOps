## HTTP Request Smuggling

L'HTTP Request Smuggling è una vulnerabilità che sfrutta discrepanze nel parsing di richieste HTTP tra server front-end e back-end, permettendo di "nascondere" parti di una richiesta per interferire con successive.  È critica perché bypassa controlli di sicurezza, accede a dati sensibili o compromette utenti. [portswigger](https://portswigger.net/web-security/request-smuggling)

## Come Funziona

In architetture con proxy/load balancer, i server devono concordare sui confini delle richieste multiple su una connessione TCP.  Un attaccante invia una richiesta ambigua usando sia `Content-Length` (CL) che `Transfer-Encoding` (TE, chunked), interpretati diversamente dai server.  HTTP/2 end-to-end è immune, ma downgrade a HTTP/1 crea rischi. [en.wikipedia](https://en.wikipedia.org/wiki/HTTP_request_smuggling)

## Tipi di Vulnerabilità

- **CL.TE**: Front-end usa CL, back-end TE.
- **TE.CL**: Front-end usa TE, back-end CL.
- **TE.TE**: Entrambi supportano TE, ma uno ignora header offuscato. [portswigger](https://portswigger.net/web-security/request-smuggling)

| Tipo | Front-end | Back-end | Causa |
|------|-----------|----------|-------|
| CL.TE | Content-Length | Transfer-Encoding | Back-end processa chunked |
| TE.CL | Transfer-Encoding | Content-Length | Front-end chunked, back-end fisso |
| TE.TE | TE (normale) | TE (offuscato) o viceversa | Offuscamento header  [portswigger](https://portswigger.net/web-security/request-smuggling) |

## Esempi Payload

### CL.TE
Front-end legge 13 byte fino a "SMUGGLED", back-end vede chunk `0` e prepende "SMUGGLED" alla prossima richiesta. [portswigger](https://portswigger.net/web-security/request-smuggling)

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0
SMUGGLED
```

### TE.CL
Front-end processa chunk `8` + `0`, back-end legge solo 3 byte dopo `8`, lasciando "SMUGGLED" per la prossima. Disabilita auto-Update Content-Length in Burp. [portswigger](https://portswigger.net/web-security/request-smuggling)

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0

```

### TE.TE (Offuscamento)
Esempi di header per far ignorare TE a un server:

```http
Transfer-Encoding: xchunked
Transfer-Encoding : chunked  # Spazio
Transfer-Encoding: chunked\nX: X  # Newline
```



## Prevenzione

Usa HTTP/2 end-to-end senza downgrade; normalizza richieste front-end e rigetta ambigue sul back-end; non assumere body vuoti; chiudi connessioni su errori.  Disabilita reuse back-end riduce rischi ma non elimina tunneling. [portswigger](https://portswigger.net/web-security/request-smuggling)
