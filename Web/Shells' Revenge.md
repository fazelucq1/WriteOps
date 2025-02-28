# Writeup: Shells' Revenge

## Overview
- **Categoria:** Web
- **Autore:** davidedc97
- **Difficoltà:** 513
- **Stato:** Up (last checked: 3 hours ago)
- **URL:** [http://shellrevenge.challs.olicyber.it](http://shellrevenge.challs.olicyber.it)

## Descrizione 📝
La challenge ci presenta un sito con la possibilità di caricare file e poi visualizzarli. L'obiettivo è sfruttare una vulnerabilità di file upload per eseguire codice PHP arbitrario e leggere il contenuto della flag, che si trova in `/flag.txt`.

## Soluzione 🎯

### 1. Identificazione della Vulnerabilità
L'upload dei file permette di caricare qualsiasi tipo di file, compresi i `.php`. Se il server non filtra correttamente le estensioni o non disabilita l'esecuzione di codice PHP nella cartella di upload, possiamo eseguire comandi arbitrari.

### 2. Creazione del File Maligno
Creiamo un file PHP (`shell.php`) con il seguente contenuto:
```php
<?php system('cat /flag.txt'); ?>
```
Questo comando eseguirà `cat /flag.txt`, mostrando il contenuto della flag.

### 3. Upload ed Esecuzione del File
- Carichiamo `shell.php` tramite la funzione di upload del sito.
- Otteniamo il link per visualizzare il file.
- Visitiamo il link e il codice PHP verrà eseguito, restituendo il contenuto della flag.

## Flag 🏁
```
flag{sh3l1_p0w3r_1s_k00p4_p0w3r}
```

## Lezioni Apprese 📚
- **File Upload Vulnerability:** Permettere l'upload di file PHP senza filtraggio consente l'esecuzione di codice arbitrario.
- **Esecuzione di Comandi:** Funzioni come `system()`, `exec()`, e `shell_exec()` possono essere pericolose se non vengono disabilitate o controllate.
- **Miglioramenti di Sicurezza:** Per mitigare questo tipo di attacco, il server dovrebbe:
  - Filtrare le estensioni dei file consentiti.
  - Memorizzare i file in una directory in cui PHP non può essere eseguito.
  - Utilizzare un meccanismo di whitelist per il tipo di file accettati.

## Tools Utilizzati 🛠️
- **Editor di Testo:** Per creare il file PHP.
- **Browser Web:** Per interagire con il sito e testare l'exploit.

---

*Writeup by Luca Crippa - Last Updated: 2024*
