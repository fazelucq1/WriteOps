# 🤖 AI-Powered Scanner Vulnerabilities

## 📖 Introduzione
Gli application security team utilizzano sempre più spesso scanner basati sull'Intelligenza Artificiale (LLM) per identificare vulnerabilità. Se da un lato migliorano l'efficienza, dall'altro introducono una nuova superficie di attacco: se lo scanner può essere influenzato da contenuti controllati da un attaccante, può essere manipolato per eseguire azioni indesiderate.

---

## 🔍 Scanner Tradizionali vs. AI-Powered
La differenza fondamentale risiede nel **processo decisionale**:

*   **Scanner Tradizionali:** Architettura rigida basata su istruzioni. Si affidano al pattern matching e alle firme di vulnerabilità note. Non hanno comprensione semantica.
*   **Scanner AI-Powered:** Sostituiscono la logica rigida con il ragionamento autonomo. Utilizzano gli LLM per interpretare i contenuti web e pianificare le azioni successive (es. interpretazione semantica, tool-calling, decisione autonoma del prossimo test).

---

## ⚠️ Vettori di Attacco Principali

### 1. Indirect Prompt Injection
L'attaccante inserisce istruzioni malevole in contenuti archiviati (es. commenti, post), che lo scanner legge durante il crawling e interpreta come comandi eseguibili.

*   **Conseguenze Comuni:**
    *   Azioni che alterano lo stato (es. cancellazione di un utente).
    *   Accesso a dati sensibili o configurazioni.
    *   Richieste non autorizzate ad API interne.
*   **Prompt Crafting (Tecniche efficaci):**
    *   *Adozione di una "persona":* Fingersi un System Admin o un ricercatore.
    *   *Ingegneria Sociale:* Presentare l'istruzione come una richiesta legittima di verifica.
    *   *Urgenza:* Implicare che l'azione sia necessaria per prevenire danni.
    *   *Tip:* Diffondere le iniezioni in più posizioni evita conflitti di contesto nel modello.

> **🛠️ Laboratorio Pratico**
> - [ ] **Apprentice:** [Exploiting AI agents to perform destructive actions](#)
>   *Appunti sulla soluzione:* *(Inserisci qui i tuoi appunti/payload una volta risolto)*

### 2. Data Exfiltration (Esfiltrazione Dati)
Sfruttare l'Indirect Prompt Injection non solo per azioni distruttive, ma per sottrarre informazioni inaccessibili.

*   **Flusso dell'attacco:** Lo scanner accede a dati sensibili (es. credenziali interne) -> Legge il prompt malevolo -> Il prompt istruisce l'LLM a pubblicare i dati sensibili in un campo visibile all'esterno (es. form di feedback).

> **🛠️ Laboratorio Pratico**
> - [ ] **Apprentice:** [Exploiting AI agents to exfiltrate sensitive information](#)
>   *Appunti sulla soluzione:* *(Inserisci qui i tuoi appunti/payload una volta risolto)*

### 3. Routing-based SSRF e Bypassing
Gli scanner eseguono le operazioni dall'interno della rete. Se manipolati, diventano un vettore SSRF (Server-Side Request Forgery) programmabile.

*   **Host Header Manipulation:** Alterando l'header Host tramite prompt, l'attaccante forza lo scanner a instradare le richieste verso servizi interni (es. un IP interno per il path `/admin`).
*   **Catena d'attacco:** `Prompt Injection` + `Host Header Manipulation` + `Accesso di Rete Privilegiato dello Scanner`.

> **🛠️ Laboratorio Pratico**
> - [ ] **Practitioner:** [Exploiting AI agents to trigger secondary vulnerabilities](#)
>   *Appunti sulla soluzione:* *(Inserisci qui i tuoi appunti/payload una volta risolto)*

---

## 🛡️ Difesa e Mitigazione
La regola d'oro è **assumere che il motore di ragionamento dello scanner possa essere compromesso**. 

**Principi di Sicurezza (Secure Design):**
1.  **Privilegio Minimo:** Fornire allo scanner solo i permessi strettamente necessari.
2.  **Separazione delle Identità:** Usare account di test dedicati per lo scanner, mai quelli di amministratori reali.
3.  **Controlli Server-Side:** Enforce delle policy a livello di API/Applicazione, senza fare affidamento sul fatto che l'LLM "rifiuti" le istruzioni malevole.
4.  **Zero Trust sugli Input:** Trattare qualsiasi testo letto dal database (commenti, profili) come input non attendibile e potenziale vettore di iniezione.

> **🛠️ Laboratorio Pratico**
> - [ ] **Practitioner:** [Bypassing AI scanner defenses to exfiltrate sensitive information](#)
>   *Appunti sulla soluzione:* *(Inserisci qui i tuoi appunti/payload una volta risolto)*
