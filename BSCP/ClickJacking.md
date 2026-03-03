# Clickjacking (UI Redressing) 

> **Obiettivo**: capire rapidamente cos’è il clickjacking, come si manifesta, come si testa in modo etico e come si previene lato server.  
> **Per il tuo flusso di lavoro**: checklist operativa + mini‑template di report.

***

## TL;DR

*   **Clickjacking**: l’utente clicca su un elemento di una **pagina esca** ma, tramite sovrapposizione (es. `iframe` trasparente), in realtà attiva un’azione su un **sito bersaglio**.
*   Diverso da **CSRF**: nel clickjacking l’utente **clicca davvero**, ma su un **UI nascosto**; nel CSRF viene forgiata una richiesta **senza interazione**.
*   **Mitigazione efficace**: lato server con **`X-Frame-Options`** e soprattutto **`Content-Security-Policy: frame-ancestors ...`**; gli **script frame-buster** sono **bypassabili** e non sufficienti da soli.

***

## Cos’è il clickjacking (in breve)

Un attacco **UI-based** in cui un sito malevolo **incorpora** (tipicamente in `iframe`) una pagina legittima e **occulta** l’interfaccia, inducendo l’utente a compiere azioni indesiderate (pagamenti, modifiche profilo, like, ecc.) **cliccando** su elementi visivamente innocui della pagina esca.

> **LAB**: *Basic clickjacking with CSRF token protection (APPRENTICE)*

***

## Differenza con CSRF (rapida)

*   **Clickjacking**: richiede **interazione** dell’utente (click), ottenuta con **redressing dell’interfaccia** (overlay/trasparenze/posizionamento).
*   **CSRF**: l’utente non deve cliccare il bottone target; la richiesta malevola viene **forgiata** (es. auto-submit form, beacon, ecc.).
*   **CSRF token** **non** impedisce clickjacking: la sessione è valida, il form è caricato **sul dominio legittimo** dentro l’`iframe`; il token viene inviato normalmente.

***

## Come funziona l’attacco (concetti, no PoC step-by-step)

*   **Layering CSS**: sovrapposizione di un `iframe` sul contenuto della pagina esca (z-index) + **bassa opacità** (quasi trasparente).
*   **Allineamento UI**: l’elemento cliccabile del target (es. “Paga”) viene **perfettamente sovrapposto** al bottone-esca.
*   **Anti-transparency heuristics**: alcuni browser applicano **soglie** (es. trasparenza estrema) per mitigare, ma non è una protezione affidabile e **non uniforme** tra browser.
*   **Multistep**: l’attacco può richiedere **più click** (es. aggiungi al carrello → conferma → paga) usando **più div/iframe** coordinati.

> **LAB**:
>
> *   *Clickjacking with form input data prefilled from a URL parameter (APPRENTICE)*
> *   *Multistep clickjacking (PRACTITIONER)*

***

## Strumenti utili

*   **Burp Clickbandit**: registra le azioni effettuate su una pagina frameable e genera un **overlay PoC** di supporto alla verifica. Abbatte tempi e complessità nella creazione di demo etiche per il report.

***

## Varianti comuni

*   **Prefilled form input**: se il sito **precompila campi da GET**, l’`iframe` può veicolare valori scelti dall’attaccante e posizionare l’overlay solo sul **submit** (riducendo i click necessari).
    > **LAB**: *Clickjacking with form input data prefilled from a URL parameter (APPRENTICE)*

*   **Frame buster script**: script lato client che tentano di **uscire dal frame** o **impedire il click** su elementi invisibili.
    *   **Limiti**: sono **bypassabili** (es. `iframe sandbox="allow-forms"` **senza** `allow-top-navigation` neutralizza i controlli sul top window); dipendono da JS e da impostazioni del browser.
    > **LAB**: *Clickjacking with a frame buster script (APPRENTICE)*

*   **Combo con DOM XSS**: il clickjacking diventa “carrier” per un payload XSS lato DOM noto; l’`iframe` punta a un URL che innesca l’XSS **al click**.
    > **LAB**: *Exploiting clickjacking vulnerability to trigger DOM-based XSS (PRACTITIONER)*

*   **Multistep clickjacking**: orchestrazione di più `iframe/div` per condurre l’utente attraverso **più azioni sequenziali** (es. add → checkout → confirm).
    > **LAB**: *Multistep clickjacking (PRACTITIONER)*

***

## Metodologia di testing (pensiero e passaggi ad alto livello)

1.  **Verifica incorniciabilità**: prova a caricare pagine sensibili in un `iframe` (stesso dominio e cross‑origin); controlla se il browser **blocca** o se in rete compaiono intestazioni di difesa.
2.  **Caccia ai target**: identifica **azioni ad alto impatto** (transazioni, cambio email/password, approvazioni).
3.  **Prefill & GET params**: verifica se campi critici possono essere **precompilati** da URL.
4.  **Frame buster**: se presenti, verifica se un **`sandbox` mirato** li neutralizza (senza eseguire exploit; osserva solo il comportamento).
5.  **Multistep**: annota sequenze e transizioni UI (focus/scroll/coordinate) che potrebbero essere **coerce‑abili** a più click.
6.  **Evidenze**: screenshot/registrazioni etiche (es. tramite Clickbandit) e tracce di rete (senza eseguire operazioni distruttive).

