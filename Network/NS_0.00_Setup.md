# Writeup: NS_0.00 - Level0 - Setup

## Overview
- **Categoria:** network
- **Autore:** enriquez
- **Difficoltà:** 457
- **Stato:** Up
- **URL:** (esercizio locale tramite docker)

## Descrizione 📝
Questa challenge è una fase di setup per iniziare a lavorare con Docker e Docker Compose. L'obiettivo principale è preparare l'ambiente necessario per l'esercizio, eseguire il comando `make up` per avviare lo scenario e poi visualizzare i log dei container con `make logs` per trovare la flag.

Le istruzioni fornite sono:
- Installare Docker e Docker Compose seguendo le guide ufficiali:
  - [Docker Install](https://docs.docker.com/engine/install/)
  - [Docker Compose Install](https://docs.docker.com/compose/install/)
- Scaricare il file `challenge_files.tar.gz` fornito.
- Eseguire i comandi:
  - `make up` per avviare lo scenario.
  - `make logs` per visualizzare i log dei container.

Analizzando i log generati, si potrà identificare la flag nascosta nel flusso di output.

## Soluzione 🎯

### 1. Preparazione dell'Ambiente
- **Installazione:** Seguire le guide ufficiali per installare Docker e Docker Compose sul proprio sistema.
- **Download:** Scaricare l'archivio `challenge_files.tar.gz` e decomprimerlo in una directory di lavoro.

### 2. Avvio dello Scenario
- Eseguire il comando:
  ```bash
  make up
  ```
  Questo comando avvia i container necessari per lo scenario.

### 3. Estrazione della Flag
- Una volta avviato lo scenario, visualizzare i log dei container con:
  ```bash
  make logs
  ```
  Analizzando il log, si potrà individuare la flag nascosta

## Flag 🏁
```
CCIT{level0_p4rt1_p4rt2_p4rt3_end}
```

### *Opzionale*. Pulizia dell'Ambiente
- Terminare lo scenario e pulire la configurazione eseguendo:
  ```bash
  make down
  make clean
  ```

## Lezioni Apprese 📚
- **Setup dell'Ambiente:** L'importanza di un corretto setup dell'ambiente tramite Docker per simulare scenari di rete.
- **Log Analysis:** Utilizzare i log generati dai container per estrarre informazioni sensibili e trovare la flag.
- **Automazione con Makefile:** L'uso di Makefile semplifica l'avvio e la gestione di scenari complessi in ambienti virtualizzati.

## Tools Utilizzati 🛠️
- **Docker & Docker Compose:** Per creare e gestire i container.
- **Terminale Linux:** Per eseguire i comandi `make up`, `make logs`, `make down` e `make clean`.
- **Editor di Testo:** Per esaminare eventuali file di configurazione e i Makefile.

---

*Writeup by Luca Crippa - Last Updated: 2024*
