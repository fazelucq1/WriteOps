# Business Logic Vulnerabilities

> **Obiettivo**: capire rapidamente cosa sono le vulnerabilità di logica applicativa, dove si nascondono, come pensarle e testarle **senza** step-by-step exploit.  
> **Per te che fai pentest e report**: trovi anche una mini–sezione “come riportarle”.

***

## TL;DR

*   Le **business logic vulns** nascono quando l’applicazione fa **assunzioni sbagliate** su **input**, **sequenze**, **ruoli** o **stati** e **non valida** sul server.
*   Sono spesso **uniche per applicazione**, difficili da scannerizzare automaticamente → **ideali per test manuali**.
*   Pensa “**come posso mettere l’app in uno stato inaspettato?**” (ordini fuori sequenza, parametri mancanti, tipi/limiti estremi, client-side controls bypassati, dipendenze tra step).

***

## Cos’è una Business Logic Vulnerability?

Una **falla nella progettazione/implementazione** delle regole di business che permette di ottenere un **comportamento non previsto** (es. saltare step, applicare sconti non dovuti, invertire flussi di denaro, bypassare 2FA), **manipolando funzionalità legittime**.

> **LAB di riferimento**: *Excessive trust in client-side controls (APPRENTICE)*, *High‑level logic vulnerability (APPRENTICE)*

***

## Perché nascono?

*   **Assunzioni sbagliate** su come gli utenti si comporteranno o su come i componenti interagiscono.
*   **Validazioni solo client-side** e/o **mancanza di controlli server-side**.
*   **Workflow complessi/non documentati**, dipendenze implicite, stati non gestiti.
*   **Team diversi** che non conoscono il tutto e non documentano le assunzioni.

> **LAB di riferimento**: *Inconsistent security controls (APPRENTICE)*, *Insufficient workflow validation (PRACTITIONER)*

***

## Impatto tipico

*   **Autenticazione/Autorizzazione**: bypass parziali o completi (↑ superficie d’attacco).
*   **Finanza/Commerce**: frodi, sconti indebiti, trasferimenti “al contrario”, “soldi infiniti”.
*   **Danno indiretto**: degrado servizio, lock di processi, abuso funzioni costose.

> **LAB di riferimento**: *Authentication bypass via flawed state machine (PRACTITIONER)*, *Infinite money logic flaw (PRACTITIONER)*

***

## Pattern comuni (con esempi pratici + lab)

### 1) Fiducia eccessiva nei controlli client-side

**Idea**: il server si fida del form/JS del browser.  
**Esempi pratici**:

*   Attributi HTML (`disabled`, `readonly`) o step UI che “proteggono” valori sensibili.
*   Prezzi/quantità/ruoli modificabili alterando la richiesta **dopo** l’invio dal browser.  
    **Cosa testare**: intercetta e modifica **tutti** i parametri critici lato server (Burp Proxy/Repeater).  
    **LAB**: *Excessive trust in client-side controls (APPRENTICE)*, *2FA broken logic (PRACTITIONER)*

***

### 2) Input “non convenzionali” non gestiti

**Idea**: il backend non gestisce limiti, segni, overflow o tipi inattesi.  
**Esempi pratici**:

*   **Quantità negative** in carrello o **importi negativi** in trasferimenti (invertono la logica dei controlli).
*   **Stringhe lunghissime** su campi nominalmente corti (stati di errore/overflow logico).
*   **Tipi anomali** (bool/stringhe su campi numerici, array su campi scalari).  
    **Cosa testare**: min/max, zero/negativi, boundary values, tipi “sbagliati”, encoding.  
    **LAB**: *High‑level logic vulnerability (APPRENTICE)*, *Low‑level logic flaw (PRACTITIONER)*, *Inconsistent handling of exceptional input (PRACTITIONER)*

> **Snippet concettuale (pseudocodice)**:

```js
// Esempio semplificato di check insufficiente
if (transferAmount <= currentBalance) {
  // Se transferAmount è -1000, la condizione passa sempre.
  completeTransfer();
}
```

***

### 3) Assunzioni sul comportamento utente

**a) “Utenti fidati restano sempre fidati”**  
Controlli forti solo all’inizio, poi **allentati** in step successivi.  
**LAB**: *Inconsistent security controls (APPRENTICE)*

**b) “I campi obbligatori ci sono sempre”**  
Rimozione di parametri/valori in transito → esecuzione di **code path alternativi**.  
**LAB**: *Weak isolation on dual-use endpoint (PRACTITIONER)*, *Password reset broken logic (APPRENTICE)*

**c) “La sequenza verrà rispettata”**  
Workflow a step: l’attaccante salta/ripete/riordina richieste (**forced browsing**).  
**LAB**: *2FA simple bypass (APPRENTICE)*, *Insufficient workflow validation (PRACTITIONER)*, *Authentication bypass via flawed state machine (PRACTITIONER)*

***

### 4) Difetti “domain-specific” (es. e‑commerce, banking, social)

**Esempi pratici**:

*   **Sconti** applicati oltre la soglia e **non ricalcolati** quando il carrello cambia.
*   **Programmi fedeltà**: punti calcolati male su annulli/resi.
*   **Inventario**: race/ri-uso step per aggirare limiti.  
    **LAB**: *Flawed enforcement of business rules (APPRENTICE)*, *Infinite money logic flaw (PRACTITIONER)*

***

### 5) **Encryption oracle** disponibile all’utente

