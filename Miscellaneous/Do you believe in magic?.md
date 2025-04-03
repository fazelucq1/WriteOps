# Writeup: Do you believe in magic?

## Descrizione 📝
La challenge fornisce un file che sembra essere un'immagine, ma quando viene aperto non è visualizzabile a causa di errori. Analizzando il file con un editor esadecimale (ad esempio, Hexitor), notiamo che i primi 8 byte non corrispondono alla firma tipica di un file PNG.

- **Firma attuale**:  
  `ba aa aa aa aa aa aa ad`
- **Firma PNG corretta**:  
  `89 50 4E 47 0D 0A 1A 0A`

Il problema risiede quindi nella firma del file: i primi 8 byte sono stati modificati e, modificandoli manualmente per farli corrispondere a quelli di un file PNG, l'immagine può essere aperta correttamente.

## Sfruttamento 🎯

1. **Analisi del file**  
   Utilizzando Hexitor (o un editor esadecimale), abbiamo ispezionato i primi 8 byte del file e osservato che non corrispondono alla firma standard di un PNG.

2. **Correzione della Firma**  
   Abbiamo modificato i byte iniziali in modo che corrispondano a:  
   `89 50 4E 47 0D 0A 1A 0A`

3. **Verifica**  
   Salvando il file con la firma corretta, l’immagine viene aperta senza errori.

## Flag 🏁
```
flag{there_is_nothing_magical_about_magic_bytes}
```

## Lezioni Apprese 📚
1. **Magic Bytes**: La firma di un file (magic bytes) è fondamentale per la sua corretta interpretazione; un semplice errore in questi byte può renderlo inutilizzabile.
2. **Analisi Esadecimale**: Utilizzare strumenti come Hexitor permette di identificare e correggere errori nei file binari.
3. **Manipolazione Manuale**: A volte, la soluzione a una challenge di crittografia o forense risiede in una semplice modifica manuale dei dati a basso livello.

## Tools Utilizzati 🛠️
- **Hexitor**: Per ispezionare e modificare i byte del file.
- **Editor esadecimale**: Per modificare la firma del file PNG.

---

*Writeup by Luca Crippa - Last Updated: 2024*
