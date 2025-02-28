# Writeup: NS_0.05 - Level5 - A Route to the Flag

## Overview
**Categoria:** network  
**Autore:** enriquez  
**Difficoltà:** 491  
**Stato:** Up  
**URL:** (esercizio locale tramite Docker)  

### Descrizione 📝
Questa sfida richiede di configurare una rotta statica sul nodo `node2` per consentirgli di raggiungere l'indirizzo **192.168.123.123** tramite una route statica. Una volta configurata la rotta, è necessario eseguire lo script `level5` per ottenere la flag.

---

## Soluzione 🎯

### 1. Accesso al NODO2
Accedere al container Docker di **node2**:
```bash
docker exec -it id_nodo2 /bin/bash
```

### 2. Configurazione della Rotta Statica
Aggiungere una rotta statica per raggiungere la rete **192.168.123.0/24** tramite il next-hop **10.0.0.1** (indirizzo di `node1` sulla rete 10.0.0.0/24):
```bash
ip route add 192.168.123.0/24 via 10.0.0.1 dev eth0
```

### 3. Test della Configurazione
Verificare la connettività con `ping`:
```bash
ping -c 4 192.168.123.123
```

Esempio di output riuscito:
```text
64 bytes from 192.168.123.123: icmp_seq=0 ttl=64 time=4.801 ms
64 bytes from 192.168.123.123: icmp_seq=1 ttl=64 time=0.119 ms
```

### 4. Esecuzione dello Script `level5`
Cercare, renderlo eseguibile e lanciare lo script:
```bash
find / | grep "level5"  # Trova la posizione dello script
chmod +x /path/to/level5
./level5
```

Esempio di output:
```text
Begin emission:
..Finished to send 1 packets.
*
Received 3 packets, got 1 answers, remaining 0 packets

CCIT{level5_p1ngpungp4m11111}
```

---

## Flag 🏁
```text
CCIT{level5_p1ngpungp4m11111}
```

---

## Lezioni Apprese 📚
1. **Rotte Statiche:** Utilizzare il comando `ip route add` per definire percorsi personalizzati verso reti remote.
2. **Next-Hop:** Capire come specificare il next-hop corretto per raggiungere una rete non collegata direttamente.
3. **Diagnosi di Rete:** Usare strumenti come `ping` per verificare la connettività dopo modifiche alla tabella di routing.

---

## Tools Utilizzati 🛠️
- **Docker:** Per accedere ai container dei nodi.
- **Terminal Linux:** Per eseguire comandi come `ip`, `ping`, `find`, `chmod`, e `./level5`.

---

Writeup by Luca Crippa - Last Updated: 2024
