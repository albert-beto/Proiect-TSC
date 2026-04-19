InkTime v6
InkTime v6 este un wearable cu consum redus, construit în jurul unui ecran e-paper și al SoC-ului Nordic nRF52840. Platforma combină Bluetooth Low Energy, monitorizare de mișcare, feedback haptic și alimentare din acumulator LiPo cu încărcare prin USB-C, totul într-o arhitectură gândită pentru autonomie mare.
Prezentare generală
Scopul proiectului este realizarea unui dispozitiv portabil care să rămână activ mult timp între încărcări, fără să sacrifice conectivitatea sau funcțiile de interacțiune. MCU-ul central coordonează comunicația radio, legătura cu perifericele pe I2C și SPI, precum și tranzițiile între stările de consum redus și cele active.
Arhitectura sistemului
Hardware
MCU principal — Nordic nRF52840 (U$1)
Nucleul sistemului este nRF52840, un SoC cu ARM Cortex-M4F, memorie Flash de 1 MB, RAM de 256 KB, radio 2.4 GHz și suport USB 2.0 Full-Speed. Acesta a fost ales pentru că poate acoperi atât partea de BLE, cât și interfețele digitale ale sistemului, fără componente radio externe suplimentare.
Avantajele principale urmărite în acest design sunt:
consum redus în modurile de repaus;
integrarea BLE 5.0 direct în SoC;
suport nativ pentru USB prin liniile D+ și D−.
Pentru tactare sunt folosite două surse de ceas:
X1 — cristal de 32 MHz pentru HFCLK;
X2 — cristal de 32.768 kHz pentru LFCLK.
Alimentare și management baterie
Subsistemul de putere este construit pentru utilizare cu acumulator LiPo și alimentare/încărcare prin USB-C.
USB-C (J1, KH-TYPE-C-16P)
Conectorul este folosit pentru alimentare și pentru legătura USB 2.0, fiind rutate doar liniile necesare proiectului.
USBLC6-2SC6Y
Această componentă oferă protecție ESD pentru D+, D− și CC. A fost poziționată înaintea restului lanțului de semnal, cât mai aproape de conector.
BQ25180 (IC4)
Încărcătorul Li-Po se ocupă de gestionarea procesului de încărcare. Curentul este configurabil prin I2C, iar circuitul oferă și semnalizare de evenimente prin pinul PMIC_INT.
Caracteristici utilizate în proiect:
configurare prin SDA/SCL;
curent de încărcare programabil;
semnal de întrerupere pentru status;
protecție termică.
MAX17048 (U1)
Fuel gauge-ul estimează nivelul bateriei pe baza tensiunii celulei și oferă informații despre stare prin I2C. Pinul ALERT poate fi folosit pentru notificări către microcontroller.
RT6160AWSC (IC3)
Convertorul DC/DC deservește partea de alimentare a sistemului și contribuie la funcționarea eficientă în regim low-power.
Periferice
BMA423 (IC2)
Accelerometrul pe 3 axe este folosit pentru detecția de mișcare și step counting. Comunicarea se face pe I2C, iar pinul CSB este legat la VDDIO pentru selectarea modului I2C.
În proiect sunt folosite:
două linii de întrerupere, INT1 și INT2;
moduri de consum redus;
funcții de activity detection specifice unui dispozitiv wearable.
DRV2605 (IC1)
Driverul haptic este controlat prin I2C. Pinul HAPTIC_EN permite activarea sau dezactivarea rapidă dintr-un GPIO al MCU-ului, iar ieșirile OUT+ și OUT− merg direct spre actuator.
Butoane (SW_UP, SW_DN, SW_ENT)
Sunt prezente trei butoane, fiecare conectat direct la GPIO-uri. Pentru stabilitate au fost folosite rezistențe de pull-up externe de 10 kΩ și filtrare RC cu 1 µF la masă.
E-paper display
Afișajul este conectat printr-un FPC de 24 pini (J3) și reprezintă elementul principal de interfață vizuală. Pentru că ecranul este bistabil, energia este consumată în principal la actualizare, nu în starea statică.
Controlul ecranului este împărțit în trei zone:
1. Interfața digitală SPI 4-wire
MOSI;
SCK;
EPD_CS;
EPD_DC;
EPD_RST;
EPD_BUSY.
2. Generarea tensiunilor de bias
Un lanț dedicat de boost, diode Schottky și condensatoare de 1 µF / 50 V produce tensiunile necesare matricei TFT: VGH, VGL, VSH și VSL.
3. Comutarea alimentării
Un load switch realizat cu DMG2305UX (Q1, P-MOSFET) este comandat prin semnalul EPD_PWR și întrerupe complet alimentarea de 3.3 V a displayului atunci când acesta nu este actualizat.
Interfața de programare — SWD
Pentru debug și programare este folosit conectorul TC2030-IDC (J2), fără headere montate permanent pe placă. Astfel se economisește spațiu, păstrând în același timp accesul la interfața SWD.
Liniile expuse sunt:
VTREF (3.3V);
SWDIO;
SWDCLK;
SWO;
nRESET;
GND.
Pe placă sunt disponibile și test pads dedicate:
TP_SWDIO;
TP_SWDCLK;
TP_SWO;
TP_RESET;
TP_3.3V;
TP_GND.
Planul pinilor pentru nRF52840
Tabelul de mai jos rezumă rolul pinilor relevanți ai MCU-ului și motivul alegerii lor în layout.
Pin nRF52840	Rol în sistem	Tip	Motivație
P0.00	Cristal 32.768 kHz	OSC	Pin dedicat pentru LFXO.
P0.01	Cristal 32.768 kHz	OSC	Pin dedicat pentru LFXO.
P0.09	Neutilizat	GPIO	Rezervat.
P0.10	Neutilizat	GPIO	Rezervat.
P0.13	MOSI către EPD	SPIM	Poziționare convenabilă față de zona conectorului displayului.
P0.14	SCK către EPD	SPIM	Lângă MOSI, pentru rutare coerentă.
P0.18	Reset extern	nRESET	Pin dedicat.
P0.19	EPD_CS	SPIM	Aproape de celelalte semnale SPI pentru display.
P0.20	EPD_DC	GPIO	Ușor de rutat împreună cu magistrala displayului.
P0.21	EPD_RST	GPIO	Grupat logic cu celelalte semnale EPD.
P0.22	SWO	SWO	Funcția standard pentru debug output.
P0.23	EPD_BUSY	GPIO input + INT	Permite detecție de evenimente prin GPIOTE.
P0.24	EPD_PWR	GPIO	Comandă gate-ul load switch-ului pentru ecran.
P0.25	HAPTIC_EN	GPIO	Activează sau dezactivează driverul haptic.
P0.26	SDA	TWIM	Parte din magistrala I2C comună.
P0.27	SCL	TWIM	Vecin cu SDA pentru rutare mai simplă.
P0.28	Neutilizat	analog	Rezervat.
P0.29	Neutilizat	GPIO	Rezervat.
P0.30	Neutilizat	GPIO	Rezervat.
P0.31	Neutilizat	GPIO	Rezervat.
P1.00	PMIC_INT	GPIO input + INT	Wake-up la evenimente de charging.
P1.01	SW_UP	GPIO input	Intrare pentru buton, cu wake-on-pin.
P1.02	SW_DN	GPIO input	Intrare pentru buton, cu wake-on-pin.
P1.03	SW_ENT	GPIO input	Intrare pentru buton, cu wake-on-pin.
P1.04	IMU_INT1	GPIO input + INT	Linie de eveniment din accelerometru.
P1.05	IMU_INT2	GPIO input + INT	A doua linie de eveniment din accelerometru.
P1.06	ALERT	GPIO input + INT	Alertă de la fuel gauge.
P1.07	EPD (spre Q3)	GPIO / PWM	Util pentru comutare controlată prin PWM/GPIO.
SWDCLK	Programare / debug	Dedicat	Linia SWD de clock.
SWDIO	Programare / debug	Dedicat	Linia SWD de date.
VBUS	Detectare USB	USB PHY	Folosit pentru sensing la conectarea USB.
D+ / D−	USB 2.0 Full-Speed	USB PHY	Pini dedicați interfeței USB.
Buget de consum în repaus
Estimarea de mai jos descrie consumul tipic în standby pentru fiecare bloc relevant:
Componentă	Curent tipic în sleep	Observații
nRF52840 (System ON, RAM retention, RTC, LFXO)	~1.5 µA	Fără radio activ
BMA423 (low-power, 25 Hz)	~4 µA	Step detection activ
BQ25180 (quiescent, fără charging)	~6 µA	
MAX17048 (active measurement)	~23 µA	Hibernate: ~3 µA
DRV2605 (EN=LOW, shutdown)	< 1 µA	Oprit prin HAPTIC_EN
RT6160 (PFM mode, fără sarcină)	~25 µA	Mod ultra-low-quiescent
E-paper (fără refresh)	~0 µA	Imaginea rămâne afișată fără alimentare
Leakage PCB + diode	~5 µA	Estimare
Total	≈ 65 µA	
Estimare autonomie
Pentru un acumulator de 150 mAh, a fost considerat următorul scenariu de utilizare:
23 h 45 min standby la 65 µA, ceea ce înseamnă aproximativ 1.54 mAh/zi;
refresh-uri zilnice ale ecranului: aproximativ 0.2 mAh/zi;
impulsuri haptice zilnice: aproximativ 0.01 mAh/zi.
Rezultă un consum mediu de circa 1.75 mAh/zi, ceea ce duce la o autonomie estimată de aproximativ 85 de zile pentru o încărcare completă.
Observații de proiectare
Pentru rezistoare și condensatoare foarte mici au fost păstrate modele 3D placeholder, deoarece geometria lor este suficient de apropiată de componentele reale pentru scopul prezentării.
Au fost acceptate 6 erori asociate modelelor 3D ale butoanelor, existente încă din fazele inițiale ale proiectului.
Au fost aprobate și erori de tip Overlap respectiv Copper clearance rezultate din utilizarea tehnicii via-in-pad.
Rezumat
InkTime v6 este un design orientat spre autonomie ridicată și integrare compactă, cu accent pe afișaj bistabil, BLE și management energetic atent. Combinația dintre nRF52840, e-paper, senzori și circuitul de alimentare îl face potrivit pentru un smartwatch sau alt wearable cu funcționare îndelungată între încărcări.
