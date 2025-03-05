# Writeup: NS_1.01 - Sanity Check

## Overview
- **Categoria**: Network  
- **Autore**: enriquez  
- **Difficoltà**: 471  
- **Stato**: Up  
- **Allegato**: `leopardo.pcapng` (file di cattura di rete)

## Descrizione 📝
La challenge introduce un file di cattura di rete (`.pcapng`) in cui è stato registrato del traffico sospetto. Il primo livello (`Level 1`) invita a dare una rapida occhiata al file per trovare una **flag** iniziale, suggerendo di guardare il commento associato alla cattura.

## Soluzione 🎯

### 1. Analisi del File di Cattura
Usiamo il comando `capinfos` per visualizzare informazioni generali sul file:
```bash
capinfos leopardo.pcapng
```
L’output riporta diverse informazioni, tra cui il **Capture comment** che contiene la flag:

```
File name:           leopardo.pcapng
File type:           Wireshark/... - pcapng (gzip compressed)
File encapsulation:  Ethernet
File timestamp precision:  microseconds (6)
Packet size limit:   file hdr: (not set)
Number of packets:   6,023
File size:           4,395 kB
Data size:           4,995 kB
Capture duration:    791.695151 seconds
Earliest packet time: 2020-03-23 12:42:49.214773
Latest packet time:   2020-03-23 12:56:00.909924
Data byte rate:      6,310 bytes/s
Data bit rate:       50 kbps
Average packet size: 829.46 bytes
Average packet rate: 7 packets/s
SHA256:              9b13b78d404c386c87b269e3d25b28644e4d2708ad4f03c85fc889c7f561395d
SHA1:                35c523887fd9dc0c7cd3499b8335b4cae1a9eddd
Strict time order:   True
Capture hardware:    Intel(R) Core(TM) i7-10510U CPU @ 1.80GHz (with SSE4.2)
Capture oper-sys:    Linux 5.3.0-42-generic
Capture application: Dumpcap (Wireshark) 2.6.10 (Git v2.6.10 packaged as 2.6.10-1~ubuntu18.04.0)
Capture comment:     CCIT{a8ec63de-0c9a-473e-9b34-11026a9e6b19}
Number of interfaces in file: 1
Interface #0 info:
                     Name = -
                     Encapsulation = Ethernet (1 - ether)
                     Capture length = 65535
                     Time precision = microseconds (6)
                     Time ticks per second = 1000000
                     Time resolution = 0x06
                     Operating system = Linux 5.3.0-42-generic
                     Number of stat entries = 0
                     Number of packets = 6023

```

### 2. Flag
La flag, quindi, è:

## Flag 🏁
```
CCIT{a8ec63de-0c9a-473e-9b34-11026a9e6b19}
```

## Lezioni Apprese 📚
1. **Commenti nei File PCAPNG**: Spesso il campo “Capture comment” può contenere informazioni utili (o addirittura la flag).  
2. **Strumenti di Analisi**: `capinfos` è un comando veloce per ottenere metadati su un file di cattura.  
3. **Passi Successivi**: Per livelli successivi, si potrà usare **Wireshark** o altri strumenti forensi per analizzare in dettaglio il traffico.

## Tools Utilizzati 🛠️
- **capinfos**: Per estrarre metadati e commenti dal file `.pcapng`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
