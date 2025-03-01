# Writeup: NS_0.06 - Level6 - NAT1

## Overview
- **Categoria**: Network  
- **Autore**: enriquez  
- **Difficoltà**: 495  
- **Stato**: Up  
- **URL**: (Esercizio locale tramite Docker)

## Descrizione 📝
La sfida consiste nel configurare correttamente il **Network Address Translation (NAT)** su uno dei nodi (in genere `node1`) affinché `node2` possa raggiungere l’indirizzo `192.168.123.1`. Una volta impostato il masquerading (e la relativa route su `node2`), dobbiamo eseguire uno script chiamato **level6** su `node2` per ottenere la flag.

## Soluzione 🎯

### 1. Configurazione del Masquerade su `node1`
Accediamo a **node1** e utilizziamo il comando `iptables` per impostare il masquerade (Source NAT). Supponendo che la rete locale tra `node1` e `node2` sia `10.0.0.0/24` e l’interfaccia verso la rete `192.168.123.0/24` sia `eth0`, il comando è:

```bash
iptables -t nat -A POSTROUTING -o eth0 --source 10.0.0.0/24 -j MASQUERADE
```
In questo modo, i pacchetti provenienti dalla rete `10.0.0.0/24` usciranno da `node1` con l’indirizzo IP di `node1` (192.168.123.x).

### 2. Configurazione della Rotta su `node2`
Accediamo a **node2** e verifichiamo se esiste già una rotta per `192.168.123.0/24`:
```bash
ip route
```
Se non la troviamo, aggiungiamo la rotta:
```bash
ip route add 192.168.123.0/24 via 10.0.0.1 dev eth0
```
> **Nota**: `10.0.0.1` corrisponde all’indirizzo IP di `node1` sulla rete `10.0.0.0/24`.

### 3. Verifica della Comunicazione
Testiamo la connessione da `node2` a `node1` (ad esempio `192.168.123.123` è l’IP di `node1` sulla rete `192.168.123.0/24`):
```bash
ping 192.168.123.123
```
Se riceviamo risposte, la configurazione NAT/route è corretta.

### 4. Esecuzione di `level6` su `node2`
Infine, eseguiamo lo script **level6** sul nodo2 per verificare la corretta configurazione e ottenere la flag:
```bash
chmod +x /usr/local/bin/level6

root@node2:/usr/local/bin# ./level6
Begin emission:
..Finished to send 1 packets.
*
Received 3 packets, got 1 answers, remaining 0 packets


The flag is CCIT{level6_them4squ3radeofz0rr000}
```
L’output mostrerà un pacchetto inviato, e successivamente la flag.

## Flag 🏁
```
CCIT{level6_them4squ3radeofz0rr000}
```

## Lezioni Apprese 📚
1. **Masquerading (Source NAT)**: L’uso di `iptables -t nat -A POSTROUTING` è cruciale per modificare l’indirizzo sorgente di un pacchetto e permettere a una sottorete di comunicare con reti esterne.  
2. **Route Statiche**: Se un nodo non conosce la rete di destinazione, è necessario aggiungere manualmente una rotta statica.  
3. **Debug di Rete**: Comandi come `ip route`, `ping` e `iptables` sono fondamentali per diagnosticare la connettività.  
4. **Script di Verifica**: Lo script `level6` simula un pacchetto di test, utile per verificare se la configurazione NAT funziona correttamente.

## Tools Utilizzati 🛠️
- **iptables**: Per configurare il NAT (masquerading e forwarding).  
- **ip route**: Per gestire e visualizzare le tabelle di routing.  
- **ping**: Per testare la connettività.  
- **Shell**: Per eseguire lo script `level6`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
