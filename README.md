# Controller încălzire în pardoseală cu ESPHome și Home Assistant

## De ce ?

Controllerele chinezești sunt mai suspecte, și pe lângă asta, scumpe. Logica din codul lor nu este cea mai grozavă, și plus de asta, nu pot fi controlate prin Home Assistant

Acest proiect controlează încălzirea prin intermediul unui ESP8266 (D1 Mini), gestionând până la **6 zone** cu electrovalve și o **pompă + centrală termică**

Părți bune:

- Protecție la pornirea și oprirea pompei și centralei printr-un delay (default 180s) configurabil din Home Assistant.
- Protecție împotriva opririi unei zone care ar lăsa singura zonă activă pornită mai puțin decât timpul de delay.
- Mod de lucru **Automat** sau **Manual** selectabil din Home Assistant.
- Relee virtuale expuse către Home Assistant, cu blocare vizuală în modul automat.
- Debounce (default 30s) configurabil pentru intrările termostatelor, prevenind comenzi false la schimbarea rapidă a stării.
- Persistența valorilor critice pentru a evita pierderea setărilor la restart.
- Stare `unavailable` în Home Assistant dacă ESP nu este conectat.
- În modul **automat** funcționează ca un controller obișnuit (dacă o zonă necesită căldură și nicio altă zonă nu este activă, așteaptă `delay` secunde după care pornește pompa și centrala; dacă o altă zonă necesită în călzire în timp ce mai e o altă zonă activă, se comandă doar actuatorul zonei; dacă nu mai există nici o zonă care necesită încălzire, se oprește centrala și pompa)
- În modul **manual** singurul lucru modificat este faptul că în momentul în care o zonă cere încălzire, nu mai cuplează automat actuatorul zonei respective, ci așteaptă cuplare din automatizare HA; în rest, toate protecțiile sunt active.
- La pierderea conexiunii cu HA, dacă contollerul este setat pe mod manual, se trece singur în mode automat; la revenirea conexiunii cu HA, se restabilește modul de operare setat în HA (fie manual, fie automat)

---

## Schema de conectare electrică

### Intrări (termostate)

**ATENȚIE !!! NU SE CONECTEAZĂ DIRECT TERMOSTATUL LA ESP ! TREBUIE TRECUT PRINTR-UN OPTOCUPLOR / RELEU SAU ORICE ALTCEVA CARE ÎN MOMENTUL ÎN CARE TERMOSTATUL COMANDĂ ÎNCHIDEREA, SĂ FIE TRANSLATAT DIN TENSIUNEA DE COMANDĂ A TERMOSTATULUI ÎN PULLUP LA 3.3v A ESP !**

Fiecare termostat este conectat la un pin digital al ESP8266:

| Zonă | Pin GPIO |
|------|----------|
| Zona 1 | D1 |
| Zona 2 | D2 |
| Zona 3 | D5 |
| Zona 4 | D6 |
| Zona 5 | D7 |
| Zona 6 | D0 |

> Recomandat: rezistență de pull-down internă activată.

### Ieșiri (relee electrovalve și pompă + centrală)

| Funcție | Pin GPIO | Observații |
|---------|----------|------------|
| Releu Zona 1 | D3 | Ieșire digitală către releu electrovalvă |
| Releu Zona 2 | D4 | „ |
| Releu Zona 3 | RX | „ |
| Releu Zona 4 | TX | „ |
| Releu Zona 5 | D8 | „ |
| Releu Zona 6 | A0 | „ |
| Pompa + Centrală | D9 | Controlează pompa și releul centralei |

- Ieșirile sunt expuse în HA doar prin **relee virtuale**.
- În **modul automat**, acestea sunt **inhibate vizual** și nu pot fi modificate direct.
- În **modul manual**, pot fi controlate direct din Home Assistant.

---

## Logica sistemului

**Mod automat**

- Când o zonă cere căldură și nu există nicio altă zonă activă, se activează releul zonei și începe **delay-ul configurabil** pentru pornirea pompei + centralei (implicit 180s, ajustabil din HA).
- Dacă mai multe zone cer încălzire, fiecare zonă se activează imediat, dar **pompa + centrala respectă delay-ul** la pornire.
- Dacă o zonă ar rămâne singură activă, dar a fost activată mai puțin decât timpul de delay, zonele care cer oprire sunt **forțate să rămână active** până expiră delay-ul.
- După expirarea delay-ului, zonele forțate se închid **automat**.
- Pompa + centrala se opresc imediat când **nu mai există nicio zonă activă**.

**Mod manual**

- Termostatele nu activează automat zonele.
- Releele virtuale pot fi controlate direct din HA.
- Toate protecțiile rămân active: delay la pornire, amânare la oprire.

**Debounce**

- Termostatele sunt considerate active/inactive numai dacă mențin starea mai mult decât intervalul configurat (0–300s, configurabil din HA).

**Persistență și unavailable**

- Valorile critice (zone active, delay-uri, modul de control) sunt stocate persistent.
- Dacă ESP se deconectează, starea în HA devine `unavailable`.

---

## Configurări disponibile în Home Assistant

- **Dropdown "Mod Control"**: Auto / Manual.
- **Număr (seconds) "Delay pornire boiler"**: 30–600s.
- **Număr (seconds) "Debounce termostate"**: 0–300s.
- **Relee virtuale** pentru fiecare zonă și pentru pompă + centrală.

---

## Logs și monitorizare

- ESPHome logger oferă informații despre:
  - Zone active / inactive.
  - Pompa + centrala pornite / oprite.
  - Zone forțate să rămână active.
  - Modul de lucru (Auto / Manual).

---

## TODO

- Integrare doi senzori de temperatură pentru monitorizare tur/retur.
- ... vedem...

---
