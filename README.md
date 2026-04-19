# InkTime v6

**InkTime v6** este un wearable low-power cu ecran e-paper, construit în jurul SoC-ului **Nordic nRF52840**. Sistemul integrează Bluetooth Low Energy, accelerometru pentru detecție de mișcare și step counting, feedback haptic și încărcare prin USB-C dintr-un acumulator LiPo.

Platforma este gândită pentru consum redus și autonomie mare, iar microcontrollerul coordonează atât conectivitatea radio, cât și interfețele I2C/SPI către periferice și managementul regimurilor de putere.

---

## 1. Diagramă bloc

![Diagrama bloc InkTime](Images/Diagram.png)

---

## 2. Descriere hardware

### 2.1 Microcontroller — Nordic nRF52840 (U$1)

nRF52840 este SoC-ul principal al sistemului. Include un nucleu ARM Cortex-M4F cu FPU, 1 MB Flash, 256 KB RAM, radio 2.4 GHz multiprotocol și interfață USB 2.0 Full-Speed integrată. [web:226]

Acest circuit a fost ales pentru câteva avantaje importante:

- consum redus în modurile **System OFF** și **System ON cu RAM retention**;
- BLE 5.0 integrat, fără necesitatea unui transceiver radio separat;
- suport USB 2.0 FS direct pe liniile `D+` și `D−`. [web:226]

Sunt utilizate două surse externe de ceas:

- `X1` — cristal de 32 MHz pentru **HFCLK**;
- `X2` — cristal de 32.768 kHz pentru **LFCLK**.

### 2.2 Subsistemul de alimentare

#### USB-C (J1, KH-TYPE-C-16P)

Conectorul USB Type-C este utilizat pentru alimentare și pentru legătura USB 2.0. În proiect sunt rutate doar liniile necesare.

#### USBLC6-2SC6Y

Circuit de protecție ESD pentru `D+`, `D−` și `CC`. Este plasat cât mai aproape de conector, înaintea celorlalte componente din lanțul de semnal.

#### BQ25180 (IC4)

BQ25180 este încărcătorul Li-Po al sistemului, controlat prin I2C. Dispozitivul permite configurarea curentului de încărcare și expune un semnal de întrerupere pentru raportarea evenimentelor. [web:231]

Caracteristici folosite în proiect:

- curent de încărcare programabil prin I2C;
- interfață `SDA/SCL` pentru configurare și status;
- pin `PMIC_INT` pentru semnalarea evenimentelor;
- protecție termică. [web:231]

#### MAX17048 (U1)

MAX17048 este fuel gauge-ul sistemului. Acesta măsoară tensiunea celulei și estimează nivelul de încărcare al bateriei prin I2C, având și un pin de alertă configurabil. [web:232]

În proiect:

- este conectat pe magistrala I2C comună;
- semnalul `ALERT` este rutat către MCU;
- este folosit pentru monitorizarea nivelului bateriei. [web:232]

#### RT6160AWSC (IC3)

RT6160AWSC este convertorul DC/DC folosit în subsistemul de alimentare.

### 2.3 Periferice

#### BMA423 (IC2)

BMA423 este accelerometrul digital pe 3 axe utilizat pentru:

- detecția de mișcare;
- step counting;
- evenimente de tip activity detection.

Acesta comunică pe **I2C**, iar pinul `CSB` este legat la `VDDIO` pentru a forța modul I2C în locul modului SPI.

În plus:

- `INT1` și `INT2` sunt rutate separat către MCU;
- fiecare linie de întrerupere poate fi asociată unor evenimente diferite.

#### DRV2605 (IC1)

DRV2605 este driverul haptic al dispozitivului. Se controlează prin **I2C**, iar pinul `HAPTIC_EN` este comandat de un GPIO al microcontrollerului.

Ieșirile `OUT+` și `OUT−` merg direct către actuatorul haptic.

#### Butoane (SW_UP, SW_DN, SW_ENT)

Sunt folosite trei butoane fizice, conectate direct la GPIO-uri.

Fiecare buton are:

- pull-up extern de 10 kΩ;
- debounce RC cu condensator de 1 µF la GND.

#### E-Paper Display (J3, FPC 24-pin)

Displayul e-paper este conectat printr-un conector FPC de 24 pini. Pentru că este bistabil, consumul de energie apare în principal în timpul refresh-ului, nu și în starea stabilă.

Controlul ecranului este împărțit în trei blocuri:

**1. Interfața digitală SPI 4-wire**

- `MOSI` — date către EPD;
- `SCK` — clock;
- `EPD_CS` — chip select, activ pe low;
- `EPD_DC` — selectare date/comandă;
- `EPD_RST` — reset, activ pe low;
- `EPD_BUSY` — semnal de status către MCU.

**2. Generatorul de tensiuni bias pentru EPD**

Un convertor boost împreună cu o rețea de diode Schottky și condensatoare de 1 µF / 50 V generează tensiunile necesare matricei TFT:

- `VGH`;
- `VGL`;
- `VSH`;
- `VSL`.

Aceste tensiuni sunt utilizate de ecran în timpul actualizării.

**3. Load switch pentru alimentarea displayului**

Un MOSFET P-channel DMG2305UX (`Q1`) este comandat prin semnalul `EPD_PWR` și întrerupe complet alimentarea de 3.3 V a displayului atunci când acesta nu este actualizat.

### 2.4 Conector SWD (J2, TC2030-IDC)

Pentru programare și debug este folosit un conector **Tag-Connect TC2030-IDC** cu 6 pini, fără header montat permanent pe placă.

Linii disponibile:

- `VTREF (3.3V)`;
- `SWDIO`;
- `SWDCLK`;
- `SWO`;
- `nRESET`;
- `GND`.

