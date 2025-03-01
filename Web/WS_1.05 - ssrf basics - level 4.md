# Writeup: WS_1.05 - ssrf basics - level 4

## Overview
- **Categoria:** Web  
- **Autore:** bonaff  
- **Difficoltà:** 499  
- **Stato:** Down (last checked: 20 hours ago)  
- **URL:** [http://ssrf4.challs.cyberchallenge.it/](http://ssrf4.challs.cyberchallenge.it/)

## Descrizione 📝
Quest’ultima challenge della serie **SSRF basics** introduce ulteriori restrizioni rispetto ai livelli precedenti. Il codice PHP controlla rigorosamente gli indirizzi IP e gli schemi degli URL (ammessi solo `http` e `https`), bloccando in modo esplicito `127.0.0.1/8` e `0.0.0.0/32`. Inoltre, è disabilitato il **follow redirect** (non segue reindirizzamenti HTTP) e viene applicata una **Content-Security-Policy** che limita ulteriormente le risorse esterne.

### Codice Sorgente Principale
```php
<?php
if(isset($_GET['source'])){
    highlight_file(__FILE__);
    return;
}

header("Content-Security-Policy: default-src 'none'; style-src cdnjs.cloudflare.com");
function cidr_match($ip, $range){
    list ($subnet, $bits) = explode('/', $range);
    $ip = ip2long($ip);
    $subnet = ip2long($subnet);
    $mask = -1 << (32 - $bits);
    $subnet &= $mask; // in case the supplied subnet was not correctly aligned
    return ($ip & $mask) == $subnet;
}

if(isset($_GET['url']) && !is_array($_GET['url'])){
    $url = $_GET['url'];
    if (filter_var($url, FILTER_VALIDATE_URL) === FALSE) {
        die('Not a valid URL');
    }
    $parsed = parse_url($url);
    $host = $parsed['host'];
    if (!in_array($parsed['scheme'], ['http','https'])){
        die('Not a valid URL');
    }

    $true_ip = gethostbyname($host);
    if(cidr_match($true_ip, '127.0.0.1/8') || cidr_match($true_ip, '0.0.0.0/32')){
        die('Not a valid URL');
    }
    /*
    Orange it's my new obsession
    Yeah, Orange it's not even a question(mark)
    Orange on the host of your gopher, cause
    Orange is the bug you discovah
    Orange as the ping on your server
    Orange cause you are so very
    Orange it's the color of parsers
    A-cause curl it just goes with the setopt
    */
    $ch = curl_init($url);
    curl_setopt ($ch, CURLOPT_FOLLOWLOCATION, 0);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    echo curl_exec($ch);
    curl_close($ch);
    return;
}
?>
```

### Restrizioni e Controlli Principali
1. **Blocco di IP Locali o Riservati**: Se l’host risolve a `127.0.0.1/8` o `0.0.0.0/32`, la richiesta viene rifiutata.  
2. **Schemi Ammessi**: Solo `http` o `https`.  
3. **Nessun Redirect**: `CURLOPT_FOLLOWLOCATION` è impostato a `0`.  
4. **CSP Ristretta**: Caricamento di risorse limitato; non ha però un impatto diretto sul bypass dell’SSRF, ma impedisce di includere script o stili esterni se si volesse tentare un attacco dal lato client.  

## Soluzione 🎯

### 1. Hint dal Codice: “Orange”
Nel codice appare un commento ripetitivo con la parola **“Orange”**. Cercando “SSRF Orange” su Google si trova un talk famoso di **Orange Tsai** riguardante varie tecniche di SSRF, tra cui **Abusing URL Parsers**.

### 2. Abusing URL Parsers
Uno dei metodi noti per bypassare i controlli sugli host è la notazione:
```
http://foo@evil.com:80@google.com
```
I parser di URL in alcuni linguaggi potrebbero interpretare diversamente le parti precedute da `@`, consentendo di forzare la richiesta verso l’host `evil.com` anziché `google.com`. Nel nostro caso, vogliamo forzare la richiesta verso `127.0.0.1` (locale) nonostante la blacklist, usando la sintassi “@host:port@”.  

### 3. Costruzione dell’URL Bypass
Sappiamo che la risorsa da ottenere è `/get_flag.php`, e vogliamo puntare a `127.0.0.1`. Proviamo la notazione:
```
http://foo@127.0.0.1:80@google.com/get_flag.php
```
- **foo@**: Informazione utente fittizia.  
- **127.0.0.1:80**: IP e porta reali a cui connettersi.  
- **@google.com**: Parser interpretato in modo da ignorare effettivamente `google.com`.  
- **/get_flag.php**: Path locale che vogliamo leggere.  

### 4. Invio della Richiesta
Passiamo l’URL “trappola” come parametro `url`:
```
http://ssrf4.challs.cyberchallenge.it/?url=http://foo@127.0.0.1:80@google.com/get_flag.php
```
Il parser di PHP, in determinate condizioni, ignora la parte successiva a `@google.com` come se fosse un “fragment” o un’ulteriore parte di userinfo, e risolve la richiesta a `127.0.0.1:80`.

### 5. Risultato e Flag
Con questa tecnica di “URL parsing abuse”, la richiesta viene effettivamente inviata a `127.0.0.1/get_flag.php`, bypassando i controlli IP. La risposta del server contiene la flag:

## Flag 🏁
```
CCIT{c62e11538a15b4ab1c5fbb4e5b8fbd94}
```

## Lezioni Apprese 📚
- **Bypass SSRF Avanzati**: Anche se si blocca `127.0.0.1`, esistono modi di manipolare il parsing degli URL (user@host:port@domain) per aggirare i controlli.  
- **Importanza della Validazione**: Le semplici funzioni di PHP (es. `filter_var($url, FILTER_VALIDATE_URL)`) potrebbero non proteggere da tutti i vettori di attacco.  
- **URL Parser Pitfalls**: Le varie librerie di parsing possono avere comportamenti diversi di fronte a URL con più simboli `@`.  
- **Riferimenti**: L’approccio “Orange SSRF” è ben documentato nel PDF di Orange Tsai su SSRF.

## Tools Utilizzati 🛠️
- **Browser Web / cURL**: Per testare la vulnerabilità e provare la notazione dell’URL.  
- **Google / PDF di Orange Tsai**: Per studiare la tecnica di “Abusing URL Parsers”.

---

*Writeup by Luca Crippa - Last Updated: 2024*
