# Writeup: OS_2.07 - Domain 7

**Domanda:** Qual è il più recente Google site verification code registrato nei record TXT di `libero.it`?

---

## Soluzione

1. **Query dei record TXT**  
   Eseguiamo il comando:
   ```bash
   dig libero.it TXT +short
   ```
2. **Filtriamo i record relativi a Google**  
   ```bash
   dig libero.it TXT +short | grep "google"
   ```
   Risultano tre record:
   ```
   "google-site-verification=fqsYdy3wwvDP-i736PKI7o6xfl203FX5pvS53EyScLM"
   "google-site-verification=rwDHe3W-KaW0jUSCtpGSk5UWBkwhAOpw2mQdmn4NQLw"
   "google-site-verification=Uhj3hk68gf5xSOTkWZsxeUYymAtnGCEa5qzQwgvKato"
   ```

3. **Identifichiamo il più recente**  
   L’ordine in cui compaiono non è sempre affidabile, ma di solito quello in cima è l’ultimo inserito. Qui è:
   
---

## Risposta
```
fqsYdy3wwvDP-i736PKI7o6xfl203FX5pvS53EyScLM
```
