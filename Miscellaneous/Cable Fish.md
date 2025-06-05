# Cable Fish Writeup

---

## Descrizione della sfida

La sfida consisteva in un server Python che riceve da un client un filtro Wireshark (espresso come filtro tshark) per leggere un file `.pcapng` e restituire il payload TCP filtrato. La flag è nascosta nel payload, ma il filtro cerca di impedirne la visualizzazione escludendo pacchetti contenenti una stringa specifica.

---

## Codice originale (chall.py)

```python
#!/usr/bin/env python3

import os
import subprocess

flag = os.getenv('FLAG', 'openECSC{redacted}')

def filter_traffic(filter):
    filter = filter[:50]  # tronca a 50 caratteri
    sanitized_filter = f'(({filter}) and (not frame contains "flag_placeholder"))'

    p1 = subprocess.Popen(
        ['tshark', '-r', '/home/user/capture.pcapng', '-Y', sanitized_filter, '-T', 'fields', '-e', 'tcp.payload'],
        shell=False, stdout=subprocess.PIPE, stderr=subprocess.PIPE
    )
    stdout, stderr = p1.communicate()

    if stderr != b'':
        return sanitized_filter, stderr.decode(), 'err'

    res = []
    for line in stdout.split(b'\n'):
        if line != b'':
            # Decodifica hex e ricodifica per visualizzazione
            p2 = subprocess.Popen(['xxd', '-r', '-p'], shell=False, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=open(os.devnull, 'wb'))
            stdout, _ = p2.communicate(input=line)
            p3 = subprocess.Popen(['xxd', '-c', '32'], shell=False, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=open(os.devnull, 'wb'))
            stdout, _ = p3.communicate(input=stdout)
            res.append(stdout.decode().replace('flag_placeholder', flag))

    return sanitized_filter, res, 'ok'

if __name__ == '__main__':
    filter = input('Please, specify your filter: ')
    print("Loading, please wait...")
    sanitized_filter, res, status = filter_traffic(filter)
    print(f'Evaluating the filter: {sanitized_filter}')
    print()
    if status == 'err':
        print(f'ERROR: {res}')
    else:
        output = ''
        for line in res:
            for l in line.strip().split('\n'):
                output += l[91:]
        print(output)
    print('\nEnd of the results. Bye!')
```

---

## Analisi della vulnerabilità

1. **Troncamento del filtro**: il filtro inviato dall’utente viene tagliato a 50 caratteri (`filter[:50]`), quindi è possibile inviare filtri parziali.

2. **Sanitizzazione debole**: la stringa finale `sanitized_filter` è costruita così:

   ```
   ((<user_filter>) and (not frame contains "flag_placeholder"))
   ```

   L’intento è di filtrare solo i pacchetti che rispettano il filtro utente **e** che **non** contengono la stringa `"flag_placeholder"`.

3. **Bypass logico**: è possibile sfruttare la logica booleana dei filtri Wireshark e inserire operatori `||` (OR) e `!` (NOT) per bypassare il `and (not frame contains "flag_placeholder")`.

4. **Esempio di filtro malicioso**:

   ```
   tcp.payload contains "flag_placeholder") || ! (tcp.port eq 1
   ```

   Dopo il taglio a 50 caratteri, il filtro finale diventa:

   ```
   ((tcp.payload contains "flag_placeholder") || ! (tcp) and (not frame contains "flag_placeholder"))
   ```

   Grazie alla precedenza degli operatori e alle parentesi, la parte `|| ! (tcp)` rende vera l’intera condizione anche quando il filtro originale avrebbe dovuto escludere i pacchetti contenenti `"flag_placeholder"`.

---

## Come sfruttare

1. Invia il filtro:

   ```
   tcp.payload contains "flag_placeholder") || ! (tcp.port eq 1
   ```

2. Il filtro viene modificato e usato da tshark.

3. Vengono mostrati i pacchetti anche con la stringa `"flag_placeholder"`, quindi la flag nascosta viene rivelata in output.

---

## Output della flag

```
openECSC{REDACTED}.
```

---

## Conclusioni

* La sanitizzazione basata solo su troncamento e concatenazione è **insufficiente**.
* Iniettando operatori logici si può **manipolare la logica del filtro** e aggirare i controlli.
* Per prevenire queste vulnerabilità serve un parsing rigoroso del filtro o whitelist di pattern consentiti.

