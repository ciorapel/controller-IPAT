🏠 Controller Inteligent de Încălzire ESPHome
=============================================

> Controller avansat de încălzire în pardoseală bazat pe ESP8266 cu integrare Home Assistant

🎯 Prezentare generală
----------------------

Controller de încălzire de nivel profesional care gestionează până la **5 zone** cu electrovalve și un **o centrală termică (și) pompă de recirculare** prin intermediul unui ESP8266 (D1 Mini). Construit ca o alternativă fiabilă la controllerele chinezești scumpe cu funcționalitate limitată și fără integrare Home Assistant.

### ✨ Caracteristici principale

*   🛡️ **Sistem inteligent de protecție**: Întârzieri configurabile pentru pornirea/oprirea centralei (implicit 180s)
    
*   🔒 **Protecție anti-ciclare**: Previne închiderea prea rapidă a zonelor pentru a evita ciclarea scurtă a centralei
    
*   🎛️ **Moduri duale de operare**: Automat (control termostate) sau Manual (control HA)
    
*   🏠 **Integrare completă Home Assistant**: Switch-uri virtuale cu indicatori vizuali de mod
    
*   💾 **Stare persistentă**: Valorile critice supraviețuiesc restarturilor ESP
    
*   🌐 **Monitorizare în timp real**: Raportare și diagnosticare a stării
    
*   📱 **Gestionarea stării indisponibile**: Gestionarea adecvată a stării HA în timpul deconectărilor
    

🔧 Cerințe hardware
-------------------

### Componente

*   ESP8266 D1 Mini
    
*   5x Module releu (pentru valve zone)
    
*   1x Modul releu (pentru centrală)
    
*   5x Optocuploare/relee pentru intrări termostate
    
*   Sursă de alimentare
    

### ⚠️ Avertisment important de siguranță

**NICIODATĂ nu conectați termostatele direct la pinii ESP!** Folosiți întotdeauna optocuploare sau relee pentru a transla tensiunea termostatului în semnale pull-up de 3.3V sigure pentru ESP.

🔌 Schema de conectare
----------------------

### Intrări termostate

| Zonă | Pin GPIO |
|------|----------|
| Zona 1 | D1 |
| Zona 2 | D2 |
| Zona 3 | D5 |
| Zona 4 | D6 |
| Zona 5 | D7 |

### Ieșiri relee

| Funcție | Pin GPIO | Observații |
|---------|----------|------------|
| Releu Zona 1 | D3 | Releu electrovalvă |
| Releu Zona 2 | D4 | „ |
| Releu Zona 3 | RX | „ |
| Releu Zona 4 | TX | „ |
| Releu Zona 5 | D8 | „ |
| Pompa + Centrală | D0 | Controlează pompa și releul centralei |

> **Notă**: Folosirea pinilor RX/TX dezactivează debugging-ul serial

🏗️ Instalare
-------------

### 1\. Cerințe preliminare

*   [ESPHome](https://esphome.io/) instalat
    
*   [Home Assistant](https://home-assistant.io/) funcțional
    
*   Mediu de dezvoltare ESP8266
    

### 2\. Configurare

1.  Copiați configurația ESPHome furnizată
    
2.  wifi\_ssid: "numele\_wifi\_tau"wifi\_password: "parola\_wifi\_tau" (fie în controller.yaml fie în secrets.yaml)
    
3.  Ajustați atribuțiile de pini dacă este necesar în secțiunea substitutions
    
4.  Flash-uiți ESP8266
    

### 3\. Configurare Home Assistant

După flash-uire, dispozitivul va apărea automat în Home Assistant ca heating\_controller cu toate entitățile expuse.
Se poate folosi fără Home Assistant dar configurarea la alte valori față de cele default nu se poate realiza decât cu modificare controller.yaml și reflash.

🎮 Moduri de operare
--------------------

### 🤖 Modul automat

*   **Controlat de termostate**: Termostatele fizice controlează activarea zonelor
    
*   **Întârzieri inteligente**: Centrala pornește după o întârziere configurabilă când se activează prima zonă
    
*   **Protecția zonelor**: Previne închiderea prematură a zonelor care ar cauza ciclare scurtă
    
*   **Răspuns imediat**: Zonele suplimentare se activează instant în timp ce centrala funcționează
    

### 🎯 Modul manual

*   **Control Home Assistant**: Switch-urile virtuale controlează toate zonele și centrala
    
*   **Capacitate de suprascriu**: Controlul manual suprascrie intrările termostatelor (termostatele nu mai comandă zonele, ci doar sunt afișate ca stare în HA să poată fi automatizate)
    
*   **Protecția menținută**: Toate întârzierile de siguranță și protecțiile rămân active
    
*   **Siguranță de rezervă**: Comută automat în modul Auto dacă se pierde conexiunea HA
    

📊 Entități Home Assistant
--------------------------

### Controale

*   **Mod Control** (Switch): Selecția modului Auto/Manual
    
*   **Întârziere activare** (Number): Întârzierea pornirii centralei (0-600s)
    
*   **Zonă virtuală 1-5** (Switch): Control manual zonă (dezactivat în modul Auto)
    
*   **Centrală virtuală** (Switch): Control manual centrală
    

### Monitorizare

*   **Stare sistem** (Text): Starea actuală a sistemului
    
*   **Zone active** (Text): Lista zonelor active în prezent
    
*   **Zone forțate** (Text): Zone menținute active pentru protecție
    
*   **Numărătoarea întârzierii centralei** (Sensor): Întârzierea rămasă pentru pornire
    

### Indicatori de stare

*   **Sistem încălzire** (Binary): Adevărat când centrala este activă
    
*   **Sistem în așteptare** (Binary): Adevărat în timpul întârzierii de pornire
    
*   **Orice zonă forțată** (Binary): Adevărat când zonele sunt menținute forțat active
    
*   **Controller online** (Binary): Starea conectivității ESP
    
    

🛡️ Caracteristici de siguranță
-------------------------------

### Protecția anti-ciclare scurtă

Când o zonă solicită să se închidă dar ar lăsa doar o zonă activă mai puțin decât întârzierea configurată:

*   Zona care se închide este **forțată să rămână activă**
    
*   Protecția expiră după ce zona rămasă își completează timpul minim de funcționare
    
*   Previne ciclarea rapidă pornit/oprit a centralei
    

### Operare de siguranță

*   Auto-revenire la modul Automat dacă HA se deconectează în timpul modului Manual
    
   
🔧 Parametri de configurare
---------------------------

### Ajustabili prin Home Assistant

*   **Întârziere activare**: 0-600 secunde (întârzierea pornirii centralei)
    
*   **Mod control**: Mod de operare Auto/Manual
    

🛠️ Îmbunătățiri viitoare
-------------------------

*   \[__\] Senzori de temperatură pentru monitorizarea tur/retur
    
*   \[__\] Integrare senzor temperatură exterior
    
*   \[__\] Integrare cu prognoze meteo

*   \[__\] ... vom vedea
    

📄 Licență
----------

Acest proiect este open source. Simțiți-vă liberi să modificați și distribuiți conform nevoilor dumneavoastră.

🤝 Contribuții
--------------

Contribuțiile sunt binevenite! Vă rugăm să trimiteți probleme, cereri de funcționalități sau pull request-uri.

**⚠️ Declinarea responsabilității**: Acest controller gestionează sisteme de încălzire. Asigurați-vă de instalarea electrică corespunzătoare și măsurile de siguranță. Nu sunt responsabil pentru orice daună sau rănire rezultată din instalarea sau folosirea necorespunzătoare.