Sunt disponibile și test pads dedicate:

- `TP_SWDIO`;
- `TP_SWDCLK`;
- `TP_SWO`;
- `TP_RESET`;
- `TP_3.3V`;
- `TP_GND`.

---

## 3. Asocierea pinilor nRF52840

Tabelul de mai jos descrie rolul pinilor relevanți ai nRF52840 și motivația alegerii lor în design.

| Pin nRF52840 | Funcție hardware | Tip | De ce acest pin |
|---|---|---|---|
| **P0.00** | Cristal 32.768 kHz (LFCLK) | OSC | Pin dedicat intrării LFXO. |
| **P0.01** | Cristal 32.768 kHz (LFCLK) | OSC | Pin dedicat intrării LFXO. |
| **P0.09** | neconectat | GPIO | Rezervat. |
| **P0.10** | neconectat | GPIO | Rezervat. |
| **P0.13** | `MOSI` spre EPD | SPIM | Aproape de zona conectorului displayului. |
| **P0.14** | `SCK` spre EPD | SPIM | Lângă MOSI, pentru rutare coerentă. |
| **P0.18** | Reset extern | nRESET | Pin dedicat resetului. |
| **P0.19** | `EPD_CS` | SPIM | Grupat cu restul semnalelor SPI ale displayului. |
| **P0.20** | `EPD_DC` | GPIO | Aproape de interfața EPD. |
| **P0.21** | `EPD_RST` | GPIO | Grupat logic cu semnalele de control ale ecranului. |
| **P0.22** | `SWO` | SWO | Pin standard pentru debug output. |
| **P0.23** | `EPD_BUSY` | GPIO input + INT | Poate fi folosit pentru evenimente GPIOTE. |
| **P0.24** | `EPD_PWR` | GPIO | Controlează load switch-ul pentru display. |
| **P0.25** | `HAPTIC_EN` | GPIO | Activează/dezactivează driverul haptic. |
| **P0.26** | `SDA` | TWIM | Parte din magistrala I2C comună. |
| **P0.27** | `SCL` | TWIM | Lângă SDA, pentru rutare simplă. |
| **P0.28** | neconectat | analog | Rezervat. |
| **P0.29** | neconectat | GPIO | Rezervat. |
| **P0.30** | neconectat | GPIO | Rezervat. |
| **P0.31** | neconectat | GPIO | Rezervat. |
| **P1.00** | `PMIC_INT` (BQ25180) | GPIO input + INT | Wake-on-interrupt pentru charging events. |
| **P1.01** | `SW_UP` | GPIO input | Intrare pentru buton. |
| **P1.02** | `SW_DN` | GPIO input | Intrare pentru buton. |
| **P1.03** | `SW_ENT` | GPIO input | Intrare pentru buton. |
| **P1.04** | `IMU_INT1` | GPIO input + INT | Linie de eveniment din accelerometru. |
| **P1.05** | `IMU_INT2` | GPIO input + INT | A doua linie de eveniment din accelerometru. |
| **P1.06** | `ALERT` (MAX17048) | GPIO input + INT | Alertă pentru starea bateriei. |
| **P1.07** | EPD (spre Q3) | GPIO / PWM | Poate fi folosit pentru comandă PWM. |
| **SWDCLK** | Programare / debug | Dedicat | Pin SWD dedicat. |
| **SWDIO** | Programare / debug | Dedicat | Pin SWD dedicat. |
| **VBUS** | Sensing tensiune USB | USB PHY | Detectează conectarea USB. |
| **D+ / D−** | USB 2.0 Full-Speed | USB PHY | Linii dedicate interfeței USB. |

---

## 4. Consum în standby

Estimarea de mai jos descrie curentul tipic în regim sleep pentru principalele blocuri ale sistemului.

| Componentă | Curent tipic sleep | Observații |
|---|---:|---|
| nRF52840 (System ON, RAM retention, RTC, LFXO) | ~1.5 µA | Fără radio activ |
| BMA423 (low-power, 25 Hz) | ~4 µA | Step detection activ |
| BQ25180 (quiescent, no charge) | ~6 µA | |
| MAX17048 (active measurement) | ~23 µA | Hibernate: ~3 µA |
| DRV2605 (EN=LOW, shutdown) | < 1 µA | Dezactivat prin `HAPTIC_EN` |
| RT6160 (PFM mode, no load) | ~25 µA | Ultra-low-quiescent mode |
| E-Paper (bistabil, nerefreshat) | ~0 µA | Imaginea se păstrează fără putere |
| Leakage PCB + diode | ~5 µA | Estimat |
| **Total sleep** | **≈ 65 µA** | |

### 4.1 Estimare autonomie pentru acumulator de 150 mAh

Pentru un profil de utilizare tipic s-a considerat:

- 23 h 45 min standby la 65 µA ≈ 1.54 mAh/zi;
- refresh-uri zilnice de ecran ≈ 0.2 mAh/zi;
- impulsuri haptice zilnice ≈ 0.01 mAh/zi.

Rezultă:

- consum mediu zilnic ≈ **1.75 mAh/zi**;
- autonomie estimată ≈ **85 zile** pentru un acumulator de 150 mAh.

---

## 5. Decizii de design și observații finale

- Pentru modelele componentelor pasive mici au fost păstrate placeholdere 3D, deoarece sunt suficient de apropiate de piesele reale ca formă și dimensiune.
- Au fost acceptate 6 erori asociate modelelor 3D ale butoanelor, existente din fazele anterioare ale proiectului.
- Au fost aprobate erori de tip **Overlap** și **Copper clearance** cauzate de utilizarea de **via in pad**.
