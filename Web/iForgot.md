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

# Writeup: iForgot ☘️

## Overview
- **Categoria:** web
- **Autore:** bennesp
- **Difficoltà:** 539
- **Stato:** Up (last checked: 30 minutes ago)
- **URL:** [http://iforgot.challs.olicyber.it](http://iforgot.challs.olicyber.it)

## Descrizione 📝
La challenge sembra aver subito una "pulizia" accurata, mostrando una pagina con il messaggio "Nothing here UwU". Tuttavia, una rapida occhiata al file [robots.txt](http://iforgot.challs.olicyber.it/robots.txt) rivela delle directory e file sensibili, tra cui `index.js`, `package.json` e la cartella `/.git`. 

Analizzando il repository Git, è possibile notare una storia interessante:  
- Un commit con il messaggio **"add challenge"**  
- Un successivo commit in cui viene **"removed flag"**

Questa operazione ha rimosso il file `flag.txt` che, tramite un semplice `git diff`, ci permette di recuperare il contenuto della flag.

## Soluzione 🎯

### 1. Analisi Iniziale
- **Navigazione:** Accedendo a [http://iforgot.challs.olicyber.it](http://iforgot.challs.olicyber.it) viene mostrato il messaggio "Nothing here UwU".
- **Robots.txt:** Visitando [robots.txt](http://iforgot.challs.olicyber.it/robots.txt) vengono svelate le directory sensibili:
  ```
  User-agent: *
  Disallow: /index.js
  Disallow: /package.json
  Disallow: /.git
  ```

### 2. Esplorazione del Repository Git
- **Accesso alla cartella `.git`:** Utilizzando strumenti come [GitTools](https://github.com/internetwache/GitTools) si scarica ed esplora il repository.
- **Verifica della cronologia:** Eseguendo il comando:
  ```bash
  git log --oneline --all
  ```
  si evidenziano i seguenti commit:
  - `e09c49e (HEAD -> master) removed flag`
  - `bf0699b add challenge`

### 3. Recupero della Flag
- **Confronto dei commit:** Utilizzando `git diff` tra i due commit:
  ```bash
  git diff bf0699b e09c49e
  ```
  viene mostrato il contenuto rimosso dal file `flag.txt`, rivelando la flag.

## Flag 🏁
```
flag{0h_r34lly_d1d_u_f0rg0t_t0_r3m0v3_th3_d0t_g1t_r3p0?!}
```

## Lezioni Apprese 📚
- **Esposizione accidentale:** Anche dopo una "pulizia", file e directory sensibili possono rimanere esposti (es. `.git`).
- **Importanza dei file di configurazione:** Il file `robots.txt` può fornire indizi utili per l'enumerazione di risorse non destinate al pubblico.
- **Utilizzo degli strumenti Git:** La conoscenza di Git e dei suoi comandi (come `git log` e `git diff`) è fondamentale per recuperare informazioni critiche.

## Tools Utilizzati 🛠️
- **Browser Web:** Per l'analisi iniziale e la navigazione
- **GitTools:** Per scaricare ed esaminare il repository Git
- **Terminale Linux:** Per eseguire comandi Git e analizzare il repository

---

*Writeup by Luca Crippa - Last Updated: 2024*
