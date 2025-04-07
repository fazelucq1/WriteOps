# Writeup della Challenge "Flag Pass" 🚩

In questa challenge l'obiettivo è validare un token tramite una serie di endpoint e operazioni che richiedono il possesso di un token admin. Di seguito viene riportata una descrizione completa del processo, dell'analisi del codice e delle operazioni svolte per ottenere la flag 🎯.

---

## 1. Descrizione della Challenge 🔍

- **Endpoint `/test`:** Viene utilizzato per ottenere un token utente.
- **Endpoint `/pass`:** Usato per verificare il token ottenuto, ma restituisce sempre `false` e un QR code che non porta a nulla.
- **Endpoint `/record_result`:** Permette di settare l'esito a `PASSED` se si fornisce il token admin insieme al proprio token utente.

**Nota:** Per poter utilizzare `/record_result` e avere esito `PASSED`, è necessario disporre del token admin 🔑.

---

## 2. Generazione del Token Admin 🛠️

Il token admin viene calcolato sul lato client attraverso una funzione che effettua una serie di operazioni matematiche e bitwise sui caratteri di una stringa. La funzione verifica, per ciascuna posizione, che il codice del carattere (ottenuto con `charCodeAt`) corrisponda a un determinato valore calcolato con operazioni aritmetiche e bitwise.

Il controllo dei caratteri avviene ad esempio in questo modo:
- **Esempio (Indice 13):**  
  ```javascript
  if (t.charCodeAt(13) === (((-30420 / 13) / 30) + 123) &&
                t.charCodeAt(11) === (((((-33063 / 103) / 107) ^ 86) + 76) + 111) &&
                t.charCodeAt(29) === (((-251 ^ 141) + 64) + 105) &&
                t.charCodeAt(3) === (((((2052 ^ 87) ^ 111) / 68) ^ 91) - 12) &&
                t.charCodeAt(16) === ((((16 ^ 23) + 61) - 119) + 150) &&
                t.charCodeAt(2) === (((-1140 / 95) ^ 82) + 139) &&
                t.charCodeAt(6) === (((20266 - 26) / 115) - 123) &&
                t.charCodeAt(23) === ((((1218 - 57) + 2) ^ 52) / 27) &&
                t.charCodeAt(21) === ((((27 - 113) - 83) + 79) + 141) &&
                t.charCodeAt(14) === ((((((167 ^ 77) ^ 133) - 107) ^ 5) ^ 80) - 29) &&
                t.charCodeAt(27) === ((((((57812794 ^ 50) / 88) + 127) / 90) / 49) - 94) &&
                t.charCodeAt(7) === (((((8375612 / 28) - 139) / 145) + 5) / 39) &&
                t.charCodeAt(18) === ((((((-7664 ^ 69) + 3) - 24) / 56) ^ 133) + 48) &&
                t.charCodeAt(20) === (((((179 ^ 85) + 32) - 117) ^ 149) ^ 55) &&
                t.charCodeAt(25) === ((((((235 + 77) - 77) - 35) - 127) + 24) ^ 5) &&
                t.charCodeAt(24) === (((-14 + 60) ^ 65) - 14) &&
                t.charCodeAt(28) === (((236 - 140) ^ 135) ^ 129) &&
                t.charCodeAt(0) === (((29 ^ 93) / 8) + 48) &&
                t.charCodeAt(12) === (((((16654 ^ 78) ^ 145) / 83) + 25) ^ 130) &&
                t.charCodeAt(8) === ((((154 ^ 126) ^ 118) - 5) - 96) &&
                t.charCodeAt(32) === (((((7 ^ 143) - 112) + 59) - 68) + 38) &&
                t.charCodeAt(4) === (((((10817 + 112) - 120) - 81) / 149) + 28) &&
                t.charCodeAt(15) === ((((((262 ^ 116) ^ 140) ^ 117) - 105) - 133) - 59) &&
                t.charCodeAt(5) === ((((((13809800 / 145) / 10) + 137) ^ 143) / 138) ^ 118) &&
                t.charCodeAt(19) === ((((-428 ^ 83) / 101) + 100) ^ 102) &&
                t.charCodeAt(10) === ((((6720 + 28) + 16) / 76) - 33) &&
                t.charCodeAt(31) === (((15456 / 138) - 14) ^ 3) &&
                t.charCodeAt(17) === (((1657 - 59) + 136) / 34) &&
                t.charCodeAt(1) === ((((((-4323 ^ 53) / 56) + 21) + 43) + 41) + 22) &&
                t.charCodeAt(9) === ((((((34470 / 15) ^ 60) + 122) / 37) - 12) ^ 7) &&
                t.charCodeAt(35) === (((298 - 27) - 146) ^ 24) &&
                t.charCodeAt(26) === (((97 ^ 133) ^ 35) - 97) &&
                t.charCodeAt(33) === (((((12604 / 137) - 95) - 63) - 3) + 122) &&
                t.charCodeAt(34) === (((((342639 / 33) + 121) ^ 3) / 133) ^ 45) &&
                t.charCodeAt(22) === ((((((132974976 / 128) - 3) - 53) + 149) / 148) / 130) &&
                t.charCodeAt(30) === ((((((2319989 + 11) / 80) / 145) ^ 114) ^ 94) ^ 134))
  ```
