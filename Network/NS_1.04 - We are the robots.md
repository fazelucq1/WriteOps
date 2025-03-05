# Writeup: NS_1.04 - We are the robots

## Overview
- **Categoria**: Network  
- **Autore**: enriquez  
- **Difficoltà**: 492  
- **Stato**: Up  
- **File Allegato**: (pcap di rete)

## Descrizione 📝
La sfida fa parte della serie relativa all’attacco alla Leopardo Company. Il file `.pcap` mostra traffico HTTP verso diversi siti/porte. Viene menzionato il file `robots.txt`, un file standard che può svelare directory o percorsi sensibili. Il livello 4 suggerisce che l’attaccante abbia esplorato le configurazioni del server e lasciato tracce nel traffico.

## Soluzione 🎯

### 1. Analisi con Wireshark
Apriamo il file pcap con **Wireshark** e utilizziamo un filtro per cercare la stringa `CCIT`, tipica delle flag:
```
frame contains "CCIT"
```

### 2. Ispezione del Pacchetto
Tra i risultati troviamo una richiesta HTTP GET simile a:
```
GET /nmaplowercheck1584963974 HTTP/1.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0 CCIT%7B37be0fff-05c4-4842-a794-33afec45592b%7D
Connection: close
Host: www.leopardocompany.com:5432
```
La flag è inclusa all’interno dell’header **User-Agent**, codificata come `%7B` ( `{` ) e `%7D` ( `}` ).

### 3. Decodifica della Flag
Convertendo `%7B` in `{` e `%7D` in `}`, otteniamo:

## Flag 🏁
```
CCIT{37be0fff-05c4-4842-a794-33afec45592b}
```

## Lezioni Apprese 📚
1. **Ricerca Mirata**: Filtrare i pacchetti con `frame contains "CCIT"` è un metodo veloce per trovare possibili flag.  
2. **User-Agent**: Un header HTTP spesso trascurato, ma può contenere informazioni sensibili o nascoste.  
3. **Decodifica URL**: I caratteri `%7B` e `%7D` corrispondono rispettivamente a `{` e `}` in URL-encoding.

## Tools Utilizzati 🛠️
- **Wireshark**: Per analizzare il traffico e applicare il filtro.  
- **URL Decoding**: Per convertire `%7B` e `%7D` in `{` e `}`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