> **Nota etica**: esegui test **solo con autorizzazione**, evita azioni che causino **danno reale** (es. transazioni vere) e usa ambienti di prova ove possibile.

***

## Prevenzione (best practice lato server)

### 1) `Content-Security-Policy: frame-ancestors`

*   **Direttiva consigliata**: controlla **chi può incorniciare** le tue pagine.
    *   Blocca tutto:
        ```http
        Content-Security-Policy: frame-ancestors 'none';
        ```
    *   Consenti **solo stesso origin**:
        ```http
        Content-Security-Policy: frame-ancestors 'self';
        ```
    *   Consenti domini specifici (esempio):
        ```http
        Content-Security-Policy: frame-ancestors 'self' https://partner.example.com;
        ```
*   **Vantaggi**: più flessibile e moderno di XFO; supporta **più origin** e casi d’uso granulari.
*   **Applicazione**: deve essere inviato **dalla risposta della pagina da proteggere** (non dall’embedder).

### 2) `X-Frame-Options` (XFO)

*   **Storico/compatibilità**: utile come **difesa in profondità**; supporto non uniforme per `ALLOW-FROM` (obsoleto/non supportato in moderni Chrome/Safari).
    *   Blocca tutto:
        ```http
        X-Frame-Options: DENY
        ```
    *   Consenti solo stesso origin:
        ```http
        X-Frame-Options: SAMEORIGIN
        ```
*   **Nota**: preferire **CSP `frame-ancestors`** come meccanismo primario; mantenere XFO per **legacy**.

### 3) Altre misure

*   **Riduci superfici sensibili incorniciabili**: applica le intestazioni su **tutte le pagine critiche** (non solo homepage).
*   **UI hardening**: conferme esplicite, re‑autenticazione per azioni critiche, indicatori visivi anti‑overlay (limitato).
*   **Frame‑buster scripts**: usali **solo** come **difesa complementare**; non sono affidabili da soli.
*   **Testing continuo**: integra scanner/web scans che controllino **assenza/presenza** di `frame-ancestors`/XFO su path nuovi.

***

## Esempi pratici (anonimizzati)

*   **Sito A** consente framing su pagine di **pagamento** (niente CSP/XFO): un sito B può sovrapporre un `iframe` su un **“Gioca e vinci”** e indurre il click sul tasto **“Paga”**.
*   **Portale B** accetta **prefill via GET** su form di cambio email: con overlay, basta posizionare il bottone **“Conferma”** sopra un CTA visibile.
*   **App C** ha frame‑buster JS: un embed con `sandbox="allow-forms"` senza `allow-top-navigation` impedisce al JS di “rompere il frame”.

***

## Checklist tascabile (pentest)

*   [ ] Le pagine sensibili **includono** `Content-Security-Policy: frame-ancestors` adeguato?
*   [ ] È presente **`X-Frame-Options`** come fallback (DENY/SAMEORIGIN)?
*   [ ] Alcune pagine **mancano** le intestazioni (es. error, legacy, sotto‑percorsi)?
*   [ ] Il sito accetta **prefill via GET** su form critici?
*   [ ] **Frame‑buster** presenti? Sono aggirabili via `sandbox` (osservazione non distruttiva)?
*   [ ] Esistono **azioni multistep** che potrebbero essere forzate via overlay sequenziali?
*   [ ] UI critica richiede **conferma**/re‑auth?
*   [ ] Scanner/CI rilevano regressioni su CSP/XFO?

***

## Mini‑template di report

**Titolo**: Assenza di `frame-ancestors`/XFO su pagine sensibili → Clickjacking  
**Severità**: Medium → High (in base all’azione impattata, es. pagamenti/profilo)  
**Descrizione**: Le pagine \[URL/percorsi] risultano **frameable** in contesto cross‑origin. L’assenza di `Content-Security-Policy: frame-ancestors` e `X-Frame-Options` consente **clickjacking** su azioni sensibili.  
**Impatto**: Possibile induzione al click dell’utente su operazioni ad alto impatto (pagamenti/modifiche).  
**Evidenze**: Caricamento in `iframe` da dominio terzo; request/response **senza** CSP/XFO; registrazione overlay etica.  
**Cause radice**: Intestazioni mancanti/solo parzialmente applicate; assenza di test regressione su header.  
**Rimedi**: Impostare `Content-Security-Policy: frame-ancestors 'none'|'self'|<origini>` su tutte le pagine critiche; mantenere `X-Frame-Options` come fallback; test automatizzati.

***

## Labs di riferimento (solo nome, niente soluzioni)

*   *Basic clickjacking with CSRF token protection (APPRENTICE)*
*   *Clickjacking with form input data prefilled from a URL parameter (APPRENTICE)*
*   *Clickjacking with a frame buster script (APPRENTICE)*
*   *Exploiting clickjacking vulnerability to trigger DOM-based XSS (PRACTITIONER)*
*   *Multistep clickjacking (PRACTITIONER)*

***

## Glossario rapido

*   **UI redressing**: manipolazione del layout per **ingannare** l’utente sul target del click.
*   **Frame busting**: script che cercano di **impedire** l’esecuzione del sito dentro un frame.
*   **`frame-ancestors`**: direttiva CSP che limita **chi** può incorniciare la pagina.
*   **XFO**: header storico (`DENY`, `SAMEORIGIN`) per limitare l’embedding in frame.
