Ecco un comando `ffuf` aggiornato con una wordlist più corposa e un set di estensioni più ampio:

```bash
ffuf -u http://target2.ine.local/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
  -e .php,.html,.js,.css,.txt,.json,.xml,.bak,.old,.tar.gz,.zip,.git \
  -recursion \
  -recursion-depth 3 \
  -mc 200,301,302 \
  -o ffuf_fullscan.json \
  -of json
```

### Cosa cambia

* **Wordlist più sostanziosa**
  Uso `raft-large-directories.txt` (più di 20 000 voci) invece di `common.txt`.

* **Estensioni aggiuntive**
  Oltre a `.php`, `.html`, `.js`, `.css`, `.txt` aggiunte:

  * `.json`, `.xml` (API/data)
  * `.bak`, `.old` (backup)
  * `.tar.gz`, `.zip` (archivi)
  * `.git` (repository nascoste)

* **Profondità di ricorsione aumentata**
  `-recursion-depth 3` per scendere fino a tre livelli di cartelle.

* **Output in JSON**
  Salvato in `ffuf_fullscan.json` per una facile analisi o parsing automatico.

Puoi ovviamente adattare la profondità o la lista di estensioni in base alle esigenze del target. Buona enumerazione!
