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

# Writeup: WS_1.03 - ZipZap - Level 1

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 500
- **Stato:** Up (last checked: 8 minutes ago)
- **URL:** [http://zipzap.challs.cyberchallenge.it](http://zipzap.challs.cyberchallenge.it)

## Descrizione 📝
La challenge permette di caricare file ZIP, che vengono automaticamente estratti dal server. Analizzando il codice sorgente fornito (`sources.zip`), si nota che l'applicazione esegue il comando:
```python
command = 'unzip -j -o {}'.format(zip_path)
```
Questo comando è vulnerabile a un attacco tramite **symlink**.

## Soluzione 🎯

### 1. Identificazione della Vulnerabilità
- Il server estrae automaticamente i file ZIP.
- L'opzione `-j` di `unzip` ignora le directory e mantiene solo i file.
- L'estrazione non verifica la presenza di **symlink**, che possono puntare a file arbitrari.

### 2. Creazione del Symlink a `/flag.txt`
Dal nostro terminale:
```sh
ln -s /flag.txt my_symlink
```
Questo crea un collegamento simbolico a `/flag.txt`.

### 3. Creazione dell'Archivio ZIP
```sh
zip --symlinks archivio.zip my_symlink
```
L'opzione `--symlinks` preserva i collegamenti simbolici nel file ZIP.

### 4. Upload ed Esecuzione dell'Exploit
- Carichiamo `archivio.zip` sul sito.
- Il server estrae il file, seguendo il symlink.
- Visitando il file estratto, otteniamo la flag.

## Flag 🏁
```
CCIT{z1p-T0tally_is_4n_hyper_l1nk_fl4g_proce770r}
```

## Lezioni Apprese 📚
- **Zip Slip Attack:** L'estrazione di file ZIP senza controlli può portare a vulnerabilità di **directory traversal** o abuso di **symlink**.
- **Protezione contro i Symlink:** Il server dovrebbe verificare ed eliminare i **symlink** prima di estrarre file ZIP.
- **Uso sicuro di `unzip`:** Si consiglia di usare `unzip` con l'opzione `-K` per ignorare i **symlink**.

## Tools Utilizzati 🛠️
- **Terminale Linux:** Per creare il symlink e zippare il file.
- **Burp Suite:** Per testare l'upload.

---

*Writeup by Luca Crippa - Last Updated: 2024*
