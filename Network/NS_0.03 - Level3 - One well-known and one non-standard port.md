# Writeup: NS_0.03 - Level3 - One well-known and one non-standard port

## Overview
- **Categoria:** Network  
- **Autore:** enriquez  
- **Difficoltà:** 480  
- **Stato:** Up  
- **URL:** (Esercizio locale tramite Docker)

## Descrizione 📝
Dopo aver risolto il **Level2**, compaiono due nuovi servizi in ascolto su **node1**. Il testo della challenge ci indica che entrambi utilizzano lo stesso **application protocol** e che dovremo identificarli in base alla porta “well-known” per poi connetterci a questi servizi dal **HOST**. L’obiettivo finale è estrarre la flag.

## Soluzione 🎯

### 1. Identificare i Servizi in Ascolto
Accediamo a **node1** e utilizziamo `netstat -tulnp` per elencare i servizi in ascolto:
```bash
root@node1:/# netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address   State   PID/Program name
tcp        0      0 0.0.0.0:80      0.0.0.0:*         LISTEN  70/level3_1
tcp        0      0 0.0.0.0:22      0.0.0.0:*         LISTEN  7/sshd
tcp        0      0 0.0.0.0:8000    0.0.0.0:*         LISTEN  73/level3_2
...
```
Notiamo:
- **Porta 80** (well-known port per HTTP)
- **Porta 8000** (non-standard port, ma presumibilmente sempre HTTP)

### 2. Connessione ai Servizi dal HOST
Dal momento che il nodo1 è raggiungibile sulla rete, possiamo connetterci da HOST con `wget` (o un browser). L’indirizzo IP del nodo1 è, ad esempio, `192.168.123.123`. Proviamo:

```bash
wget http://192.168.123.123:80
wget http://192.168.123.123:8000
```
Verranno scaricati due file (ad esempio `index.html` e un secondo file con un contenuto differente).

### 3. Analisi dei File Scaricati
1. **File dalla Porta 80**:  
   - Contiene una parte di testo che suggerisce un frammento della flag.

2. **File dalla Porta 8000**:  
   - All’interno troviamo un riferimento a un’immagine in Base64 oppure un contenuto codificato che, se decodificato, rivela un altro frammento di testo.

Combinando i due frammenti, si ottiene la flag completa.

### 4. Decodifica Base64 (se necessario)
Se il secondo file contiene un tag `<img src="data:image/png;base64, ...">` o una stringa Base64, possiamo decodificarla con:
```bash
echo "STRINGA_BASE64" | base64 -d > immagine.png
```
Aprendo l’immagine o ispezionandone il contenuto, troveremo l’altra parte della flag.

### 5. Flag Finale
Unendo le parti, otteniamo:

## Flag 🏁
```
CCIT{level3_4br4cafl4gg4}
```

## Lezioni Apprese 📚
- **Identificazione dei Servizi**: `netstat -tulnp` è utile per scoprire quali porte sono in ascolto e quali programmi sono associati.
- **HTTP su Porte Diverse**: Un servizio può essere eseguito su una porta non standard, pur utilizzando lo stesso protocollo HTTP.
- **Decodifica Contenuti**: La flag può essere nascosta in diversi formati (es. Base64) e richiede analisi o decodifica manuale.

## Tools Utilizzati 🛠️
- **Docker & Docker Compose**: Per la gestione dell’ambiente.
- **netstat**: Per identificare i servizi in ascolto.
- **wget**: Per recuperare i contenuti esposti sulle porte HTTP.
- **base64**: Per la decodifica di stringhe in caso di contenuti codificati.

---

*Writeup by Luca Crippa - Last Updated: 2024*
