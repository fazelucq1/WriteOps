# Writeup: OS_2.01 - Domain 1

## Overview
- **Categoria**: OSINT (Open Source Intelligence)  
- **Autore**: osintitalia  
- **Difficoltà**: 473  
- **Stato**: Up  
- **Domanda**: *What is the primary MX DNS record of `libero.it`?*

## Descrizione 📝
La challenge chiede di determinare il record **MX** (Mail eXchanger) principale del dominio `libero.it`. In pratica, dobbiamo eseguire una query DNS per ottenere i record MX e identificare il valore con priorità più bassa (che corrisponde al server primario).

## Soluzione 🎯

### 1. Uso di `dig` per i Record MX
Eseguiamo:
```bash
dig libero.it MX
```
L’output contiene:
```
;; ANSWER SECTION:
libero.it.   17035   IN   MX   10 smtp-in.libero.it.
```
Il numero `10` indica la priorità (più basso → maggiore priorità). Quindi il **primary MX** è:

## Flag 🏁
```
smtp-in.libero.it
```

## Lezioni Apprese 📚
1. **Record MX**: Specificano i server di posta in arrivo per un dominio.
2. **Priorità**: Un valore numerico più basso indica un server con priorità più alta.
3. **Strumenti OSINT**: Comandi DNS (`dig`, `nslookup`) sono fondamentali per raccogliere informazioni su domini e server.

## Tools Utilizzati 🛠️
- **dig**: Per eseguire la query DNS MX su `libero.it`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