- Il controllo si ripete per tutte le posizioni (dal 0 al 35), con operazioni differenti ad ogni indice.

---

## 3. Reverse Engineering e Calcolo del Token 🔄

Per risolvere la challenge, è necessario "invertire" il processo e calcolare i valori attesi per ogni carattere. È stato fornito un codice Python che esegue esattamente questa operazione:

- **Conversione in intero a 32 bit:**  
  La funzione `to_32bit_signed(n)` simula il comportamento di JavaScript per ottenere un intero a 32 bit con segno.

- **Calcolo dei caratteri:**  
  Per ciascun indice (0-35) il codice esegue una serie di operazioni aritmetiche, bitwise e divisioni.  
  Ad esempio:
 
```python
    def to_32bit_signed(n):
    """Converte un numero in un intero a 32 bit con segno, come in JavaScript."""
    n = n & 0xFFFFFFFF  # Maschera a 32 bit
    if n & 0x80000000:  # Se il bit di segno è settato
        return n - 0x100000000  # Converti in valore negativo
    return n

# Inizializza un array per i codici dei caratteri (36 posizioni)
char_codes = [0] * 36

# Indice 0
bu = 29 ^ 93
bv = bu / 8
bw = bv + 48
char_codes[0] = int(bw)

# Indice 1
dq = -4323 ^ 53
dr = dq / 56
ds = dr + 21
dt = ds + 43
du = dt + 41
dv = du + 22
char_codes[1] = int(dv)

# Indice 2
r = -1140 / 95
s = to_32bit_signed(int(r) ^ 82)
t = s + 139
char_codes[2] = int(t)

# Indice 3
i = 2052 ^ 87
j = i ^ 111
k = j / 68
l = to_32bit_signed(int(k) ^ 91)
m = l - 12
char_codes[3] = int(m)

# Indice 4
cl = 10817 + 112
cm = cl - 120
cn = cm - 81
co = cn / 149
cp = co + 28
char_codes[4] = int(cp)

# Indice 5
cw = 13809800 / 145
cx = cw / 10
cy = cx + 137
cz = to_32bit_signed(int(cy) ^ 143)
da = cz / 138
db = to_32bit_signed(int(da) ^ 118)
char_codes[5] = int(db)

# Indice 6
u = 20266 - 26
v = u / 115
w = v - 123
char_codes[6] = int(w)

# Indice 7
ar = 8375612 / 28
at = ar - 139
au = at / 145
av = au + 5
aw = av / 39
char_codes[7] = int(aw)

# Indice 8
cc = 154 ^ 126
cd = cc ^ 118
ce = cd - 5
cf = ce - 96
char_codes[8] = int(cf)

# Indice 9
dw = 34470 / 15
dx = to_32bit_signed(int(dw) ^ 60)  # Correzione applicata
dy = dx + 122
dz = dy / 37
ea = dz - 12
eb = to_32bit_signed(int(ea) ^ 7)
char_codes[9] = int(eb)

# Indice 10
dg = 6720 + 28
dh = dg + 16
di = dh / 76
dj = di - 33
char_codes[10] = int(dj)

# Indice 11
a = -33063 / 103
b = a / 107
c = to_32bit_signed(int(b) ^ 86)
d = c + 76
e = d + 111
char_codes[11] = int(e)

# Indice 12
bx = 16654 ^ 78
by = bx ^ 145
bz = by / 83
ca = bz + 25
cb = to_32bit_signed(int(ca) ^ 130)
char_codes[12] = int(cb)

# Indice 13
char_codes[13] = int((((-30420 / 13) / 30) + 123))

# Indice 14
af = 167 ^ 77
ag = af ^ 133
ah = ag - 107
ai = ah ^ 5
aj = to_32bit_signed(ai ^ 80)
ak = aj - 29
char_codes[14] = int(ak)

# Indice 15
cq = 262 ^ 116
cr = cq ^ 140
cs = cr ^ 117
ct = cs - 105
cu = ct - 133
cv = cu - 59
char_codes[15] = int(cv)

# Indice 16
n = 16 ^ 23
o = n + 61
p = o - 119
q = p + 150
char_codes[16] = int(q)

# Indice 17
dn = 1657 - 59
do = dn + 136
dp = do / 34
char_codes[17] = int(dp)

# Indice 18
ax = -7664 ^ 69
ay = ax + 3
az = ay - 24
ba = az / 56
bb = to_32bit_signed(int(ba) ^ 133)
bc = bb + 48
char_codes[18] = int(bc)

# Indice 19
dc = -428 ^ 83
dd = dc / 101
de = dd + 100
df = to_32bit_signed(int(de) ^ 102)
char_codes[19] = int(df)

# Indice 20
bd = 179 ^ 85
be = bd + 32
bf = be - 117
bg = bf ^ 149
bh = to_32bit_signed(bg ^ 55)
char_codes[20] = int(bh)

# Indice 21
ab = 27 - 113
ac = ab - 83
ad = ac + 79
ae = ad + 141
char_codes[21] = int(ae)

# Indice 22
es = 132974976 / 128
et = es - 3
eu = et - 53
ev = eu + 149
ew = ev / 148
ex = ew / 130
char_codes[22] = int(ex)

# Indice 23
x = 1218 - 57
y = x + 2
z = y ^ 52
aa = z / 27
char_codes[23] = int(aa)

# Indice 24
bo = -14 + 60
bp = to_32bit_signed(bo ^ 65)
bq = bp - 14
char_codes[24] = int(bq)

# Indice 25
bi = 235 + 77
bj = bi - 77
bk = bj - 35
bl = bk - 127
bm = bl + 24
bn = to_32bit_signed(bm ^ 5)
char_codes[25] = int(bn)

# Indice 26
ef = 97 ^ 133
eg = ef ^ 35
eh = eg - 97
char_codes[26] = int(eh)

# Indice 27
al = 57812794 ^ 50
am = al / 88
an = am + 127
ao = an / 90
ap = ao / 49
aq = ap - 94
char_codes[27] = int(aq)

# Indice 28
br = 236 - 140
bs = br ^ 135
bt = to_32bit_signed(bs ^ 129)
char_codes[28] = int(bt)

# Indice 29
f = to_32bit_signed(-251 ^ 141)
g = f + 64
h = g + 105
char_codes[29] = int(h)

# Indice 30
ey = 2319989 + 11
ez = ey / 80
fa = ez / 145
fb = to_32bit_signed(int(fa) ^ 114)  # Correzione applicata
fc = fb ^ 94
fd = to_32bit_signed(fc ^ 134)
char_codes[30] = int(fd)

# Indice 31
dk = 15456 / 138
dl = dk - 14
dm = to_32bit_signed(int(dl) ^ 3)
char_codes[31] = int(dm)

# Indice 32
cg = 7 ^ 143
ch = cg - 112
ci = ch + 59
cj = ci - 68
ck = cj + 38
char_codes[32] = int(ck)

# Indice 33
ei = 12604 / 137
ej = ei - 95
ek = ej - 63
el = ek - 3
em = el + 122
char_codes[33] = int(em)

# Indice 34
en = 342639 / 33
eo = en + 121
ep = to_32bit_signed(int(eo) ^ 3)  # Correzione applicata
eq = ep / 133
er = to_32bit_signed(int(eq) ^ 45)
char_codes[34] = int(er)

# Indice 35
ec = 298 - 27
ed = ec - 146
ee = to_32bit_signed(ed ^ 24)
char_codes[35] = int(ee)

# Costruisci il token convertendo i codici in caratteri
token = ''.join(chr(code) for code in char_codes)
print("Il token è:", token)
```

