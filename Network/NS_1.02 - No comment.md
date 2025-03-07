# Writeup: NS_1.02 - No comment

## Overview
- **Categoria**: Network  
- **Autore**: enriquez  
- **Difficoltà**: 478  
- **Stato**: Up  
- **File Allegato**: (pcap di rete)

## Descrizione 📝
Secondo livello della serie dedicata all’attacco alla Leopardo Company. Il file `.pcap` mostra traffico HTTP, compresi file JavaScript. Analizzando uno di questi (ad es. `api.js`), si trova un frammento di codice che rivela le credenziali di un utente amministratore, seppur offuscate.

## Soluzione 🎯

### 1. Analisi del Codice in `api.js`
All’interno del file JavaScript, troviamo:
```js
String.prototype.obf = function () {
    var bytes = [];
    for (var i = 0; i < this.length; i++) {
        bytes.push(this.charCodeAt(i).toString(16));
    }
    return bytes.join('$');
} 

String.prototype.deobf = function () {
    var arr = this.split('$');
    return arr.map(function(c) { 
        return String.fromCharCode(parseInt(c, 16))
    }).reduce(function(a, b) {return a + b})
} 

var api_user = "apiadmin";
var api_password = "43$43$49$54$7b$38$30$30$62$30$63$32$31$2d$33$34$37$36$2d$34$33$62$34$2d$62$65$64$31$2d$30$30$61$35$39$34$36$65$30$34$64$35$7d"
```

### 2. Decodifica della Password
- **api_password** è una stringa di codici esadecimali separati da `$`.
- Ogni codice (es. `43`) corrisponde a un carattere ASCII (es. `0x43 -> C`).

Ad esempio, usando un qualsiasi convertitore esadecimale (come [CyberChef](https://gchq.github.io/CyberChef/) o un semplice script Python), otteniamo:

```
"43" -> 0x43 -> 'C'
"43" -> 0x43 -> 'C'
"49" -> 0x49 -> 'I'
"54" -> 0x54 -> 'T'
"7b" -> 0x7b -> '{'
...
"7d" -> 0x7d -> '}'
```

Ricostruendo tutti i caratteri, otteniamo:

## Flag 🏁
```
CCIT{800b0c21-3476-43b4-bed1-00a5946e04d5}
```

## Lezioni Apprese 📚
1. **Offuscamento Semplice**: Trasformare i caratteri ASCII in esadecimale è una forma di offuscamento facilmente reversibile.  
2. **Analisi di File JavaScript**: In una cattura di rete (pcap), i file `.js` possono contenere credenziali o chiavi, se il server non li protegge adeguatamente.  
3. **Uso di Strumenti Online**: Siti come CyberChef velocizzano notevolmente la conversione tra formati (hex, base64, ecc.).

## Tools Utilizzati 🛠️
- **Wireshark**: Per individuare e seguire i file JavaScript nel pcap.  
- **CyberChef** (o script personalizzati): Per convertire la stringa esadecimale in ASCII.

---

*Writeup by Luca Crippa - Last Updated: 2024*
