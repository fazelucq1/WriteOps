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

# Writeup: WS_1.07 - basic lfi

## Overview
- **Categoria:** web
- **Autore:** bonaff
- **Difficoltà:** 491
- **Stato:** Up (last checked: 6 minutes ago)
- **URL:** [http://basiclfi.challs.cyberchallenge.it/](http://basiclfi.challs.cyberchallenge.it/)

## Descrizione 📝
Questa challenge propone un semplice sito basato su un template HTML per un blog in PHP, che apparentemente non offre funzionalità particolari. Tuttavia, analizzando il sorgente della pagina si nota l'inclusione di file tramite una richiesta vulnerabile:
  
```html
href="/static.php?static_file=blog.css"
```

Il file `static.php` include i file specificati tramite il parametro `static_file` senza effettuare una sanitizzazione corretta, aprendo così la porta a un attacco di Local File Inclusion (LFI).

## Soluzione 🎯

### 1. Identificazione della Vulnerabilità
Il parametro `static_file` viene utilizzato per includere file dinamicamente, rendendo possibile un attacco LFI se non opportunamente limitato. Ciò consente di eseguire un attacco di directory traversal.

### 2. Sfruttamento della Vulnerabilità
Per leggere il contenuto del file `flag.txt` presente nella root, basta modificare il parametro `static_file` inserendo una directory traversal:
  
```
../../../../../flag.txt
```

Costruendo l'URL come segue:
  
```
http://basiclfi.challs.cyberchallenge.it/static.php?static_file=../../../../../flag.txt
```

Il server includerà il contenuto del file `flag.txt` nella risposta.

### 3. Ottenimento della Flag
Visitando l'URL modificato, si ottiene la seguente risposta che contiene la flag:

## Flag 🏁
```
CCIT{p01nt_po1nt_sl4sh_po1nt_p0int_s1ash}
```

## Lezioni Apprese 📚
- **Local File Inclusion (LFI):** L'inclusione non sanitizzata di file può esporre file sensibili presenti nel sistema.
- **Directory Traversal:** Manipolando il percorso dei file, è possibile accedere a file al di fuori della directory prevista dall'applicazione.
- **Sanitizzazione degli Input:** È fondamentale validare e limitare i parametri di input per prevenire attacchi di LFI e altre vulnerabilità simili.

## Tools Utilizzati 🛠️
- **Browser Web:** Per analizzare il sorgente e testare l'URL di inclusione.
- **Editor di Testo:** Per esaminare il codice e individuare la vulnerabilità.

---

*Writeup by Luca Crippa - Last Updated: 2024*
