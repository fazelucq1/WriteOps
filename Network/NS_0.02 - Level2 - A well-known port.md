# CTF Writeups & HTB Machines 💀

In questo repository archiviamo tutte le writeups di CTF (Capture The Flag) e macchine HTB che ho completato.

## Contenuti 📚
- **CTF Writeups**: Soluzioni dettagliate delle sfide CTF  
- **HTB Machines**: Guide complete per le macchine Hack The Box

## Contributi 🤝
Per contribuire con un writeup, contattami via email: `luca.crippa05@gmail.com`

## Licenza ⚖️
Repository condivisa per scopi educativi. Rispetta sempre le policy delle piattaforme quando condividi materiale.

---

# Writeup: NS_0.02 - Level2 - A well-known port

## Overview
- **Categoria:** network
- **Autore:** enriquez
- **Difficoltà:** 472
- **Stato:** Up
- **URL:** (esercizio locale tramite Docker)

## Descrizione 📝
Questa challenge fa parte della serie di esercizi per configurare e interagire con i nodi Docker. In questo scenario, il nodo1 è collegato alla macchina HOST tramite una rete privata (192.168.168.0/24).  
Su ogni nodo è in esecuzione un servizio di rete su una porta ben nota. L'obiettivo è identificare il servizio attivo su node1 ed utilizzare il client corrispondente, a partire dalla macchina HOST, per connettersi a quel servizio.  
Dal momento che è richiesto di usare il client SSH, notiamo che il nodo1 ha il servizio SSH in ascolto sulla porta 22. La password per l'utente root è fornita (ccit).

## Soluzione 🎯

### 1. Verifica dei Servizi in Ascolto
- Accediamo al container di node1 e listiamo i servizi in ascolto con:
  ```bash
  netstat -tuln
  ```
  L'output mostra, tra gli altri, che il servizio SSH è in ascolto sulla porta 22:
  ```
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
  tcp        0      0 127.0.0.11:36997        0.0.0.0:*               LISTEN
  tcp6       0      0 :::22                   :::*                    LISTEN
  udp        0      0 127.0.0.11:55132        0.0.0.0:*
  ```

### 2. Connessione al Servizio SSH
- Dal HOST, utilizziamo il client SSH per connetterci al nodo1. Supponendo che l'indirizzo assegnato a node1 nella rete privata sia `192.168.123.123`, il comando è:
  ```bash
  ssh root@192.168.123.123
  ```
- Inseriamo la password `ccit` quando richiesto.

### 3. Ottenimento della Flag
- Una volta effettuata la connessione SSH, il sistema restituisce un messaggio con la flag:
  ```
  This is the f1ag: CCIT{level2_2d01d80465de952d0788}
  ```

## Flag 🏁
```
CCIT{level2_2d01d80465de952d0788}
```

## Lezioni Apprese 📚
- **Scansione dei Servizi:** Utilizzare comandi come `netstat` permette di identificare rapidamente i servizi in ascolto sui container.
- **Utilizzo di SSH in Ambienti Containerizzati:** L'accesso interattivo tramite SSH è fondamentale per interagire con nodi Docker e raccogliere informazioni critiche.
- **Importanza delle Credenziali:** In ambienti di test, le credenziali (qui: `root` / `ccit`) sono spesso semplificate per facilitare l'accesso e la risoluzione della challenge.

## Tools Utilizzati 🛠️
- **Docker & Docker Compose:** Per la gestione dell'ambiente della challenge.
- **SSH Client:** Per accedere interattivamente al container.
- **Terminale Linux:** Per eseguire i comandi di rete e di connessione.

---

*Writeup by Luca Crippa - Last Updated: 2024*
