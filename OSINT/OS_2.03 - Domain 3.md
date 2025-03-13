# Writeup: OS_2.03 - Domain 3

## Overview
- **Categoria**: OSINT  
- **Autore**: osintitalia  
- **Difficoltà**: 488  
- **Stato**: Up  
- **Domanda**: *What is the name of the web server product hosting libero.it?*

## Descrizione 📝
La challenge chiede di determinare il **nome del web server** che ospita il dominio `libero.it`. È un classico esercizio OSINT che prevede l’ispezione degli **header HTTP** restituiti dal server.

## Soluzione 🎯

### 1. Analisi degli Header con `curl -I`
Utilizziamo `curl` per recuperare gli header di risposta del server:
```bash
curl -I https://www.libero.it
```
Parte dell’output:
```
HTTP/2 200
content-type: text/html; charset=UTF-8
date: Thu, 13 Mar 2025 11:09:38 GMT
server: Apache
cache-control: public, max-age=20
x-frame-options: SAMEORIGIN
vary: Accept-Encoding
x-cache: Hit from cloudfront
via: 1.1 b6fe7f5e64ff6b2767128fbe99eaf03a.cloudfront.net (CloudFront)
x-amz-cf-pop: CDG55-P1
x-amz-cf-id: bRVnRCe-fLEHM0BCeNVSBnWfpK7Qt2ezdSi1NqIDWbjEQBNucm9bSA==
age: 8
```
Tra gli header c’è **`server: Apache`**, che indica il prodotto web server utilizzato.

## Flag 🏁
```
Apache
```

## Lezioni Apprese 📚
1. **Header HTTP**: L’header `Server` spesso rivela la tecnologia usata per erogare le pagine web.  
2. **OSINT e Comandi di Base**: `curl -I` è uno strumento semplice ma potente per raccogliere informazioni sui servizi web.

## Tools Utilizzati 🛠️
- **curl**: Per visualizzare gli header di risposta HTTP (`-I` mostra solo gli header).

---

*Writeup by Luca Crippa - Last Updated: 2024*