**Idea**: l’app consente di cifrare input arbitrario e di riutilizzare quel ciphertext altrove.  
**Rischio**: creare input cifrati “validi” per endpoint sensibili che usano lo stesso algoritmo/chiave.  
**LAB**: *Authentication bypass via encryption oracle (PRACTITIONER)*

***

### 6) **Discrepanze nel parsing email**

**Idea**: diversi componenti interpretano l’email in modo diverso (quoting/encoding/CFWS).  
**Impatto**: **accesso non autorizzato** a funzioni riservate a domini specifici.  
**LAB**: *Bypassing access controls using email address parsing discrepancies (EXPERT)*

***

## Come pensarle e trovarle (metodologia rapida)

1.  **Mappa il workflow**: quali step, quali variabili di stato, quando vengono calcolati sconti/tasse/ruoli?
2.  **Forza stati anomali**: salta uno step, ripetilo due volte, torna indietro, cambia l’ordine.
3.  **Gioca con i parametri**: rimuovi **uno alla volta**; cambia tipo (stringa↔numero↔array), segno (±), dimensione (molto lungo/corto).
4.  **Controlla la coerenza**: i controlli sono uguali in ogni endpoint/stadio?
5.  **Cerca dipendenze implicite**: lo sconto applicato allo step 2 resta valido dopo modifiche allo step 4?
6.  **Osserva gli errori**: eccezioni, messaggi di debug, stati null → indicano percorsi/assunzioni.

> **Strumenti**: Burp Proxy/Repeater per manomettere richieste; Comparer per differenze di risposta; Sequencer/Logger++ per tracciare stati.

***

## Checklist tascabile (pentest)

*   [ ] Ho testato **salti di step** e **riordino** delle richieste (forced browsing)?
*   [ ] Ho provato **parametri mancanti** (nome e/o valore) **uno per volta**?
*   [ ] Ho testato **limiti e segni** su tutti i numerici (0, −1, max int, overflow)?
*   [ ] Ho provato **tipi inattesi** (stringhe su numeri, array su scalari, booleani)?
*   [ ] Ho verificato **ricalcoli** (sconti, tasse, ruoli) dopo modifiche tardive?
*   [ ] Ho confrontato **controlli lato UI vs server** (sono allineati ovunque)?
*   [ ] Ho cercato **stati inconsistenti** (doppio submit, replay, step paralleli)?
*   [ ] Ho controllato **cookie/headers nascosti** come vettori di stato?
*   [ ] Ho osservato **errori/log** per informazioni su logica interna?

***

## Best practice (prevenzione per dev/tester)

*   **Documentare i workflow** e le **assunzioni** (per ogni step: input, stato atteso, effetti).
*   **Validazione server-side** rigorosa: tipo, range, formato, semantica (es. quantità ≥ 0).
*   **Controlli coerenti** in **ogni** endpoint/stadio; evitare “salti” di fiducia.
*   **State machine esplicite**: rifiutare richieste fuori sequenza o in stato non valido.
*   **Ricalcolo** di sconti/limiti **al checkout** (non solo all’evento che li ha generati).
*   **Defence‑in‑depth** per funzioni sensibili (re-autenticazione, 2FA per step critici).
*   **Test di confine**/mutazione input in QA; negative testing come requisito.
*   **Evita oracoli** (criptografia): non esporre cifratura/decifratura riusabile cross-endpoint.
*   **Parsing email robusto** e **consistente** tra tutti i componenti.

***

## Mini‑template per il report (utile per te)

**Titolo**: Inconsistenza logica nel calcolo sconto al checkout  
**Severità**: Medium → High (in funzione del dominio)  
**Descrizione**: Lo sconto applicato al superamento soglia non viene ricalcolato se il carrello viene ridotto sotto soglia **dopo** l’applicazione iniziale.  
**Impatto**: Perdita economica; possibilità di checkout con sconto non dovuto.  
**Causa radice**: Assunzione implicita che il carrello resti invariato dopo l’applicazione sconto.  
**Repro ad alto livello** (no exploit step-by-step):

*   Applicare sconto superando soglia → ridurre items prima del pagamento → sconto persiste.  
    **Rimedi**: Ricalcolo sconti al momento del pagamento; validazione server-side su soglie; test QA su variazioni tardive di carrello.  
    **Evidenze**: timestamp richieste + differenze risposta (es. totale/sconto).

***

## Glossario rapido

*   **Business rules**: regole che definiscono come deve comportarsi l’applicazione.
*   **Forced browsing**: accesso diretto agli step/URL fuori dal flusso previsto dalla UI.
*   **Oracle** (crittografia): endpoint che consente di cifrare/decifrare input arbitrari.
*   **State machine**: insieme di stati leciti e transizioni consentite tra di essi.

***

## Riferimenti utili

*   PortSwigger Web Security Academy — *Business logic vulnerabilities*  
    <https://portswigger.net/web-security/logic-flaws>
*   PortSwigger Research — *Splitting the Email Atom: Exploiting Parsers to Bypass Access Controls*  
    <https://portswigger.net/research/splitting-the-email-atom>

***

### Nota etica

Esegui questi test **solo con autorizzazione**. Le tecniche descritte servono per **migliorare la sicurezza** e redigere report professionali, non per causare danni.

***

Se vuoi, posso **adattare questi appunti in un `README.md` pronto per GitHub** con una struttura personalizzata (es. aggiungendo sezioni per i tuoi lab correnti o lo stile del tuo repo). Vuoi anche un **template di checklist in `.md`** stampabile?
