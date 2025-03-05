# Writeup: NS_1.05 - Name Swap

## Overview
- **Categoria**: Network  
- **Autore**: enriquez  
- **Difficoltà**: 490  
- **Stato**: Up  
- **File Allegato**: (pcap di rete)

## Descrizione 📝
Siamo al quinto livello della serie di challenge legate all’attacco alla Leopardo Company. Il file di cattura (pcap) contiene pacchetti che, in ordine cronologico, mostrano le varie fasi dell’attacco. L’obiettivo di questo livello è individuare una porzione di traffico che rivela la flag, cercando un pattern specifico all’interno dei pacchetti.

## Soluzione 🎯

### 1. Analisi con Wireshark
Apriamo il file pcap con **Wireshark** e utilizziamo il **filtro**:
```
frame contains "CCIT{"
```
Questo ci permette di isolare i frame che contengono la stringa `CCIT{`, indicazione tipica delle flag.

### 2. Ispezione del Flusso TCP
Individuato il pacchetto, facciamo **tasto destro → Follow → TCP Stream** (o “Segui flusso TCP”) per vedere la conversazione in testo. All’interno, troviamo una riga simile a:
```
.flag:CCIT{fc0eaa80-47e9-4c8a-87c7-de60a381b80f}.operator:$apr1$lbb2/Ao0$dLj9QIXSH4FIuFMfQQFaK..webmaster:{SHA}3848d1674b648f1d358ec7d68e1a8e764f11c027.
```

### 3. Ottenimento della Flag
La flag si trova tra `CCIT{` e `}`, quindi:

## Flag 🏁
```
CCIT{fc0eaa80-47e9-4c8a-87c7-de60a381b80f}
```

## Lezioni Apprese 📚
1. **Ricerca nei Pacchetti**: Utilizzare i filtri di Wireshark (`frame contains "CCIT{"`) è un metodo rapido per individuare stringhe note all’interno del traffico.  
2. **Follow TCP Stream**: Funzionalità essenziale per ricostruire la conversazione testuale di un flusso TCP.  
3. **Sequenza Cronologica**: Anche se la sfida menziona la cronologia, in questo caso la semplice ricerca della stringa `CCIT{` è sufficiente per trovare la flag.

## Tools Utilizzati 🛠️
- **Wireshark**: Per filtrare i pacchetti e seguire il flusso TCP.  
- **frame contains**: Filtro di Wireshark per cercare stringhe all’interno dei frame.

---

*Writeup by Luca Crippa - Last Updated: 2024*
