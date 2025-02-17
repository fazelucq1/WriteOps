# CTF Writeups & HTB Machines 💀

In questo repository archiviamo tutte le writeups di CTF (Capture The Flag) e macchine HTB (Hack The Box) che ho completato.

## Contenuti 📚
- **CTF Writeups**: Soluzioni dettagliate delle sfide CTF
- **HTB Machines**: Guide complete per le macchine Hack The Box

## Contributi 🤝
Per contribuire con un writeup, contattami via email: `luca.crippa05@gmail.com`

## Licenza ⚖️
Repository condivisa per scopi educativi. Rispetta sempre le policy delle piattaforme quando condividi materiale.

---

# Writeup: Admin's secret 🔓

## Overview
- **Categoria:** web
- **Autore:** riccardotornesello
- **Difficoltà:** 751
- **Stato:** Up (last checked: 4 minutes ago)
- **URL:** [http://adminsecret.challs.olicyber.it](http://adminsecret.challs.olicyber.it)

## Descrizione 📝
La challenge ci offre un sito in cui è possibile registrarsi, ma solo l'amministratore ha accesso alle informazioni più interessanti. La vulnerabilità si trova nel file `register.php`, dove viene eseguita un'inserimento di dati nel database senza alcuna sanitizzazione degli input. L'obiettivo è sfruttare una SQL Injection per alterare il valore del parametro `admin` da `false` a `true`.

## Soluzione 🎯

### 1. Analisi del File `register.php`
Nel file `register.php` troviamo la seguente query SQL:
```php
$sql = "INSERT INTO users(username,password,admin) VALUES ('" . $username . "','" . $password . "',false);";
```
La vulnerabilità si trova nell'inserimento del valore `false` per il campo `admin`, che non è sanitizzato e può essere manipolato tramite SQL Injection.

### 2. Payload per SQL Injection
Per modificare il valore di `admin` a `true`, possiamo usare il seguente payload:
- **username:** `fazelucq`
- **password:** `ciao', true) --`

La query finale diventa:
```sql
INSERT INTO users(username,password,admin) VALUES ('fazelucq','ciao', true);-- ',false);
```
Il `--` commenta la parte successiva della query, lasciando solo l'inserimento del nuovo utente con il valore `admin` impostato a `true`.

### 3. Registrazione come Admin
Dopo aver registrato l'utente con il payload, possiamo accedere alla pagina di login con le credenziali:
- **username:** `fazelucq`
- **password:** `ciao`

Una volta effettuato il login, avremo accesso alla flag.

## Flag 🏁
```
flag{c0me_se1_div3ntato_admin}
```

## Lezioni Apprese 📚
- **Vulnerabilità SQL Injection:** Anche se la parte più critica della query sembra essere il parametro `admin`, è sempre importante verificare se ci sono opportunità per manipolare query SQL in modo non previsto.
- **Commenti SQL:** L'uso dei commenti (`--`) è un modo efficace per ignorare il resto di una query e manipolare il comportamento del database.

## Tools Utilizzati 🛠️
- **Browser Web:** Per testare il sito e interagire con le pagine.
- **Editor di Testo:** Per modificare i payload e eseguire i test.

---

*Writeup by Luca Crippa - Last Updated: 2024*
