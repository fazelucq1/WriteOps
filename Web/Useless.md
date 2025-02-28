Writeup: Useless 🗂️
====================

Overview
--------

*   **Categoria:** Network
    
*   **Autore:** Giaxo
    
*   **Difficoltà:** 🔴 827/496
    
*   **Stato:** Up
    

Descrizione 📝
--------------

La challenge fornisce un file **capture.pcapng**, rubato da un server importante.L'analisi manuale con Wireshark non sembra fornire informazioni utili, ma un approccio alternativo permette di estrarre la flag.

Soluzione 🎯
------------

### 1\. Analisi del file

Aprendo il file con **Wireshark**, non troviamo pacchetti rilevanti.Proviamo quindi ad analizzarlo con strings per cercare dati nascosti.

### 2\. Ricerca della flag

Utilizziamo il comando:

*bash*

`   strings capture.pcapng | grep "flag{"   `

### Flag 🏁

flag{4lw4y5_ch3ck_th3_c0mm3nt5}

Lezioni Apprese 📚
------------------

*   L'importanza di controllare tutti i dettagli di un file
    
*   Come strings può rivelare informazioni nascoste nei file binari
    
*   Approcci alternativi per analizzare file di rete
    

Tools Utilizzati 🛠️
--------------------

*   Wireshark (per l’analisi del traffico)
    
*   strings (per l’estrazione di testo)
    

_Writeup by Luca Crippa - Last Updated: 2024_