- **Generazione del token admin:**  
  Dopo aver calcolato i valori numerici per ogni posizione, questi vengono convertiti in caratteri con la funzione `chr()` e concatenati per formare il token admin. Il codice Python stampava:

  ```
  Il token è: 8218d355-38ff-4bc3-9336-adf7f1ba55be
  ```

---

## 4. Utilizzo del Token Admin per Ottenere il PASSED ✅

Dopo aver ottenuto il token admin, il passo finale consiste nel fornire sia il token admin che il proprio token utente all'endpoint `/record_result`. In questo modo, il server riconosce la validità dell'operazione e imposta il risultato della challenge a **PASSED**.

- **Token Admin ottenuto:**  
  `8218d355-38ff-4bc3-9336-adf7f1ba55be`
- **Token Utente (ad esempio):**  
  `2cdbc626-dba4-464d-9e22-155851735303`

Una volta inviati i token corretti, il sistema registra l'esito e per ottenere la flag è sufficiente scannerizzare il QR code fornito 📱.

---

## 5. Flag 🏆

La flag per la challenge è:

```
flag{1_l0v3_cl13nt_ch3cks_<3}
```

---

## Conclusioni 📌

1. **Analisi del Controllo:**  
   È stato necessario esaminare il codice JavaScript che controlla ogni carattere del token admin, comprendendo le operazioni matematiche e bitwise coinvolte.

2. **Reverse Engineering:**  
   È stato sviluppato uno script Python per "invertire" il controllo e calcolare i codici corretti per ogni posizione, ottenendo così il token admin.

3. **Utilizzo della Logica:**  
   Utilizzando il token admin insieme al token utente, l'endpoint `/record_result` permette di settare l'esito della challenge a PASSED e ottenere la flag tramite il QR code.

Questa writeup spiega in modo semplice e dettagliato come è stato possibile risolvere la challenge sfruttando il reverse engineering del token admin e l'uso corretto degli endpoint messi a disposizione.

---

*Writeup by Luca Crippa - Last Updated: 2025*
