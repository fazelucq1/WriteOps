# Writeup: SS_0.01 - The safe

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 434  
- **Stato**: Up  
- **File Allegato**: `the_safe` (ELF)

## Descrizione 📝
La sfida consiste in un eseguibile ELF chiamato **`the_safe`**, che richiede l’inserimento di una password per sbloccare il contenuto. L’obiettivo è scoprire tale password e ottenere la flag finale.

## Soluzione 🎯

### 1. Analisi Iniziale
Rendiamo l’eseguibile avviabile e lo eseguiamo:
```bash
chmod +x the_safe
./the_safe
```
Il programma richiede una password, sostenendo che sia **“super-secure”**. Senza ulteriori indizi, cerchiamo informazioni al suo interno.

### 2. Ricerca con `strings`
Utilizziamo `strings` per estrarre le stringhe leggibili all’interno del binario:
```bash
strings the_safe
```
Tra le varie stringhe, spicca:
```
you_cAnT_gue55_th1z_s3cur3_p@ssword
```
Questa è con ogni probabilità la password “nascosta” all’interno del file.

### 3. Inserimento della Password
Rieseguiamo `./the_safe` e forniamo la password trovata:
```
you_cAnT_gue55_th1z_s3cur3_p@ssword
```
Viene rivelata la flag:

## Flag 🏁
```
CCIT{d0nT-u5e-plA1nt3xT}
```

## Lezioni Apprese 📚
1. **Analisi di Base dei Binari**: Utilizzare comandi come `strings` può rivelare informazioni sensibili in eseguibili non offuscati.  
2. **Password Hardcoded**: Inserire password in chiaro all’interno di un binario è una pratica insicura.  
3. **Protezione dei Binari**: Per evitare che un semplice `strings` riveli informazioni critiche, si dovrebbe ricorrere a tecniche di offuscamento o a un flusso di autenticazione più sicuro.

## Tools Utilizzati 🛠️
- **strings**: Per estrarre stringhe in chiaro da un binario ELF.  
- **Shell Linux**: Per eseguire e testare il binario.

---

*Writeup by Luca Crippa - Last Updated: 2024*
