# Writeup: NS_0.01 - Level1 - Nodes console

## Overview
- **Categoria:** network
- **Autore:** enriquez
- **Difficoltà:** 463
- **Stato:** Up
- **URL:** (esercizio locale tramite Docker)

## Descrizione 📝
Questa sfida è la continuazione della NS_0.00 e ci richiede di accedere ad una shell interattiva sul nodo 1. Dopo aver eseguito il comando `make up` per avviare lo scenario, occorre individuare l'ID del container associato al nodo1 e aprire una shell (ad esempio con `/bin/bash`) per ottenere la flag.

## Soluzione 🎯

### 1. Avviare lo Scenario
- Assicurarsi di aver installato Docker e Docker Compose.
- Decomprimere l'allegato `challenge_files.tar.gz`.
- Eseguire il comando:
  ```bash
  make up
  ```

### 2. Identificare il Container del Nodo1
- Utilizzare il comando:
  ```bash
  docker ps
  ```
  per visualizzare i container in esecuzione. L'output potrebbe essere simile a:
  ```
  CONTAINER ID   IMAGE         COMMAND                  CREATED       STATUS       PORTS     NAMES
  eb284d5665bd   ubuntu-ccit   "/bin/sh -c /usr/loc…"   3 hours ago   Up 3 hours             challenge_files-node1-1
  f6309f5f5dea   ubuntu-ccit   "/bin/sh -c /usr/loc…"   3 hours ago   Up 3 hours             challenge_files-node2-1
  9c0559ba5d68   ubuntu-ccit   "/bin/sh -c /usr/loc…"   3 hours ago   Up 3 hours             challenge_files-node3-1
  ```
- Da questo elenco, individua l'ID del nodo1 (in questo esempio, `eb284d5665bd`).

### 3. Accedere al Container del Nodo1
- Apri una shell interattiva sul container del nodo1 eseguendo:
  ```bash
  docker exec -it eb284d5665bd /bin/bash
  ```
- Una volta all'interno, verrà mostrato un messaggio che indica il contenuto della flag, ad esempio:
  ```
  This is the f1ag: CCIT{level1_954e84dadc40a36201ab}
  ```

## Flag 🏁
```
CCIT{level1_954e84dadc40a36201ab}
```

## Lezioni Apprese 📚
- **Gestione dei Container:** È fondamentale saper utilizzare Docker per elencare e interagire con i container in esecuzione.
- **Accesso Interattivo:** Il comando `docker exec -it` permette di accedere a una shell interattiva, essenziale per risolvere le sfide basate su ambienti containerizzati.
- **Preparazione dell'Ambiente:** La corretta configurazione e pulizia dell'ambiente (tramite `make up`, `make down` e `make clean`) facilita lo sviluppo e la risoluzione delle challenge.

## Tools Utilizzati 🛠️
- **Docker & Docker Compose:** Per creare e gestire l'ambiente della challenge.
- **Terminale Linux:** Per eseguire i comandi e accedere ai container.

---

*Writeup by Luca Crippa - Last Updated: 2024*
