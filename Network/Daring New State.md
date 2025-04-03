# Writeup: Daring New State

## Descrizione 📝
La challenge "Daring New State" riguarda l'analisi di un server DNS personalizzato che opera sulla porta **12008**. L'obiettivo è scoprire informazioni nascoste relative al dominio **the.flag**, utilizzando diverse query DNS per esaminare ogni tipo di record. Anche se a un primo sguardo il dominio sembra già registrato, indagando più a fondo troviamo indizi interessanti che portano alla flag.

## Sfruttamento 🎯

### 1. Query Iniziale
Utilizzando **dig** contro il server DNS su porta 12008, abbiamo eseguito:
```bash
dig @newstate.challs.olicyber.it -p 12008 the.flag any
```
L'output ha restituito:
- Un record **SOA** per `the.flag` con informazioni sulla zona.
- Un record **NS** che indica che il nameserver per il dominio è **here.is.the.flag**.

### 2. Analisi del Nameserver
Per approfondire, abbiamo eseguito:
```bash
dig @newstate.challs.olicyber.it -p 12008 here.is.the.flag any
```
Questo ha restituito:
- Un record **A** per `here.is.the.flag` con indirizzo `127.0.0.1`.
- Un record **TXT** che contiene un indizio:  
  > "flag{master_... look at the canonical name of base64decode.the.flag"

### 3. Esplorazione del Record CNAME
Successivamente, abbiamo eseguito una query sul record **CNAME**:
```bash
dig @newstate.challs.olicyber.it -p 12008 base64decode.the.flag any
```
L'output ha mostrato:
```
base64decode.the.flag.  86400   IN      CNAME   Li4ub2ZfZG5zXzw+fSByZXBsYWNlIDw+IHdpdGggaXAgb2YgZDgxODc3NmQ3.the.flag.
```
Il record CNAME contiene una stringa codificata in Base64. Decodificandola (utilizzando, ad esempio, CyberChef) si ottiene un messaggio che indica:  
> "look at the canonical name of base64decode.the.flag"  
In particolare, viene fornita una guida per sostituire la parte della flag:  
```
flag{master_of_dns_<>}
```
dove `<>` va sostituito con l'indirizzo IP ottenuto da un altro record.

### 4. Recupero dell'Indirizzo IP
Eseguiamo quindi una query per il record:
```bash
dig @newstate.challs.olicyber.it -p 12008 d818776d7.the.flag any
```
L'output è:
```
d818776d7.the.flag.  86400   IN      A       127.254.13.25
```
Questo fornisce l'indirizzo IP **127.254.13.25**.

### 5. Composizione della Flag
Sostituendo `<>` con l'indirizzo ottenuto, la flag finale diventa:
```
flag{master_of_dns_127.254.13.25}
```

## Flag 🏁
```
flag{master_of_dns_127.254.13.25}
```

## Lezioni Apprese 📚
1. **Esplorazione Completa dei Record DNS**: Utilizzare vari tipi di record (SOA, NS, TXT, CNAME, A) permette di ricostruire informazioni nascoste e ottenere indizi utili.
2. **Utilizzo di Server DNS Personalizzati**: Anche se un dominio sembra già registrato, analizzando i record si possono trovare ulteriori dettagli.
3. **Decodifica dei Messaggi**: La stringa codificata in Base64 all'interno del record CNAME ha fornito l'indicazione chiave per completare la flag.

## Tools Utilizzati 🛠️
- **dig**: Per eseguire query DNS su vari record.
- **CyberChef**: Per decodificare stringhe Base64.
- **Terminale**: Per analizzare le risposte DNS e comporre la flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
