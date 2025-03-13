# Writeup: OS_2.02 - Domain 2

## Overview
- **Categoria**: OSINT  
- **Autore**: osintitalia  
- **Difficoltà**: 494  
- **Stato**: Up  
- **Domanda**: *What online ticket system / helpdesk software is `libero.it` using for internal assistance? (only name of company/product without subdomain and tld)*

## Descrizione 📝
La challenge chiede di identificare il software di helpdesk / ticket system utilizzato da `libero.it` per la gestione dell’assistenza interna. L’indizio più probabile si trova nei **record DNS** (es. SPF) o altre fonti OSINT.

## Soluzione 🎯

### 1. Analisi dei Record TXT
Eseguendo una query DNS su `libero.it` per i record TXT:
```bash
dig libero.it TXT +short
```
Uno dei record mostra:
```
include:mail.zendesk.com
```
Il riferimento a **Zendesk** indica che `libero.it` si affida a questo servizio di helpdesk / ticket system.

### 2. Risposta alla Domanda
Il nome del prodotto/servizio di helpdesk è **zendesk**.

## Flag 🏁
```
zendesk
```

## Lezioni Apprese 📚
1. **SPF e TXT Record**: I record DNS, specialmente TXT (es. SPF), spesso includono domini di terze parti usati per la gestione della posta e dell’assistenza.  
2. **OSINT**: Una semplice query DNS può rivelare informazioni sui fornitori di servizi interni di un’azienda.

## Tools Utilizzati 🛠️
- **dig**: Per visualizzare i record TXT di `libero.it`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
