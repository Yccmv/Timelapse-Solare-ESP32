# 📷 Sistema Timelapse Solare per Cantiere — ESP32-S3 XIAO Sense

> Tutorial completo per costruire un sistema timelapse autonomo, alimentato a energia solare, resistente agli agenti atmosferici, basato su ESP32-S3 Seeed Studio XIAO Sense.  
> Pensato per cantieri o installazioni outdoor di lunga durata (mesi/anni).

---

## Indice

1. [Introduzione e obiettivi](#1-introduzione-e-obiettivi)
2. [Lista componenti](#2-lista-componenti)
3. [Strumenti necessari](#3-strumenti-necessari)
4. [Architettura del sistema](#4-architettura-del-sistema)
5. [Schema elettrico](#5-schema-elettrico)
6. [Fase 1 — Circuito di alimentazione](#fase-1--circuito-di-alimentazione)
7. [Fase 2 — Setup dell'ambiente di sviluppo](#fase-2--setup-dellambiente-di-sviluppo)
8. [Fase 3 — Primo firmware: test di base](#fase-3--primo-firmware-test-di-base)
9. [Fase 4 — Integrazione RTC DS1302](#fase-4--integrazione-rtc-ds1302)
10. [Fase 5 — Integrazione camera OV5640](#fase-5--integrazione-camera-ov5640)
11. [Fase 6 — Scrittura su SD card](#fase-6--scrittura-su-sd-card)
12. [Fase 7 — Firmware completo con deep sleep](#fase-7--firmware-completo-con-deep-sleep)
13. [Fase 8 — Calcolo autonomia e ottimizzazione energetica](#fase-8--calcolo-autonomia-e-ottimizzazione-energetica)
14. [Fase 9 — Assemblaggio nel box IP68](#fase-9--assemblaggio-nel-box-ip68)
15. [Fase 10 — Manutenzione e recupero immagini](#fase-10--manutenzione-e-recupero-immagini)
16. [Troubleshooting](#troubleshooting)
17. [FAQ](#faq)
18. [Licenza](#licenza)

---

## 1. Introduzione e obiettivi

Questo progetto nasce dall'esigenza di documentare un cantiere edile nel corso di mesi, scattando fotografie a intervalli regolari in modo completamente automatico e senza accesso alla rete elettrica.

### Cosa fa il sistema

- Scatta una fotografia a intervalli configurabili (es. ogni 30 minuti, ogni ora)
- Si alimenta esclusivamente tramite pannello solare con batteria tampone
- Salva le immagini su scheda SD con nome file basato su data e ora precisi
- Va in modalità **deep sleep** tra uno scatto e l'altro per ridurre i consumi al minimo
- È contenuto in una scatola stagna **IP68** resistente a pioggia, polvere e umidità
- Funziona in autonomia per mesi senza intervento umano

### A chi è rivolto

Questo tutorial è scritto per chi ha una minima familiarità con l'elettronica hobbyistica (sa usare un multimetro, ha già saldato qualche pin) ma non è necessariamente uno sviluppatore software. Ogni passaggio è spiegato nel dettaglio, con le motivazioni dietro ogni scelta.

### Durata stimata del progetto

| Fase | Tempo stimato |
|------|--------------|
| Setup alimentazione e test | 2–3 ore |
| Ambiente di sviluppo | 1–2 ore |
| Firmware base + RTC | 3–4 ore |
| Integrazione camera + SD | 4–6 ore |
| Firmware completo | 2–3 ore |
| Assemblaggio box | 2–3 ore |
| **Totale** | **~15–20 ore** |

---

## 2. Lista componenti

### Componenti principali

| # | Componente | Modello / Specifiche | Quantità | Note |
|---|-----------|----------------------|----------|------|
| 1 | Pannello solare | Monocristallino 10W | 1 | Tensione uscita ~18V a pieno sole |
| 2 | Batteria | Anfel 12V 7.2Ah piombo-acido | 1 | Batteria VRLA sigillata |
| 3 | Regolatore di carica | PWM o MPPT (vedi note) | 1 | Compatibile 12V, almeno 5A |
| 4 | Convertitore DC-DC | Bauer Electronics SPW-1224V0503C1 | 1 | Ingresso 12V → uscita 5V/3A |
| 5 | Microcontrollore | Seeed Studio XIAO ESP32-S3 Sense | 1 | Include camera OV2640 integrata |
| 6 | Camera esterna | OV5640 5MP | 1 | Opzionale se si vuole più risoluzione |
| 7 | RTC | DS1302 con CR2032 | 1 | Orologio a basso consumo, batteria coin cell |
| 8 | SD card | SanDisk High Endurance 128GB | 1 | Classe A1 o superiore |
| 9 | Filtro UV | 40.5mm multistrato ultrasottile | 1 | Per proteggere l'obiettivo |
| 10 | Box | Gewiss IP68 | 1 | Scegli dimensioni adeguate |
| 11 | Disseccante | Gel di silice | 1 | Buste da 5g rinnovabili |

### Componenti accessori

| # | Componente | Utilizzo |
|---|-----------|---------|
| 12 | Kit resistenze 1/4W 5% | Divisori di tensione, pull-up, pull-down |
| 13 | Cavo HDMI / mini-HDMI | Collegamento temporaneo a monitor per debug |
| 14 | Cavi Dupont M-F e M-M | Connessioni su breadboard durante sviluppo |
| 15 | Breadboard | Prototipazione senza saldature |
| 16 | Morsettiere a vite | Connessioni stabili nel box finale |
| 17 | Passacavi PG7 | Ingresso cavi nel box IP68 |
| 18 | Stagno e saldatore | Per connessioni permanenti |
| 19 | Termorestringenti 2mm/4mm | Isolamento saldature |
| 20 | Nastro isolante | Protezione aggiuntiva |

### Strumenti di misura consigliati

| Strumento | Utilizzo | Indispensabile? |
|-----------|---------|-----------------|
| Multimetro digitale | Misura tensioni e continuità | ✅ Sì |
| Pinza amperometrica | Misura consumi reali | Consigliato |
| Termometro a infrarossi | Verifica surriscaldamenti | Opzionale |

---

## 3. Strumenti necessari

### Software (tutti gratuiti)

- **Arduino IDE 2.x** — ambiente di sviluppo per il firmware
  - Download: https://www.arduino.cc/en/software
- **Driver CH340/CP2102** — per la comunicazione USB-seriale con l'ESP32
  - Inclusi in Arduino IDE 2.x oppure: https://www.wch-ic.com/downloads/CH341SER_EXE.html
- **CoolTerm** o **PuTTY** — per il monitor seriale (alternativa al monitor integrato in Arduino IDE)
- **ExifTool** (opzionale) — per verificare i metadati EXIF delle immagini salvate

### Hardware

- Computer con porta USB-A o USB-C (per programmare l'ESP32)
- Cavo USB-C (per il XIAO ESP32-S3)
- Multimetro digitale (fondamentale)
- Saldatore a temperatura regolabile (consigliato 350°C per connessioni su PCB)
- Stagno senza piombo 0.8mm
- Spelafili / tronchese
- Cacciavite a croce piccolo

---

## 4. Architettura del sistema

### Diagramma a blocchi

```
[Pannello Solare 10W]
        │
        ▼
[Regolatore di Carica]
        │
        ▼
[Batteria 12V 7.2Ah] ────────────────────────────────────────────────
        │                                                             │
        ▼                                                             │
[DC-DC 12V→5V/3A]                                           (carica di mantenimento)
        │
        ▼ 5V
[ESP32-S3 XIAO Sense]
   ├── [RTC DS1302]  ←── SPI (Clock, Data, CE)
   ├── [Camera OV5640 / OV2640 integrata]  ←── DVP/CSI
   └── [SD Card 128GB]  ←── SPI (MOSI, MISO, CLK, CS)
```

### Flusso di esecuzione del firmware

```
[Accensione / Wake-up da deep sleep]
        │
        ▼
[Leggi orario da RTC DS1302]
        │
        ▼
[È nell'intervallo di scatto configurato?]
   No ──► [Calcola prossimo wake-up] ──► [Deep sleep]
   Sì
        │
        ▼
[Inizializza camera]
        │
        ▼
[Scatta fotografia]
        │
        ▼
[Salva JPEG su SD con nome YYYYMMDD_HHMMSS.jpg]
        │
        ▼
[Log su file testo (opzionale)]
        │
        ▼
[Calcola prossimo wake-up]
        │
        ▼
[Deep sleep per N minuti]
```

### Consumi energetici stimati

| Stato | Consumo |
|-------|---------|
| Deep sleep ESP32-S3 | ~14 µA |
| Attivo senza WiFi | ~80–120 mA |
| Scatto + scrittura SD | ~150–200 mA (picco) |
| Durata ciclo attivo | ~3–5 secondi |

Con un intervallo di 30 minuti, il sistema è attivo ~5 secondi ogni 1800 secondi: efficienza energetica >99%.

---

## 5. Schema elettrico

### Connessioni principali

#### DC-DC SPW-1224V0503C1 → ESP32-S3 XIAO

```
DC-DC OUT (+5V) ──── XIAO pin 5V (o VIN)
DC-DC OUT (GND) ──── XIAO pin GND
```

> ⚠️ **Importante**: Il convertitore SPW-1224V0503C1 ha una tensione di uscita fissa 5V/3A. Verifica sempre la polarità con il multimetro prima di collegare l'ESP32.

#### RTC DS1302 → ESP32-S3 XIAO

| DS1302 | XIAO ESP32-S3 | Note |
|--------|---------------|------|
| VCC    | 3.3V (pin 11) | Alimentazione 3.3V |
| GND    | GND           | Massa comune |
| CLK    | GPIO 7 (D5)   | Clock SPI |
| DAT    | GPIO 6 (D4)   | Data SPI (bidirezionale) |
| RST/CE | GPIO 5 (D3)   | Chip Enable |

> 💡 Il DS1302 usa un protocollo SPI "non standard" (bit-banging). Non usare i pin hardware SPI del XIAO per questo modulo — il firmware usa GPIO a controllo manuale.

#### SD Card (lettore microSD SPI) → ESP32-S3 XIAO

| SD Card | XIAO ESP32-S3 | Note |
|---------|---------------|------|
| VCC     | 3.3V          | Alimentazione 3.3V |
| GND     | GND           | Massa comune |
| MOSI    | GPIO 9 (D7)   | Master Out Slave In |
| MISO    | GPIO 8 (D6)   | Master In Slave Out |
| SCK     | GPIO 7 (D5)   | Clock (condiviso con RTC?) ⚠️ |
| CS      | GPIO 4 (D2)   | Chip Select SD |

> ⚠️ **Attenzione conflitto pin**: Se DS1302 e SD Card condividono il pin CLK/D5, occorre ridisegnare l'assegnazione. Vedere la sezione [Risoluzione conflitti SPI](#risoluzione-conflitti-spi) più avanti.

#### Schema pin XIAO ESP32-S3 Sense (riepilogo)

```
         [USB-C]
    ┌────────────────┐
GND │ 1          14 │ 3V3
5V  │ 2          13 │ RST
D0  │ 3   XIAO   12 │ A0/D0
D1  │ 4  ESP32   11 │ A1/D1
D2  │ 5   -S3    10 │ A2/D2
D3  │ 6  Sense    9 │ SCK/D8
D4  │ 7           8 │ MISO/D9
D5  │ 8           7 │ MOSI/D10
    └────────────────┘
```

---

## Fase 1 — Circuito di alimentazione

> 🎯 **Obiettivo**: Verificare che il circuito di alimentazione fornisca una tensione stabile e corretta prima di collegare qualsiasi componente elettronico sensibile.

### 1.1 Collegamento pannello → regolatore di carica

**Procedura:**

1. Prima di collegare qualsiasi cosa, misura la tensione di uscita del pannello solare a vuoto con il multimetro in modalità DC Volt (scala 20V o superiore). Dovresti leggere tra 17V e 21V in condizioni di buona luce.

2. Collega i cavi del pannello solare ai morsetti del regolatore di carica:
   - Filo rosso (positivo pannello) → morsetto **PV+** del regolatore
   - Filo nero (negativo pannello) → morsetto **PV-** del regolatore

3. **Non collegare ancora la batteria.**

> ⚠️ Ogni regolatore di carica ha una sequenza di collegamento precisa. La regola generale è: **prima batteria, poi pannello**. Consulta sempre il manuale del tuo specifico regolatore.

### 1.2 Collegamento regolatore → batteria

1. Collega prima i cavi della batteria al regolatore:
   - Filo rosso (positivo batteria) → morsetto **BAT+**
   - Filo nero (negativo batteria) → morsetto **BAT-**

2. Ora collega i cavi del pannello (se non l'hai già fatto).

3. Il regolatore dovrebbe accendersi con i suoi LED indicatori.

4. Misura la tensione ai morsetti della batteria con il multimetro:
   - Batteria scarica: 11.5V – 12.0V
   - Batteria carica: 12.6V – 13.0V
   - Batteria in carica: 13.6V – 14.4V (a seconda del regolatore)

### 1.3 Collegamento e verifica DC-DC SPW-1224V0503C1

1. Collega l'ingresso del convertitore DC-DC ai morsetti **LOAD+** e **LOAD-** del regolatore di carica (o direttamente ai poli della batteria con un fusibile da 3A in serie sul positivo).

2. **Prima di collegare l'ESP32**, misura la tensione di uscita del convertitore con il multimetro:
   - Punta rossa → pin OUT+ del DC-DC
   - Punta nera → pin OUT- del DC-DC
   - Lettura attesa: **4.95V – 5.05V**

3. Se la lettura è corretta, il circuito di alimentazione è pronto.

> 🔴 **Se la tensione di uscita fosse superiore a 5.5V, NON collegare l'ESP32.** Il microcontrollore si danneggierebbe irreparabilmente. Verifica i collegamenti e il modello esatto del convertitore.

### 1.4 Fusibile di protezione (raccomandato)

Inserisci un portafusibile con fusibile da **2A** sul cavo positivo tra la batteria e il DC-DC. Questo protegge l'intero circuito in caso di cortocircuito.

```
[BAT+] ──── [Fusibile 2A] ──── [DC-DC IN+]
[BAT-] ──────────────────────── [DC-DC IN-]
```

---

## Fase 2 — Setup dell'ambiente di sviluppo

> 🎯 **Obiettivo**: Installare e configurare Arduino IDE con il supporto per ESP32-S3, e verificare che il computer riconosca la scheda.

### 2.1 Installazione Arduino IDE 2.x

1. Scarica Arduino IDE 2.x da https://www.arduino.cc/en/software
2. Esegui l'installer e segui le istruzioni (installazione standard)
3. Al primo avvio, Arduino IDE potrebbe chiedere di installare driver aggiuntivi — conferma con **Sì**

### 2.2 Aggiunta del supporto ESP32 (Espressif Board Package)

1. Apri Arduino IDE
2. Vai su **File → Preferences** (Windows/Linux) oppure **Arduino IDE → Preferences** (macOS)
3. Nel campo **"Additional boards manager URLs"**, incolla questo URL:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Clicca **OK**
5. Vai su **Tools → Board → Boards Manager**
6. Nella barra di ricerca, scrivi `esp32`
7. Trova **"esp32 by Espressif Systems"** e clicca **Install** (versione 2.0.14 o superiore)
8. L'installazione scarica circa 300MB — attendi il completamento

### 2.3 Selezione della scheda corretta

1. Collega il XIAO ESP32-S3 Sense al computer tramite cavo USB-C
2. Vai su **Tools → Board → esp32 → XIAO_ESP32S3**
3. Vai su **Tools → Port** e seleziona la porta COM corrispondente al XIAO:
   - Windows: `COM3`, `COM4`, ecc.
   - macOS: `/dev/cu.usbmodem...` oppure `/dev/cu.wchusbserial...`
   - Linux: `/dev/ttyUSB0` oppure `/dev/ttyACM0`

> 💡 Se la porta non appare, installa i driver CH340: https://www.wch-ic.com/downloads/CH341SER_EXE.html

### 2.4 Configurazione Tools raccomandata

| Impostazione | Valore consigliato |
|-------------|-------------------|
| Board | XIAO_ESP32S3 |
| PSRAM | OPI PSRAM |
| Flash Size | 8MB |
| Upload Speed | 921600 |
| Core Debug Level | None (in produzione) |

### 2.5 Installazione librerie necessarie

Vai su **Tools → Manage Libraries** e installa le seguenti librerie:

| Libreria | Versione | Utilizzo |
|---------|---------|---------|
| `Seeed_Arduino_SDCard` | ultima | Gestione SD card ottimizzata per XIAO |
| `Rtc by Makuna` | ultima | Gestione DS1302 |
| `ESP32 Camera` | inclusa in esp32 | Già inclusa nel pacchetto Espressif |

### 2.6 Test di connessione: il primo "Blink"

Carica questo sketch minimale per verificare che tutto funzioni:

```cpp
// Blink LED integrato XIAO ESP32-S3
// Il LED integrato è sul pin 21 (LED_BUILTIN)

void setup() {
  Serial.begin(115200);
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.println("XIAO ESP32-S3 - Sistema Timelapse");
  Serial.println("Test LED avviato...");
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("LED acceso");
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("LED spento");
  delay(1000);
}
```

**Procedura upload:**
1. Copia il codice in un nuovo sketch (File → New Sketch)
2. Clicca il pulsante **Upload** (freccia verso destra in alto a sinistra)
3. Attendi "Done uploading" nella barra inferiore
4. Apri **Tools → Serial Monitor**, imposta velocità a **115200 baud**
5. Dovresti vedere i messaggi "LED acceso" / "LED spento" e il LED fisico lampeggiare

> 🔴 Se l'upload fallisce con errore "Failed to connect", tieni premuto il pulsante **BOOT** sul XIAO mentre clicchi Upload, poi rilascialo dopo 2 secondi.

---

## Fase 3 — Primo firmware: test di base

> 🎯 **Obiettivo**: Verificare comunicazione seriale, lettura pin analogici (tensione batteria), e funzionamento del deep sleep.

### 3.1 Lettura tensione batteria tramite divisore resistivo

Per monitorare la carica della batteria (12V) con l'ADC dell'ESP32 (massimo 3.3V), utilizziamo un partitore di tensione con due resistenze.

**Calcolo del partitore:**

- Tensione massima batteria: 14.4V
- Tensione massima ADC: 3.3V
- Rapporto necessario: 3.3 / 14.4 = 0.229

Usando resistenze dal kit in dotazione:
- **R1 = 47kΩ** (tra BAT+ e punto di misura)
- **R2 = 15kΩ** (tra punto di misura e GND)

Verifica: Vout = 14.4V × (15000 / (47000 + 15000)) = 14.4 × 0.242 = **3.48V**

> ⚠️ 3.48V supera leggermente i 3.3V! Usa R2 = **12kΩ**:  
> Vout = 14.4V × (12000 / 59000) = **2.93V** ✅

**Collegamento:**
```
[BAT+] ──── [R1: 47kΩ] ──── [A0 / GPIO1 del XIAO] ──── [R2: 12kΩ] ──── [GND]
```

**Codice di test:**

```cpp
// Test lettura tensione batteria

const int PIN_BATTERIA = A0;  // GPIO1
const float R1 = 47000.0;     // 47kΩ
const float R2 = 12000.0;     // 12kΩ
const float VREF = 3.3;       // Tensione riferimento ADC
const int ADC_MAX = 4095;     // ADC 12 bit

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("=== Test Monitoraggio Batteria ===");
}

void loop() {
  // Leggi valore ADC (media di 10 campioni per ridurre il rumore)
  long somma = 0;
  for (int i = 0; i < 10; i++) {
    somma += analogRead(PIN_BATTERIA);
    delay(10);
  }
  int adcValue = somma / 10;

  // Converti in tensione reale
  float vADC = (adcValue / (float)ADC_MAX) * VREF;
  float vBatt = vADC * ((R1 + R2) / R2);

  // Stima percentuale carica (12V=0%, 12.7V=50%, 13.0V=100% circa)
  int percentuale = constrain((int)((vBatt - 11.5) / (13.0 - 11.5) * 100), 0, 100);

  Serial.print("ADC raw: ");
  Serial.print(adcValue);
  Serial.print(" | V_ADC: ");
  Serial.print(vADC, 3);
  Serial.print("V | V_Batteria: ");
  Serial.print(vBatt, 2);
  Serial.print("V | Carica: ");
  Serial.print(percentuale);
  Serial.println("%");

  delay(5000);
}
```

### 3.2 Test deep sleep con wake-up su timer

Il deep sleep è **fondamentale** per l'autonomia del sistema. In questa modalità, il consumo scende a ~14 µA, rispetto ai ~100mA in modalità attiva.

```cpp
// Test Deep Sleep con RTC timer
// L'ESP32-S3 si sveglia dopo N secondi, esegue il codice, poi torna in sleep

#define SLEEP_SECONDI 30   // Modifica qui l'intervallo di sleep

// Contatore scatti salvato in RTC memory (sopravvive al deep sleep!)
RTC_DATA_ATTR int contatore_scatti = 0;
RTC_DATA_ATTR int boot_count = 0;

void setup() {
  Serial.begin(115200);
  delay(500);

  boot_count++;
  Serial.println("=================================================");
  Serial.print("Boot numero: ");
  Serial.println(boot_count);
  Serial.print("Scatti effettuati: ");
  Serial.println(contatore_scatti);

  // Leggi il motivo del wake-up
  esp_sleep_wakeup_cause_t wakeup_reason = esp_sleep_get_wakeup_cause();
  
  switch(wakeup_reason) {
    case ESP_SLEEP_WAKEUP_TIMER:
      Serial.println("Wake-up: Timer RTC");
      break;
    case ESP_SLEEP_WAKEUP_UNDEFINED:
      Serial.println("Wake-up: Primo avvio o reset manuale");
      break;
    default:
      Serial.print("Wake-up: Motivo sconosciuto (");
      Serial.print(wakeup_reason);
      Serial.println(")");
  }

  // === QUI andrà il codice di scatto e salvataggio ===
  Serial.println("Simulazione scatto in corso...");
  delay(2000);  // Simula il tempo necessario per uno scatto reale
  contatore_scatti++;
  Serial.println("Scatto completato.");
  // ====================================================

  // Configura il timer per il prossimo wake-up
  Serial.print("Vado in deep sleep per ");
  Serial.print(SLEEP_SECONDI);
  Serial.println(" secondi...");
  Serial.flush();  // Importante: svuota il buffer seriale prima del sleep!

  esp_sleep_enable_timer_wakeup((uint64_t)SLEEP_SECONDI * 1000000ULL);
  esp_deep_sleep_start();

  // Questa riga non viene mai raggiunta
}

void loop() {
  // Non usato — l'ESP32 va in sleep in setup()
}
```

> 💡 **Perché `RTC_DATA_ATTR`?** La memoria RAM normale viene cancellata durante il deep sleep. La RTC memory (circa 8KB) persiste anche durante il sleep, permettendo di mantenere variabili tra un ciclo e l'altro.

---

## Fase 4 — Integrazione RTC DS1302

> 🎯 **Obiettivo**: Configurare il modulo RTC per avere data e ora precisi, indipendentemente dalla rete. L'RTC mantiene l'ora anche se la batteria principale si scarica, grazie alla sua batteria coin cell CR2032.

### 4.1 Collegamento hardware DS1302

Schema di collegamento dettagliato:

```
DS1302         XIAO ESP32-S3
───────         ─────────────
VCC  ──────────  3V3 (pin 11)
GND  ──────────  GND (pin 1 o 12)
CLK  ──────────  D5 / GPIO7
DAT  ──────────  D4 / GPIO6
RST  ──────────  D3 / GPIO5
```

**Resistenze pull-up**: Il bus dati (DAT) del DS1302 richiede una resistenza pull-up da **10kΩ** verso 3.3V per funzionare correttamente in modalità bit-banging.

```
3V3 ──── [10kΩ] ──── DS1302_DAT
```

### 4.2 Installazione libreria RTC

1. Arduino IDE → Tools → Manage Libraries
2. Cerca: `Rtc by Makuna`
3. Installa la versione più recente (≥ 2.3.6)

### 4.3 Impostazione ora iniziale

La prima operazione da fare è impostare l'ora corrente sul DS1302. Esegui questo sketch **una sola volta**:

```cpp
// SKETCH DA ESEGUIRE UNA SOLA VOLTA per impostare l'ora
// Dopo l'upload, sostituiscilo con il firmware definitivo

#include <ThreeWire.h>
#include <RtcDS1302.h>

// Pin del DS1302 (modifica se necessario)
#define PIN_CLK  7   // D5
#define PIN_DAT  6   // D4
#define PIN_RST  5   // D3

ThreeWire myWire(PIN_DAT, PIN_CLK, PIN_RST);
RtcDS1302<ThreeWire> Rtc(myWire);

void setup() {
  Serial.begin(115200);
  delay(1000);

  Rtc.Begin();

  // =====================================================
  // MODIFICA QUESTI VALORI con la data/ora attuale!
  // Formato: Anno(4 cifre), Mese, Giorno, Ore, Minuti, Secondi
  // Esempio: 17 Giugno 2025 alle 14:30:00
  RtcDateTime dataOraAttuale(2025, 6, 17, 14, 30, 0);
  // =====================================================

  if (!Rtc.IsDateTimeValid()) {
    Serial.println("RTC non valido, imposto data/ora...");
    Rtc.SetDateTime(dataOraAttuale);
  }

  if (Rtc.GetIsWriteProtected()) {
    Serial.println("RTC in sola lettura, rimuovo protezione...");
    Rtc.SetIsWriteProtected(false);
  }

  if (!Rtc.GetIsRunning()) {
    Serial.println("RTC fermo, avvio oscillatore...");
    Rtc.SetIsRunning(true);
  }

  Rtc.SetDateTime(dataOraAttuale);
  Serial.println("Data/ora impostata con successo!");
  Serial.println("Ora puoi caricare il firmware principale.");
}

void loop() {
  // Leggi e mostra l'ora ogni secondo per verifica
  RtcDateTime ora = Rtc.GetDateTime();
  
  char buffer[32];
  snprintf(buffer, sizeof(buffer),
    "%04d/%02d/%02d %02d:%02d:%02d",
    ora.Year(), ora.Month(), ora.Day(),
    ora.Hour(), ora.Minute(), ora.Second()
  );
  
  Serial.println(buffer);
  delay(1000);
}
```

**Verifica**: Apri il Serial Monitor e controlla che l'ora visualizzata sia corretta e avanzi normalmente.

### 4.4 Funzione di lettura ora per il firmware principale

```cpp
#include <ThreeWire.h>
#include <RtcDS1302.h>

#define PIN_CLK  7
#define PIN_DAT  6
#define PIN_RST  5

ThreeWire myWire(PIN_DAT, PIN_CLK, PIN_RST);
RtcDS1302<ThreeWire> Rtc(myWire);

// Struttura per contenere data e ora
struct DataOra {
  uint16_t anno;
  uint8_t  mese, giorno;
  uint8_t  ore, minuti, secondi;
};

// Leggi l'ora corrente dal DS1302
DataOra leggiOra() {
  RtcDateTime now = Rtc.GetDateTime();
  DataOra t;
  t.anno    = now.Year();
  t.mese    = now.Month();
  t.giorno  = now.Day();
  t.ore     = now.Hour();
  t.minuti  = now.Minute();
  t.secondi = now.Second();
  return t;
}

// Genera un nome file nel formato YYYYMMDD_HHMMSS.jpg
String generaNomeFile(DataOra t) {
  char nome[32];
  snprintf(nome, sizeof(nome),
    "/timelapse/%04d%02d%02d_%02d%02d%02d.jpg",
    t.anno, t.mese, t.giorno,
    t.ore, t.minuti, t.secondi
  );
  return String(nome);
}
```

---

## Fase 5 — Integrazione camera OV5640

> 🎯 **Obiettivo**: Configurare e testare la camera per scattare fotografie di qualità adeguata per un timelapse di cantiere.

### 5.1 Quale camera usare?

Il **XIAO ESP32-S3 Sense** include una camera **OV2640** integrata (2MP). Il modulo **OV5640** esterno offre 5MP ma richiede più configurazione.

**Confronto:**

| Caratteristica | OV2640 (integrata) | OV5640 (esterna) |
|---------------|-------------------|------------------|
| Risoluzione max | 1600×1200 (2MP) | 2592×1944 (5MP) |
| Interfaccia | CSI (integrata) | DVP / MIPI CSI |
| Difficoltà setup | Molto bassa | Alta |
| Consumo | Basso | Medio |
| Raccomandato per | Uso generico | Cantieri con alta necessità di dettaglio |

**Raccomandazione**: Per la maggior parte dei cantieri, la **OV2640 integrata** è sufficiente e molto più semplice da usare. Questo tutorial usa la camera integrata; le istruzioni per OV5640 sono nella sezione [Configurazione OV5640 esterno](#configurazione-ov5640-esterno).

### 5.2 Pinout camera integrata XIAO ESP32-S3 Sense

La camera OV2640 integrata è già cablata internamente alla scheda. Non servono collegamenti aggiuntivi. La configurazione si fa solo via software.

### 5.3 Codice di test camera

```cpp
// Test camera XIAO ESP32-S3 Sense (OV2640 integrata)
// Scatta una foto e la mostra nel serial monitor come dimensione file

#include "esp_camera.h"

// Configurazione pin camera XIAO ESP32-S3 Sense
// IMPORTANTE: questi pin sono specifici per la versione Sense
#define PWDN_GPIO_NUM   -1
#define RESET_GPIO_NUM  -1
#define XCLK_GPIO_NUM   10
#define SIOD_GPIO_NUM   40
#define SIOC_GPIO_NUM   39
#define Y9_GPIO_NUM     48
#define Y8_GPIO_NUM     11
#define Y7_GPIO_NUM     12
#define Y6_GPIO_NUM     14
#define Y5_GPIO_NUM     16
#define Y4_GPIO_NUM     18
#define Y3_GPIO_NUM     17
#define Y2_GPIO_NUM     15
#define VSYNC_GPIO_NUM  38
#define HREF_GPIO_NUM   47
#define PCLK_GPIO_NUM   13

bool inizializzaCamera() {
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer   = LEDC_TIMER_0;
  config.pin_d0       = Y2_GPIO_NUM;
  config.pin_d1       = Y3_GPIO_NUM;
  config.pin_d2       = Y4_GPIO_NUM;
  config.pin_d3       = Y5_GPIO_NUM;
  config.pin_d4       = Y6_GPIO_NUM;
  config.pin_d5       = Y7_GPIO_NUM;
  config.pin_d6       = Y8_GPIO_NUM;
  config.pin_d7       = Y9_GPIO_NUM;
  config.pin_xclk     = XCLK_GPIO_NUM;
  config.pin_pclk     = PCLK_GPIO_NUM;
  config.pin_vsync    = VSYNC_GPIO_NUM;
  config.pin_href     = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn     = PWDN_GPIO_NUM;
  config.pin_reset    = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;       // 20MHz
  config.pixel_format = PIXFORMAT_JPEG; // Formato JPEG

  // Impostazioni qualità immagine
  config.frame_size   = FRAMESIZE_UXGA;  // 1600x1200 (massima per OV2640)
  config.jpeg_quality = 10;              // 0-63, più basso = qualità migliore
  config.fb_count     = 1;              // Frame buffer

  // Se disponibile PSRAM, usa buffer più grande
  if (psramFound()) {
    config.frame_size   = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
    config.fb_count     = 2;
    Serial.println("PSRAM trovata, uso configurazione alta qualità");
  } else {
    config.frame_size   = FRAMESIZE_SVGA; // 800x600 senza PSRAM
    config.jpeg_quality = 12;
    config.fb_count     = 1;
    Serial.println("PSRAM non trovata, uso configurazione standard");
  }

  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("ERRORE: Inizializzazione camera fallita (0x%x)\n", err);
    return false;
  }

  // Ottimizzazione sensore per uso outdoor
  sensor_t *s = esp_camera_sensor_get();
  s->set_brightness(s, 0);     // Luminosità: -2 a 2
  s->set_contrast(s, 0);       // Contrasto: -2 a 2
  s->set_saturation(s, 0);     // Saturazione: -2 a 2
  s->set_whitebal(s, 1);       // Auto white balance: on
  s->set_awb_gain(s, 1);       // AWB gain: on
  s->set_wb_mode(s, 0);        // Auto WB mode
  s->set_exposure_ctrl(s, 1);  // Auto exposure: on
  s->set_aec2(s, 0);           // AEC DSP: off
  s->set_gain_ctrl(s, 1);      // Auto gain: on
  s->set_hmirror(s, 0);        // Specchio orizzontale: off
  s->set_vflip(s, 0);          // Ribaltamento verticale: off

  Serial.println("Camera inizializzata con successo!");
  return true;
}

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("=== Test Camera ===");

  if (!inizializzaCamera()) {
    Serial.println("Impossibile inizializzare la camera. Blocco.");
    while(1) delay(1000);
  }

  Serial.println("Attendo 2 secondi per stabilizzazione sensore...");
  delay(2000);

  // Scatta una foto
  Serial.println("Scatto in corso...");
  camera_fb_t *fb = esp_camera_fb_get();
  
  if (!fb) {
    Serial.println("ERRORE: Impossibile catturare immagine!");
    return;
  }

  Serial.print("Foto scattata! Dimensione: ");
  Serial.print(fb->len);
  Serial.println(" bytes");
  Serial.print("Risoluzione: ");
  Serial.print(fb->width);
  Serial.print("x");
  Serial.println(fb->height);

  // Rilascia il frame buffer
  esp_camera_fb_return(fb);
  Serial.println("Test completato con successo!");
}

void loop() {
  delay(10000);
}
```

### 5.4 Regolazione qualità immagine

Per un timelapse di cantiere all'aperto, questi sono i parametri consigliati:

| Parametro | Valore outdoor | Spiegazione |
|-----------|---------------|-------------|
| `frame_size` | `FRAMESIZE_UXGA` (1600×1200) | Massima risoluzione disponibile |
| `jpeg_quality` | `10` | Alta qualità (range 0–63, 0=max qualità) |
| `set_whitebal` | `1` (auto) | Adatta ai cambi luce durante il giorno |
| `set_exposure_ctrl` | `1` (auto) | Gestione automatica esposizione |
| `set_aec2` | `1` (on) | AEC avanzata per ambienti variabili |

> 💡 **Dimensione file JPEG stimata**: con qualità 10 a 1600×1200, ogni foto pesa circa **200–400KB**. Su SD da 128GB si possono salvare **300.000–600.000 fotografie** — più che sufficienti per anni di timelapse.

### 5.5 Configurazione OV5640 esterno

> ⚠️ Sezione avanzata — salta se usi la camera integrata.

Il modulo OV5640 esterno richiede un adattatore FPC compatibile con il connettore CSI del XIAO ESP32-S3. La connessione avviene tramite un flat cable FPC a 24 pin.

Risorse aggiuntive:
- Schema elettrico OV5640 su XIAO: https://wiki.seeedstudio.com/xiao_esp32s3_camera_usage/
- Libreria di supporto: usa sempre `esp_camera.h` con la configurazione pin OV5640 specifica

---

## Fase 6 — Scrittura su SD card

> 🎯 **Obiettivo**: Montare la SD card e salvare le foto con naming automatico basato su data/ora.

### 6.1 Collegamento lettore microSD

Il XIAO ESP32-S3 Sense include uno slot microSD integrato sulla scheda. Non servono collegamenti aggiuntivi — la SD è gestita internamente.

> ⚠️ Se usi un modulo SD esterno separato, verifica che lavori a **3.3V** e non a 5V.

### 6.2 Formattazione della SD card

Prima di usare la scheda:
1. Inserisci la SanDisk in un lettore USB collegato al computer
2. Formatta in **FAT32** (non exFAT, non NTFS)
   - Windows: tasto destro → Formatta → FAT32
   - macOS: Utility Disco → Cancella → MS-DOS FAT
   - Linux: `sudo mkfs.fat -F 32 /dev/sdX`
3. Crea una cartella `timelapse` nella root della SD

### 6.3 Codice di test SD card

```cpp
// Test scrittura SD Card su XIAO ESP32-S3 Sense

#include "FS.h"
#include "SD.h"
#include "SPI.h"

// Il XIAO ESP32-S3 Sense ha lo slot SD integrato
// Non serve specificare pin SPI — usa quelli di default della scheda

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("=== Test SD Card ===");

  // Inizializza SD card
  if (!SD.begin()) {
    Serial.println("ERRORE: Impossibile montare la SD card!");
    Serial.println("Controlla:");
    Serial.println("  - Scheda inserita correttamente");
    Serial.println("  - Formattata in FAT32");
    Serial.println("  - Contatti puliti");
    while(1) delay(1000);
  }

  // Info sulla scheda
  uint8_t cardType = SD.cardType();
  String tipoScheda;
  switch(cardType) {
    case CARD_MMC:  tipoScheda = "MMC"; break;
    case CARD_SD:   tipoScheda = "SD"; break;
    case CARD_SDHC: tipoScheda = "SDHC/SDXC"; break;
    default:        tipoScheda = "Sconosciuto";
  }
  
  Serial.print("Tipo scheda: ");
  Serial.println(tipoScheda);
  Serial.print("Dimensione totale: ");
  Serial.print(SD.cardSize() / (1024 * 1024));
  Serial.println(" MB");
  Serial.print("Spazio usato: ");
  Serial.print(SD.usedBytes() / (1024 * 1024));
  Serial.println(" MB");

  // Crea directory timelapse se non esiste
  if (!SD.exists("/timelapse")) {
    if (SD.mkdir("/timelapse")) {
      Serial.println("Directory /timelapse creata.");
    } else {
      Serial.println("ERRORE: Impossibile creare /timelapse");
    }
  } else {
    Serial.println("Directory /timelapse già esistente.");
  }

  // Scrivi un file di test
  File testFile = SD.open("/timelapse/test.txt", FILE_WRITE);
  if (!testFile) {
    Serial.println("ERRORE: Impossibile aprire file di test");
    return;
  }
  testFile.println("Test scrittura SD - Sistema Timelapse");
  testFile.println("Se leggi questo, la SD funziona correttamente!");
  testFile.close();
  Serial.println("File di test scritto con successo.");

  // Leggi il file di test per verifica
  testFile = SD.open("/timelapse/test.txt", FILE_READ);
  if (testFile) {
    Serial.println("--- Contenuto file di test ---");
    while (testFile.available()) {
      Serial.write(testFile.read());
    }
    testFile.close();
    Serial.println("--- Fine file ---");
  }
  
  Serial.println("Test SD completato con successo!");
}

void loop() {
  delay(10000);
}
```

### 6.4 Funzione salvataggio JPEG su SD

```cpp
// Salva un frame buffer JPEG su SD card
// Restituisce true se il salvataggio è riuscito

bool salvaFotoSD(camera_fb_t *fb, const char* percorso) {
  File file = SD.open(percorso, FILE_WRITE);
  if (!file) {
    Serial.print("ERRORE: Impossibile aprire file: ");
    Serial.println(percorso);
    return false;
  }

  size_t bytescritti = file.write(fb->buf, fb->len);
  file.close();

  if (bytescritti != fb->len) {
    Serial.printf("ERRORE: Scritti %d di %d bytes\n", bytescritti, fb->len);
    return false;
  }

  Serial.printf("Foto salvata: %s (%d bytes)\n", percorso, fb->len);
  return true;
}
```

### 6.5 Gestione del log di sistema

Per monitorare il funzionamento del sistema a distanza (scaricando la SD periodicamente), è utile mantenere un file log:

```cpp
void scriviLog(const char* messaggio) {
  File logFile = SD.open("/timelapse/sistema.log", FILE_APPEND);
  if (!logFile) return;
  
  // Ottieni ora corrente
  DataOra t = leggiOra();
  char riga[128];
  snprintf(riga, sizeof(riga),
    "[%04d-%02d-%02d %02d:%02d:%02d] %s",
    t.anno, t.mese, t.giorno,
    t.ore, t.minuti, t.secondi,
    messaggio
  );
  
  logFile.println(riga);
  logFile.close();
}
```

---

## Fase 7 — Firmware completo con deep sleep

> 🎯 **Obiettivo**: Integrare tutti i componenti in un firmware definitivo, produzione-ready, con gestione degli errori e log.

### 7.1 Firmware completo

Crea un nuovo sketch e salva come `timelapse_cantiere.ino`:

```cpp
/**
 * ================================================================
 * SISTEMA TIMELAPSE SOLARE PER CANTIERE
 * Hardware: Seeed Studio XIAO ESP32-S3 Sense
 * Camera: OV2640 integrata
 * RTC: DS1302
 * Storage: SD Card (slot integrato)
 * Alimentazione: Solare 10W + Batteria 12V/7.2Ah + DC-DC 5V
 * ================================================================
 * 
 * Configurazione:
 *   - INTERVALLO_MINUTI: minuti tra uno scatto e l'altro
 *   - ORA_INIZIO / ORA_FINE: fascia oraria degli scatti
 *   - JPEG_QUALITY: qualità JPEG (10=alta, 30=media)
 * 
 * Struttura cartelle SD:
 *   /timelapse/YYYY/MM/DD/YYYYMMDD_HHMMSS.jpg
 *   /timelapse/sistema.log
 */

#include "esp_camera.h"
#include "FS.h"
#include "SD.h"
#include "SPI.h"
#include <ThreeWire.h>
#include <RtcDS1302.h>

// ================================================================
// CONFIGURAZIONE UTENTE — modifica questi valori
// ================================================================
#define INTERVALLO_MINUTI   30    // Scatta ogni 30 minuti
#define ORA_INIZIO          7     // Prima ora di scatto (7:00)
#define ORA_FINE            19    // Ultima ora di scatto (19:00)
#define JPEG_QUALITY        10    // Qualità JPEG: 0=max, 63=min
#define BATT_MINIMA_V       11.8  // Sotto questa tensione, salta scatto
#define PIN_BATT_ADC        1     // GPIO per lettura tensione batteria
// ================================================================

// Pin RTC DS1302
#define PIN_CLK  7
#define PIN_DAT  6
#define PIN_RST  5

// Pin camera OV2640 (XIAO ESP32-S3 Sense — NON modificare)
#define PWDN_GPIO_NUM   -1
#define RESET_GPIO_NUM  -1
#define XCLK_GPIO_NUM   10
#define SIOD_GPIO_NUM   40
#define SIOC_GPIO_NUM   39
#define Y9_GPIO_NUM     48
#define Y8_GPIO_NUM     11
#define Y7_GPIO_NUM     12
#define Y6_GPIO_NUM     14
#define Y5_GPIO_NUM     16
#define Y4_GPIO_NUM     18
#define Y3_GPIO_NUM     17
#define Y2_GPIO_NUM     15
#define VSYNC_GPIO_NUM  38
#define HREF_GPIO_NUM   47
#define PCLK_GPIO_NUM   13

// Variabili in RTC memory (persistono tra sleep)
RTC_DATA_ATTR int   totale_scatti    = 0;
RTC_DATA_ATTR int   totale_errori    = 0;
RTC_DATA_ATTR float ultima_tensione  = 0.0;

// Oggetti globali
ThreeWire myWire(PIN_DAT, PIN_CLK, PIN_RST);
RtcDS1302<ThreeWire> Rtc(myWire);

// ================================================================
// STRUTTURE DATI
// ================================================================

struct DataOra {
  uint16_t anno;
  uint8_t  mese, giorno;
  uint8_t  ore, minuti, secondi;
  bool     valida;
};

// ================================================================
// FUNZIONI DI SUPPORTO
// ================================================================

DataOra leggiOra() {
  DataOra t;
  if (!Rtc.IsDateTimeValid()) {
    Serial.println("[RTC] ATTENZIONE: Data/ora non valida!");
    t.valida = false;
    t.anno = 2025; t.mese = 1; t.giorno = 1;
    t.ore = 12;    t.minuti = 0; t.secondi = 0;
    return t;
  }
  RtcDateTime now = Rtc.GetDateTime();
  t.anno    = now.Year();
  t.mese    = now.Month();
  t.giorno  = now.Day();
  t.ore     = now.Hour();
  t.minuti  = now.Minute();
  t.secondi = now.Second();
  t.valida  = true;
  return t;
}

String generaNomeFile(DataOra t) {
  char nome[64];
  snprintf(nome, sizeof(nome),
    "/timelapse/%04d/%02d/%02d/%04d%02d%02d_%02d%02d%02d.jpg",
    t.anno, t.mese, t.giorno,
    t.anno, t.mese, t.giorno, t.ore, t.minuti, t.secondi
  );
  return String(nome);
}

void creaPercorso(const char* percorso) {
  char temp[64];
  strncpy(temp, percorso, sizeof(temp));
  for (int i = 1; temp[i]; i++) {
    if (temp[i] == '/') {
      temp[i] = '\0';
      if (!SD.exists(temp)) SD.mkdir(temp);
      temp[i] = '/';
    }
  }
}

void scriviLog(const char* livello, const char* messaggio) {
  File logFile = SD.open("/timelapse/sistema.log", FILE_APPEND);
  if (!logFile) return;
  DataOra t = leggiOra();
  char riga[192];
  snprintf(riga, sizeof(riga),
    "[%04d-%02d-%02d %02d:%02d:%02d] [%s] %s",
    t.anno, t.mese, t.giorno, t.ore, t.minuti, t.secondi,
    livello, messaggio
  );
  logFile.println(riga);
  logFile.close();
  Serial.println(riga);
}

float leggiTensioneBatteria() {
  const float R1 = 47000.0;
  const float R2 = 12000.0;
  long somma = 0;
  for (int i = 0; i < 20; i++) {
    somma += analogRead(PIN_BATT_ADC);
    delay(5);
  }
  float vADC = ((float)(somma / 20) / 4095.0) * 3.3;
  return vADC * ((R1 + R2) / R2);
}

// ================================================================
// INIZIALIZZAZIONE CAMERA
// ================================================================

bool inizializzaCamera() {
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer   = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM; config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM; config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM; config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM; config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk     = XCLK_GPIO_NUM;
  config.pin_pclk     = PCLK_GPIO_NUM;
  config.pin_vsync    = VSYNC_GPIO_NUM;
  config.pin_href     = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn     = PWDN_GPIO_NUM;
  config.pin_reset    = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  if (psramFound()) {
    config.frame_size   = FRAMESIZE_UXGA;
    config.jpeg_quality = JPEG_QUALITY;
    config.fb_count     = 2;
  } else {
    config.frame_size   = FRAMESIZE_SVGA;
    config.jpeg_quality = JPEG_QUALITY + 5;
    config.fb_count     = 1;
  }

  if (esp_camera_init(&config) != ESP_OK) return false;

  sensor_t *s = esp_camera_sensor_get();
  s->set_whitebal(s, 1);
  s->set_awb_gain(s, 1);
  s->set_exposure_ctrl(s, 1);
  s->set_aec2(s, 1);
  s->set_gain_ctrl(s, 1);
  return true;
}

// ================================================================
// FUNZIONE PRINCIPALE DI SCATTO
// ================================================================

bool eseguiScatto() {
  // 1. Leggi ora
  DataOra t = leggiOra();

  // 2. Verifica fascia oraria
  if (t.ore < ORA_INIZIO || t.ore >= ORA_FINE) {
    char msg[64];
    snprintf(msg, sizeof(msg), "Fuori fascia oraria (%02d:%02d). Skip.", t.ore, t.minuti);
    scriviLog("INFO", msg);
    return false;
  }

  // 3. Verifica tensione batteria
  float vBatt = leggiTensioneBatteria();
  ultima_tensione = vBatt;
  char msgBatt[64];
  snprintf(msgBatt, sizeof(msgBatt), "Tensione batteria: %.2fV", vBatt);
  scriviLog("INFO", msgBatt);

  if (vBatt < BATT_MINIMA_V) {
    scriviLog("WARN", "Batteria scarica! Scatto saltato.");
    return false;
  }

  // 4. Inizializza camera
  if (!inizializzaCamera()) {
    scriviLog("ERRORE", "Inizializzazione camera fallita!");
    return false;
  }

  // 5. Stabilizzazione sensore (importante!)
  delay(500);

  // 6. Scatta (con retry in caso di frame corrotto)
  camera_fb_t *fb = nullptr;
  for (int tentativo = 0; tentativo < 3; tentativo++) {
    fb = esp_camera_fb_get();
    if (fb && fb->len > 5000) break;  // Frame valido (>5KB)
    if (fb) esp_camera_fb_return(fb);
    fb = nullptr;
    delay(200);
  }

  if (!fb) {
    scriviLog("ERRORE", "Impossibile catturare immagine dopo 3 tentativi!");
    esp_camera_deinit();
    return false;
  }

  // 7. Genera percorso con struttura ad albero per data
  String percorso = generaNomeFile(t);
  creaPercorso(percorso.c_str());

  // 8. Salva su SD
  File file = SD.open(percorso.c_str(), FILE_WRITE);
  bool successo = false;
  if (file) {
    size_t bscritti = file.write(fb->buf, fb->len);
    file.close();
    successo = (bscritti == fb->len);
  }

  // 9. Rilascia risorse camera
  esp_camera_fb_return(fb);
  esp_camera_deinit();

  // 10. Aggiorna contatori e log
  if (successo) {
    totale_scatti++;
    char msg[128];
    snprintf(msg, sizeof(msg),
      "Scatto #%d OK: %s (%.2fV batt.)",
      totale_scatti, percorso.c_str(), vBatt
    );
    scriviLog("OK", msg);
  } else {
    totale_errori++;
    scriviLog("ERRORE", "Errore scrittura su SD!");
  }

  return successo;
}

// ================================================================
// CALCOLO PROSSIMO SLEEP
// ================================================================

uint64_t calcolaSecondiSleep() {
  DataOra t = leggiOra();
  
  // Minuti rimanenti al prossimo scatto programmato
  int minutiCorrente = t.ore * 60 + t.minuti;
  int minutiProssimo = ((minutiCorrente / INTERVALLO_MINUTI) + 1) * INTERVALLO_MINUTI;
  
  // Se il prossimo scatto è fuori dalla fascia oraria, dormi fino all'inizio
  if (minutiProssimo >= ORA_FINE * 60) {
    minutiProssimo = ORA_INIZIO * 60;
    // Aggiungi 24h se siamo già dopo l'inizio della fascia
    if (minutiCorrente >= ORA_INIZIO * 60) minutiProssimo += 24 * 60;
  }
  
  int minutiSleep = minutiProssimo - minutiCorrente;
  if (minutiSleep <= 0) minutiSleep = INTERVALLO_MINUTI;
  
  // Sottrai i secondi già trascorsi nel minuto corrente
  minutiSleep = minutiSleep * 60 - t.secondi;
  
  return (uint64_t)minutiSleep * 1000000ULL;  // Microsecondi
}

// ================================================================
// SETUP — eseguito ad ogni wake-up
// ================================================================

void setup() {
  Serial.begin(115200);
  delay(500);

  Serial.println("\n====== TIMELAPSE CANTIERE ======");
  Serial.printf("Scatti totali: %d | Errori: %d\n", totale_scatti, totale_errori);

  // Inizializza RTC
  Rtc.Begin();
  if (!Rtc.IsDateTimeValid()) {
    Serial.println("[RTC] Attenzione: ora non impostata correttamente!");
  }

  // Inizializza SD
  if (!SD.begin()) {
    Serial.println("[SD] ERRORE: SD non trovata!");
    // Vai comunque in sleep per riprovare al prossimo ciclo
  } else {
    // Esegui lo scatto
    eseguiScatto();
  }

  // Calcola durata del prossimo sleep
  uint64_t usSleep = calcolaSecondiSleep();
  Serial.printf("Deep sleep per %.1f minuti...\n", usSleep / 60000000.0);
  Serial.flush();

  // Configura wake-up e vai in sleep
  esp_sleep_enable_timer_wakeup(usSleep);
  esp_deep_sleep_start();
}

void loop() {
  // Non usato
}
```

### 7.2 Risoluzione conflitti SPI

Se DS1302 e SD card condividono pin, usa questa mappatura alternativa:

| Dispositivo | Funzione | GPIO alternativo |
|------------|---------|-----------------|
| DS1302 CLK | Clock   | GPIO 3 (D1) |
| DS1302 DAT | Data    | GPIO 4 (D2) |
| DS1302 RST | Enable  | GPIO 2 (D0) |
| SD MOSI    | Dati TX | GPIO 9 (D7) |
| SD MISO    | Dati RX | GPIO 8 (D6) |
| SD SCK     | Clock   | GPIO 7 (D5) |
| SD CS      | Select  | GPIO 44 (D10) |

---

## Fase 8 — Calcolo autonomia e ottimizzazione energetica

### 8.1 Budget energetico giornaliero

**Produzione solare stimata:**

| Condizione | Ore equivalenti pieno sole | Energia prodotta (10W) |
|-----------|---------------------------|----------------------|
| Estate, cielo sereno | 5–6 ore | **50–60 Wh** |
| Primavera/autunno, variabile | 3–4 ore | **30–40 Wh** |
| Inverno, nuvoloso | 1–2 ore | **10–20 Wh** |

**Consumo giornaliero del sistema:**

Con intervallo di 30 minuti (48 scatti/giorno), fascia 7–19 (12 ore):
- Ciclo attivo: 5 secondi × 180mA = **0.25 mAh per scatto**
- 48 scatti: **12 mAh**
- Deep sleep: 23h 56min × 0.014mA = **~0.33 mAh**
- **Consumo totale giornaliero: ~12.5 mAh a 5V = ~62.5 mWh = 0.063 Wh**

> Il sistema consuma circa **0.06 Wh/giorno** — enormemente meno di ciò che produce anche in condizioni nuvolose. Il pannello da 10W è molto sovradimensionato (intenzionalmente, per gestire giorni di pioggia consecutivi).

### 8.2 Autonomia con sola batteria (senza sole)

- Capacità batteria: 12V × 7.2Ah = **86.4 Wh**
- Efficienza scarica: 80% (non scendere mai sotto il 20% per le piombo-acido)
- Energia disponibile: 86.4 × 0.8 = **69 Wh**
- Autonomia: 69 Wh / 0.063 Wh/giorno ≈ **~1000 giorni** teorici

In pratica, con meteo pessimo e perdite varie, l'autonomia senza sole è di **30–60 giorni realistici**.

### 8.3 Ottimizzazione firmware per consumo minimo

```cpp
// Disabilita WiFi e Bluetooth (non usati nel firmware base)
// Inserisci prima di esp_deep_sleep_start()
WiFi.mode(WIFI_OFF);
btStop();

// Abbassa frequenza CPU durante le operazioni SD (non critico)
// setCpuFrequencyMhz(80);  // Default 240MHz → risparmio ~30%

// Disabilita LED integrato durante il funzionamento normale
pinMode(LED_BUILTIN, OUTPUT);
digitalWrite(LED_BUILTIN, LOW);
```

---

## Fase 9 — Assemblaggio nel box IP68

> 🎯 **Obiettivo**: Installare tutti i componenti nel box stagno in modo ordinato, sicuro e manutenibile.

### 9.1 Layout interno suggerito

```
┌─────────────────────────────────────┐
│  [Lente camera + filtro UV]         │
│           (parete frontale)         │
├─────────────────────────────────────┤
│  [ESP32-S3 XIAO]  [DS1302 RTC]     │
│  [SD card inserita]                 │
│                                     │
│  [DC-DC 5V]      [Morsettiere]     │
│                                     │
│  [Busta disseccante]                │
└─────────────────────────────────────┘
```

### 9.2 Fori e passacavi

1. **Foro obiettivo**: fora la parete frontale del box con una punta da 40.5mm (diametro del filtro UV). Usa un trapano a colonna per un foro preciso. Sigilla con silicone trasparente bicomponente intorno al bordo del filtro.

2. **Passacavo cavo pannello solare**: usa un pressacavo PG7 (adatto per cavi Ø 3–6.5mm). Inseriscilo sul lato inferiore del box per evitare infiltrazioni d'acqua.

3. **Nessun altro foro**: tutti gli altri componenti sono interni.

### 9.3 Procedure di sigillatura

1. Prima di installare tutto, verifica la tenuta del box vuoto immerso in acqua per 30 secondi
2. Pulisci la guarnizione con alcool isopropilico prima di chiudere
3. Non forzare mai la chiusura — il box IP68 si chiude con le viti di chiusura, non a pressione
4. Applica una piccola quantità di silicone sul pressacavo del cavo solare
5. **Mai usare silicone acetico** (odore di aceto) su componenti elettronici — usa silicone neutro

### 9.4 Posizionamento del disseccante

- Inserisci 1–2 buste da 5g di gel di silice nel box
- Posizionale lontano dalla lente della camera (possono creare riflessi)
- Sostituiscile ogni 6 mesi o quando cambiano colore (da blu a rosa per quelle indicatrici)
- Per ricaricarle: forno a 120°C per 2 ore

### 9.5 Orientamento e installazione in campo

**Inclinazione della lente**: punta la camera verso il punto di massimo interesse del cantiere (es. la gru, il fronte di avanzamento). Considera che nel corso dei mesi la luce solare cambierà — evita orientamenti a ovest se il sole pomeridiano potrebbe creare flare costanti.

**Inclinazione pannello solare**:
- In Italia (latitudine media 43°N): inclina il pannello a **43–47°** rispetto all'orizzontale
- Orientamento: rivolto a **SUD** (azimut 180°)
- Distanza dal box: almeno 50cm per non ombreggiarlo

**Fissaggio**: usa staffe regolabili in acciaio inox (non alluminio — si corrode con umidità e cemento). Prevedi almeno 4 punti di fissaggio per il box.

### 9.6 Prima accensione in campo

Checklist prima di sigillare e installare definitivamente:

- [ ] Firmware caricato e testato al banco
- [ ] RTC impostato con ora corretta
- [ ] SD card formattata e cartella /timelapse creata
- [ ] Tensione DC-DC verificata a 5.0V
- [ ] Test scatto completo eseguito (foto salvata su SD)
- [ ] Log sistema verificato
- [ ] Batteria carica (>12.5V)
- [ ] Disseccante inserito
- [ ] Filtro UV montato

---

## Fase 10 — Manutenzione e recupero immagini

### 10.1 Recupero immagini

**Opzione A — Rimozione SD card** (più semplice):
1. Apri il box (porta un cacciavite)
2. Estrai la microSD
3. Inseriscila nel computer tramite adattatore
4. Copia la cartella `/timelapse/` sul computer
5. Reinserisci la SD e richiudi

**Opzione B — WiFi via Access Point** (più comoda, richiede aggiornamento firmware):
Aggiunta futura: l'ESP32-S3 può creare un Access Point WiFi su richiesta (doppia pressione del pulsante BOOT → attiva AP per 5 minuti → server web per scaricare le immagini).

### 10.2 Creazione del video timelapse

Una volta raccolta una serie di immagini, usa **ffmpeg** per creare il video:

```bash
# Installa ffmpeg (se non presente)
# Windows: https://ffmpeg.org/download.html
# macOS: brew install ffmpeg
# Linux: sudo apt install ffmpeg

# Crea timelapse a 25fps con tutte le immagini in ordine
ffmpeg -framerate 25 -pattern_type glob -i "*.jpg" \
  -vf "scale=1600:1200:flags=lanczos" \
  -c:v libx264 -crf 18 -preset slow \
  -pix_fmt yuv420p \
  timelapse_cantiere.mp4

# Con data sovrapposta (usa ffmpeg con filtro drawtext)
ffmpeg -framerate 25 -pattern_type glob -i "*.jpg" \
  -vf "scale=1600:1200,drawtext=text='%{metadata\:date\:N/D}':fontsize=24:x=10:y=10:fontcolor=white:shadowcolor=black:shadowx=2:shadowy=2" \
  -c:v libx264 -crf 18 \
  timelapse_con_data.mp4
```

### 10.3 Controllo periodico consigliato

| Frequenza | Attività |
|-----------|---------|
| Settimanale | Verifica da remoto (se WiFi disponibile) o visiva del LED di stato |
| Mensile | Controllo fisico del box, verifica tenuta, pulizia lente |
| Ogni 6 mesi | Sostituzione disseccante, download immagini, controllo batteria |
| Annuale | Verifica stato batteria (carica e test di capacità), pulizia pannello |

---

## Troubleshooting

### La camera non si inizializza

**Sintomi**: Serial monitor mostra "Inizializzazione camera fallita (0x...)"

**Cause e soluzioni**:
1. **Pin errati**: Verifica di aver selezionato correttamente "XIAO_ESP32S3" (non una scheda generica ESP32-S3). I pin camera sono specifici per la versione Sense.
2. **Alimentazione insufficiente**: La camera richiede picchi di corrente. Usa il pin 5V del XIAO, non 3.3V.
3. **Modulo camera non inserito**: Il XIAO Sense ha il modulo camera su un connettore FPC separato — verifica che sia agganciato correttamente.
4. **PSRAM non abilitata**: In Tools → PSRAM → seleziona "OPI PSRAM".

### La SD card non viene riconosciuta

**Sintomi**: "SD card mount failed"

**Cause e soluzioni**:
1. Formattazione errata → riformatta in FAT32 (non exFAT)
2. Scheda non inserita fino in fondo → premi fino a sentire il click
3. Scheda difettosa → testa la SD su computer con H2testw (Windows) o F3 (Linux/Mac)
4. Velocità SPI troppo alta → nella libreria SD, prova `SD.begin(4000000)` (4MHz invece del default)

### Il DS1302 mostra ora sbagliata

**Sintomi**: Data/ora sempre a 01/01/2000 o scorrono troppo velocemente/lentamente

**Cause e soluzioni**:
1. **Batteria CR2032 scarica**: sostituisci la batteria sul modulo DS1302
2. **Protezione scrittura attiva**: esegui nuovamente lo sketch di impostazione ora
3. **Resistenza pull-up mancante**: aggiungi 10kΩ tra DAT e 3.3V
4. **Oscillatore fermo**: chiama `Rtc.SetIsRunning(true)` nello sketch di impostazione

### L'upload via Arduino IDE fallisce

**Sintomi**: "A fatal error occurred: Failed to connect to ESP32-S3"

**Soluzioni**:
1. Tieni premuto il pulsante **BOOT** sul XIAO durante l'upload
2. Prova un cavo USB diverso (molti cavi sono solo carica, senza dati)
3. Abbassa la velocità di upload: Tools → Upload Speed → 460800
4. Reinstalla i driver CH340

### La batteria si scarica rapidamente

**Cause e soluzioni**:
1. **WiFi/BT non disabilitato**: aggiungi `WiFi.mode(WIFI_OFF); btStop();` prima del deep sleep
2. **Deep sleep non attivo**: verifica che il monitor seriale mostri "Deep sleep per X minuti"
3. **Perdita di corrente nel circuito**: misura la corrente in sleep con amperometro in serie — deve essere < 1mA
4. **Batteria degradata**: le batterie piombo-acido perdono capacità dopo 3–5 anni

### Le foto sono sfocate o sovraesposte

**Soluzioni**:
1. **Fuoco**: la OV2640 ha fuoco fisso a ~50cm–∞. Per soggetti a distanza (cantiere) non serve regolazione.
2. **Sovraesposizione**: imposta `s->set_aec2(s, 1)` per l'AEC avanzata
3. **Lente sporca**: pulisci il filtro UV con panno in microfibra
4. **Condensa interna**: aumenta il disseccante o verifica la tenuta del box

---

## FAQ

**D: Posso aggiungere il WiFi per vedere le foto da remoto?**  
R: Sì, ma considera che il WiFi consuma ~200mA durante la connessione. Con un upload periodico notturno (es. 1 volta al giorno), il consumo extra è trascurabile. Il firmware con WiFi è una possibile espansione futura di questo progetto.

**D: Quanto spazio occupano le foto su un anno intero?**  
R: Con 48 scatti/giorno × 300KB/foto × 365 giorni ≈ **5.3GB/anno**. Con la SD da 128GB hai spazio per ~24 anni di timelapse.

**D: Posso usare questo sistema con altre fotocamere?**  
R: Sì, l'ESP32-S3 supporta anche moduli con interfaccia MIPI CSI più moderna. Tuttavia, richiede modifiche significative al firmware.

**D: Il sistema funziona anche di notte?**  
R: Con la camera OV2640 standard, le foto notturne sono di scarsa qualità per mancanza di illuminazione. Puoi limitare gli scatti alla fascia diurna (come configurato) oppure aggiungere un illuminatore IR e modificare il firmware per abilitare la visione notturna.

**D: Come aggiorno il firmware senza aprire il box?**  
R: Con OTA (Over The Air) via WiFi. È una funzionalità avanzata non inclusa in questa versione base, ma facilmente aggiungibile con la libreria `ArduinoOTA` di Espressif.

**D: La batteria piombo-acido regge le temperature invernali?**  
R: Le batterie VRLA piombo-acido funzionano tra -20°C e +50°C, ma perdono capacità al freddo (a 0°C circa il 20-30% in meno). Posiziona il box in modo che riceva luce solare diretta per mantenersi più caldo.

---

## Licenza

Questo progetto è rilasciato sotto licenza **MIT**.

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

*Progetto realizzato con ESP32-S3 XIAO Sense, Arduino IDE e amore per il fai da te.*  
*Pull request e segnalazioni di bug sono benvenute.*
