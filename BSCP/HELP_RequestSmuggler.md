## 1. Pulizia della richiesta di partenza

A partire dalla richiesta segnalata (esempio il tuo `POST /`):

- Rimuovo header non necessari (User-Agent fancy, Accept-Language, ecc.) per semplificare il PoC.  
- Mantengo: `Host`, `Content-Length`, eventuale `Transfer-Encoding`, `Connection`, `Content-Type`, cookie/sessione. [portswigger](https://portswigger.net/web-security/request-smuggling)
- Confermo che la richiesta “pulita” funziona ancora e riceve risposta normale (200 / 302 / ecc.). [portswigger](https://portswigger.net/web-security/request-smuggling/finding)

***

## 2. Capire che tipo di desync (CL.TE vs TE.CL)

Obiettivo: capire chi usa CL e chi usa TE (front-end vs back-end). [portswigger](https://portswigger.net/web-security/request-smuggling)

### Caso sospetto CL.TE (front-end usa Content-Length, back-end usa Transfer-Encoding)

Provo una richiesta in questo stile: [portswigger](https://portswigger.net/web-security/request-smuggling)

```http
POST / HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

- Se la risposta alla **richiesta successiva** (un normale GET /) diventa 404 invece che 200, ho confermato una CL.TE via “differential responses”. [portswigger](https://portswigger.net/web-security/request-smuggling/finding)

### Caso sospetto TE.CL (front-end usa TE, back-end usa Content-Length)

Provo una richiesta tipo: [portswigger](https://portswigger.net/web-security/request-smuggling)

```http
POST / HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0
```

- Stessa logica: se la richiesta successiva riceve 404 o comportamento anomalo, confermo TE.CL. [portswigger](https://portswigger.net/web-security/request-smuggling/finding)

***

## 3. Conferma pratica (interferenza con un’altra request)

1. Scelgo una **normal request stabile** (es: `GET /` o `POST /search?q=test`) che normalmente dà sempre 200. [portswigger](https://portswigger.net/web-security/request-smuggling/finding)
2. Creo una **attack request smuggler** (come sopra) che includa nel body una seconda richiesta `GET /404` o qualcosa che so che restituisce 404. [portswigger](https://portswigger.net/web-security/request-smuggling/finding)
3. Eseguo la sequenza:
   - invio la attack request  
   - subito dopo invio la normal request  
4. Se la normal request cambia comportamento (404, header strani, response rotta), la vulnerabilità è confermata. [portswigger](https://portswigger.net/web-security/request-smuggling/finding)

***

## 4. Dalla conferma allo sfruttamento

Una volta confermato il desync, decido cosa voglio ottenere. Qualche pattern classico: [portswigger](https://portswigger.net/web-security/request-smuggling/exploiting)

- **Bypass auth / front-end control**: smugglo un `GET /admin HTTP/1.1` o simili, che il back-end considera già filtrato. [portswigger](https://portswigger.net/web-security/request-smuggling/exploiting/lab-bypass-front-end-controls-cl-te)
- **XSS tramite smuggling**: se esiste un riflesso controllabile (User-Agent, parametro email, ecc.), smugglo una request contenente il payload XSS che verrà riflesso nel contesto sbagliato. [portswigger](https://portswigger.net/web-security/request-smuggling/exploiting)
- **Cache poisoning**: smugglo una request che forza una risposta malevola su una risorsa statica (`/static/include.js`) per avvelenare la cache dei successivi utenti. [portswigger](https://portswigger.net/web-security/request-smuggling/advanced)

Esempio (semi-generico) per bypass front-end: [portswigger](https://portswigger.net/web-security/request-smuggling/exploiting)

```http
POST / HTTP/1.1
Host: target
Content-Length: 120
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target
Foo: x
```



Vuoi che ti scriva direttamente un file `README.md` già formattato (con sezione “Usage with Burp HTTP Request Smuggler” e un PoC generico) pronto da committare nel tuo repo?  
