# Writeup: NS_0.04 - Level4 - 3-way Handshake

## Overview
**Categoria:** network  
**Autore:** enriquez  
**Difficoltà:** 457  
**Stato:** Up  
**URL:** (esercizio locale tramite Docker)  

### Descrizione 📝
Questa sfida richiede di configurare correttamente le interfacce di rete tra due nodi (`node1` e `node2`) e di eseguire un handshake TCP a tre vie da `node1` verso `node2`. Una volta completato il handshake, è necessario analizzare l'output generato per ricavare la flag.

Le istruzioni principali sono:
1. Configurare le interfacce di rete sui container Docker.
2. Eseguire uno script denominato `level4` presente su `node1`.
3. Analizzare i pacchetti di rete catturati tramite `tcpdump` per estrarre le informazioni necessarie alla composizione della flag.

---

## Soluzione 🎯

### 1. Preparazione dell'Ambiente
Prima di iniziare, assicurarsi che Docker sia installato sul sistema. Scaricare e decomprimere il file `challenge_files.tar.gz` fornito per accedere ai container preconfigurati.

---

### 2. Accesso ai Nodi
Accedere ai container Docker corrispondenti ai nodi:

#### Accedere a **node1**:
```bash
docker exec -it id_nodo1 /bin/bash
```

#### Accedere a **node2**:
```bash
docker exec -it id_nodo2 /bin/bash
```

---

### 3. Configurazione delle Interfacce di Rete

Una volta dentro i container, configurare le interfacce di rete con gli indirizzi IP specificati:

#### Configurazione di **Node1**:
```bash
ip addr add 10.0.0.1/24 dev eth0  # Configura eth0 per la rete 10.0.0.0/24
ip addr add 192.168.168.123/24 dev eth1  # Configura eth1 per la rete 192.168.168.0/24
```

#### Configurazione di **Node2**:
```bash
ip addr add 10.0.0.2/24 dev eth0  # Configura eth0 per la rete 10.0.0.0/24
```

---

### 4. Esecuzione del 3-Way Handshake

#### Cercare lo script `level4` in **node1**:
```bash
find / | grep "level4"
```

#### Rendere lo script eseguibile:
```bash
chmod +x /path/to/level4
```

#### Eseguire lo script:
```bash
./level4
```

Questo comando avvierà il handshake TCP e genererà un output con i pacchetti di rete catturati tramite `tcpdump`.

Esempio di output di `tcpdump`:
```text
Begin emission:
...................................................................................................................................................................................................................................................................................................................Finished to send 1 packets.
.......................................................................................................................................................................................................................................................................................................................*
Received 619 packets, got 1 answers, remaining 0 packets
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
Begin emission:
...................................................................................................................Finished to send 1 packets.
..................................................................................................................*
Received 230 packets, got 1 answers, remaining 0 packets.
Sent 1 packets.
```

---

### 5. Analisi dei Pacchetti di Handshake

Dall'output di `tcpdump`, individuare i dati necessari per comporre la flag:

Esempio di output di `tcpdump`:
```text
17:54:55.265746 IP 10.0.0.1.2799 > 10.0.0.2.ssh: Flags [S], seq 2254, win 8192, length 0
17:54:55.265952 IP 10.0.0.2.ssh > 10.0.0.1.2799: Flags [S.], seq 517862217, ack 2255, win 64240, options [mss 1460], length 0
17:54:55.266499 IP 10.0.0.1.2799 > 10.0.0.2.ssh: Flags [R], seq 2255, win 0, length 0
```

- **ephemeralport**: Porta temporanea utilizzata da `node1` (2799).
- **destinationport**: Porta di destinazione sul `node2` (22 per SSH).
- **clientstartsequencenumber**: Numero di sequenza di partenza del client (2254).

---

### 6. Composizione della Flag

Utilizzare i dati estratti per costruire la flag seguendo il formato specificato:

```
CCIT{level4_[ephemeralport]_[destinationport]_[clientstartsequencenumber]}
```
---

## Flag 🏁
```
CCIT{level4_2799_22_2254}
```

---

## Opzionale. Pulizia dell'Ambiente
Terminare lo scenario e pulire la configurazione eseguendo:
```bash
make down
make clean
```

---

## Lezioni Apprese 📚
1. **Configurazione di Rete:** Impostare manualmente gli indirizzi IP delle interfacce di rete nei container Docker.
2. **TCP 3-Way Handshake:** Capire come funziona il processo di stabilimento di una connessione TCP.
3. **Analisi di Pacchetti:** Utilizzare strumenti come `tcpdump` per catturare e analizzare il traffico di rete.
4. **Scripting:** Eseguire script personalizzati all'interno dei container per automatizzare operazioni.

---

## Tools Utilizzati 🛠️
- **Docker & Docker Compose:** Per creare e gestire i container.
- **Terminal Linux:** Per eseguire comandi come `ip`, `tcpdump`, `find`, `chmod`, e `./level4`.
- **Editor di Testo:** Per esaminare eventuali file di configurazione o script.
  
---

Writeup by Luca Crippa - Last Updated: 2024
