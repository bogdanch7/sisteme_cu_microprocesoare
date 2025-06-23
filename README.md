# Sistem de Interfon cu Autentificare RFID și Tastatură

Acest proiect implementează un sistem de interfon bazat pe un microcontroler Arduino, conceput pentru a permite autentificarea utilizatorilor prin multiple metode și a oferi feedback vizual și sonor. Sistemul integrează o tastatură matriceală, un cititor RFID și un senzor de mișcare PIR pentru o securitate și funcționalitate sporite. 

---

## Scopul Proiectului

Scopul principal al acestui proiect este de a crea un sistem de interfon cu capabilități de autentificare avansate, oferind o soluție de control acces care utilizează atât metode tradiționale (PIN), cât și moderne (RFID), cu monitorizare a prezenței. 

---

## Descriere Generală și Funcționalități

Montajul oferă următoarele funcționalități cheie:
- Autentificare prin RFID: Utilizatorii pot deschide sistemul folosind o cartelă de acces sau un tag RFID, al cărui UID este comparat cu o listă predefinită de carduri autorizate. 
- Tastatură de Securitate: O tastatură matriceală 4x4 permite introducerea unui cod de acces numeric. Sistemul validează codul introdus față de un cod presetat, permițând accesul doar la introducerea corectă. 
- Feedback Vizual și Auditiv: Sistemul utilizează LED-uri (verde pentru acces permis, albastru pentru acces respins) și un buzzer pentru a semnala statusul autentificării și validitatea acțiunilor. 
- Monitorizare prin Senzor PIR: Un senzor de mișcare (PIR) detectează prezența unei persoane în apropierea interfonului, putând activa sau dezactiva sistemul pentru eficiență energetică.

---

## Componente Hardware Utilizate

- Microcontroler Arduino Plusivo: Placa principală de control care gestionează toate input-urile de la tastatură, RFID și senzorul PIR, controlând totodată LED-urile și buzzer-ul. 
- Tastatură Matriceală 4x4: Utilizată pentru introducerea codului de acces. Este conectată la Arduino prin 8 pini digitali. 
- Modul RFID RC522: Cititor/scriitor RFID care comunică prin protocolul SPI și operează la 13.56MHz. Acesta citește UID-urile cardurilor și le trimite către Arduino pentru verificare. 
- Buzzer: Oferă feedback sonor rapid pentru diverse evenimente, cum ar fi apăsările de taste sau rezultatul autentificării. 
- LED-uri (Verde și Albastru): Semnalizează vizual statusul sistemului (acces permis/respins).  Protejate de rezistențe. 
- Senzor PIR HW416: Detectează mișcarea și prezența, activând/dezactivând funcționalitățile sistemului. 
- Breadboard: Placă de prototipare pentru montajul temporar al componentelor și realizarea rapidă a circuitului. 
- LCD 16x2 (cu modul I2C): (Vizibil în schema Fritzing) utilizat pentru afișarea locală a mesajelor și stărilor sistemului. 

---

## Schema Electrică și Conexiuni

- Conexiunile principale între componente sunt următoarele: 
- Arduino - Tastatură Matriceală: Rândurile și coloanele tastaturii sunt conectate la pinii digitali ai Arduino-ului. 
- Arduino - Modul RFID RC522: Pinul SDA al modulului RFID este conectat la pinul digital 10 al Arduino-ului. Pinii SCK, MOSI, MISO și RST sunt conectați conform specificațiilor SPI (la pinii 13, 11, 12, respectiv 9). 
- Arduino - Buzzer: Conectat la un pin digital al Arduino-ului (pinul 5 în cod). 
- Arduino - LED-uri: Conectate la pini digitali (verde la pin 3, albastru la pin 4) prin rezistențe de 220 ohmi. 
- Arduino - Senzor PIR: Conectat la un pin digital al Arduino-ului pentru detectarea mișcării. 
- Alimentare: Arduino este alimentat prin portul USB, iar toate celelalte componente sunt alimentate din aceeași sursă. 

O schemă vizuală detaliată a montajului este disponibilă mai jos, realizată în Fritzing:
![fritzing](https://github.com/user-attachments/assets/d795d6ec-bb5f-4726-afc8-8bc5797bcf3d)

---

## Codul Sursă

Codul este scris în limbajul Arduino (C/C++) și utilizează mai multe biblioteci pentru a interacționa cu componentele hardware.

#include <Keypad.h>          // Include biblioteca Keypad pentru tastatura matriceală
#include <SPI.h>             // Include biblioteca SPI pentru comunicarea cu modulul RFID
#include <MFRC522.h>         // Include biblioteca MFRC522 pentru cititorul RFID
#include <LiquidCrystal_I2C.h> // Include biblioteca pentru controlul LCD I2C

#define SS_PIN 10            // Definește pinul Slave Select (SS) pentru modulul RFID
#define RST_PIN 9            // Definește pinul Reset (RST) pentru modulul RFID

// Creează o instanță a clasei MFRC522, specificând pinii SS și RST
MFRC522 mfrc522(SS_PIN, RST_PIN);
// Creează o instanță a clasei LiquidCrystal_I2C pentru LCD
// (adresa I2C, număr coloane, număr rânduri - adresa tipică 0x27)
LiquidCrystal_I2C lcd(0x27, 16, 2);

const byte ROWS = 4;         // Definește numărul de rânduri al tastaturii
const byte COLS = 4;         // Definește numărul de coloane al tastaturii

// Definește layout-ul tastaturii (caracterele asociate fiecărui buton)
char keys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

// Definește pinii Arduino conectați la rândurile tastaturii
byte rowPins[ROWS] = {8, 7, 6, 5};
// Definește pinii Arduino conectați la coloanele tastaturii
byte colPins[COLS] = {4, 3, 2, 1};

// Creează o instanță a clasei Keypad, utilizând layout-ul și pinii definiți
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// Definește pinii pentru LED-uri și Buzzer
const int greenLed = 3;      // Pinul pentru LED-ul Verde (Acces Permis)
const int blueLed = 4;       // Pinul pentru LED-ul Albastru (Acces Respins)
const int buzzer = 5;        // Pinul pentru Buzzer
const int pirPin = 2;        // Pinul pentru senzorul PIR (detecție mișcare)

String correctCode = "1234"; // Codul de acces corect (ex: PIN)
String inputCode = "";       // Variabila pentru stocarea codului introdus de utilizator

// UID-ul (Unique ID) unei cartele RFID autorizate (exemplu)
// Acesta este UID-ul cartelei care va permite accesul
byte authorizedCard1[4] = {0x03, 0x9F, 0x94, 0x9A};

void setup() {
  Serial.begin(9600);        // Inițializează comunicarea serială la 9600 bps
  SPI.begin();               // Inițializează bus-ul SPI (necesar pentru RFID)
  mfrc522.PCD_Init();        // Inițializează modulul RFID MFRC522
  lcd.init();                // Inițializează LCD-ul
  lcd.backlight();           // Pornește iluminarea de fundal a LCD-ului

  // Setează pinii LED-urilor și buzzer-ului ca OUTPUT
  pinMode(greenLed, OUTPUT);
  pinMode(blueLed, OUTPUT);
  pinMode(buzzer, OUTPUT);
  // Setează pinul senzorului PIR ca INPUT
  pinMode(pirPin, INPUT);

  // Afișează un mesaj inițial pe LCD
  lcd.print("System Ready!");
  delay(2000);               // Așteaptă 2 secunde
  lcd.clear();               // Șterge conținutul LCD-ului
}

void loop() {
  // Verifică starea senzorului PIR
  int pirState = digitalRead(pirPin);
  if (pirState == HIGH) {    // Dacă se detectează mișcare
    Serial.println("Motion Detected!"); // Afișează pe monitorul serial
    lcd.setCursor(0, 0);     // Setează cursorul LCD la rândul 0, coloana 0
    lcd.print("Motion Detected!"); // Afișează mesaj pe LCD
    // Aici se pot adăuga acțiuni specifice la detectarea mișcării (ex: activare sistem)
  } else {
    // Dacă nu se detectează mișcare, sistemul poate intra într-o stare de repaus
    // sau de consum redus de energie (nu este implementat detaliat aici)
  }

  // Citește tasta apăsată de pe tastatură
  char key = keypad.getKey();

  if (key) {                 // Dacă o tastă este apăsată
    Serial.print("Tasta apasata: "); // Afișează pe monitorul serial
    Serial.println(key);
    lcd.setCursor(0, 0);     // Afișează tasta apăsată pe LCD
    lcd.print("Tasta: ");
    lcd.print(key);

    if (key == '#') {        // Dacă tasta '#' este apăsată (confirmare cod)
      Serial.print("Cod introdus: "); // Afișează codul introdus
      Serial.println(inputCode);
      lcd.setCursor(0, 1);   // Afișează codul introdus pe rândul 1 al LCD-ului
      lcd.print("Cod: ");
      lcd.print(inputCode);

      if (inputCode == correctCode) { // Compară codul introdus cu cel corect
        allowAccess("Cod corect!"); // Dacă este corect, permite accesul
      } else {
        denyAccess("Cod gresit!"); // Dacă este greșit, refuză accesul
      }
      inputCode = "";        // Resetează variabila pentru următorul cod
    } else if (key == '*') { // Dacă tasta '*' este apăsată (resetare cod)
      inputCode = "";        // Golește codul introdus
      Serial.println("Cod resetat."); // Afișează mesaj de resetare
      lcd.clear();           // Șterge LCD-ul
      lcd.print("Cod resetat."); // Afișează mesaj de resetare pe LCD
      delay(1000);           // Așteaptă 1 secundă
      lcd.clear();           // Șterge LCD-ul
    } else {
      inputCode += key;      // Adaugă tasta apăsată la codul introdus
    }
  }

  // Verifică modulul RFID pentru o nouă cartelă prezentă și îi citește UID-ul
  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    Serial.print("Card detectat: ");
    String cardUid = "";
    // Parcurge și afișează fiecare octet al UID-ului cartelei detectate
    for (byte i = 0; i < mfrc522.uid.size; i++) {
      Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "); // Formatează pentru afișare
      Serial.print(mfrc522.uid.uidByte[i], HEX); // Afișează octetul în format HEX
      cardUid += String(mfrc522.uid.uidByte[i] < 0x10 ? "0" : ""); // Construiește string-ul UID
      cardUid += String(mfrc522.uid.uidByte[i], HEX);
    }
    Serial.println();
    Serial.print("Card UID: ");
    Serial.println(cardUid);

    lcd.setCursor(0, 0);     // Afișează mesaj de detectare card pe LCD
    lcd.print("Card detectat!");
    lcd.setCursor(0, 1);     // Afișează UID-ul cardului pe LCD (parțial, dacă este lung)
    lcd.print("UID: ");
    lcd.print(cardUid.substring(0, 8));

    // Compară UID-ul cartelei detectate cu UID-ul autorizat
    if (compareRFID(mfrc522.uid.uidByte, authorizedCard1)) {
      allowAccess("Card autorizat!"); // Dacă este autorizat, permite accesul
    } else {
      denyAccess("Card neautorizat!"); // Dacă nu este autorizat, refuză accesul
    }

    mfrc522.PICC_HaltA();    // Oprește PICC (Proximity Integrated Circuit Card)
    mfrc522.PCD_StopCrypto1(); // Oprește criptarea pe PCD (Proximity Coupling Device)
  }
}

// Funcție pentru a semnala acces permis
void allowAccess(String message) {
  digitalWrite(greenLed, HIGH); // Aprinde LED-ul verde
  digitalWrite(blueLed, LOW);   // Asigură că LED-ul albastru este stins
  tone(buzzer, 1000);           // Generează un ton la buzzer (1000 Hz)
  Serial.println(message + " Acces permis."); // Afișează mesaj pe serial
  lcd.clear();                  // Șterge LCD-ul
  lcd.print(message);           // Afișează mesajul pe LCD
  lcd.setCursor(0, 1);
  lcd.print("Acces permis.");
  delay(3000);                  // Așteaptă 3 secunde
  noTone(buzzer);               // Oprește tonul buzzer-ului
  digitalWrite(greenLed, LOW);  // Stinge LED-ul verde
  lcd.clear();                  // Șterge LCD-ul
}

// Funcție pentru a semnala acces refuzat
void denyAccess(String message) {
  digitalWrite(blueLed, HIGH);  // Aprinde LED-ul albastru
  digitalWrite(greenLed, LOW);  // Asigură că LED-ul verde este stins
  tone(buzzer, 500);            // Generează un ton la buzzer (500 Hz)
  Serial.println(message + " Acces refuzat."); // Afișează mesaj pe serial
  lcd.clear();                  // Șterge LCD-ul
  lcd.print(message);           // Afișează mesajul pe LCD
  lcd.setCursor(0, 1);
  lcd.print("Acces refuzat.");
  delay(3000);                  // Așteaptă 3 secunde
  noTone(buzzer);               // Oprește tonul buzzer-ului
  digitalWrite(blueLed, LOW);   // Stinge LED-ul albastru
  lcd.clear();                  // Șterge LCD-ul
}

// Funcție pentru a compara două UID-uri RFID
bool compareRFID(byte* uid1, byte* uid2) {
  for (byte i = 0; i < 4; i++) { // Compară octet cu octet (presupunând UID de 4 octeți)
    if (uid1[i] != uid2[i]) {    // Dacă orice octet nu corespunde
      return false;              // Returnează fals (UID-uri diferite)
    }
  }
  return true;                   // Dacă toți octeții corespund, returnează adevărat (UID-uri identice)
}

---

## Biblioteci Utilizate
- Keypad.h: Pentru a citi input-ul de la tastatura matriceală. 
- SPI.h: Pentru comunicarea cu modulul RFID. 
- MFRC522.h: Pentru comunicarea cu modulul RFID RC522. 
- LiquidCrystal_I2C.h: Pentru controlul afișajului LCD 16x2.
Adafruit BusIO.h, Adafruit LiquidCrystal.h, Adafruit MCP23017.h: Biblioteci auxiliare pentru comunicarea I2C/SPI și controlul LCD/expandoarelor GPIO, dacă sunt utilizate. 

---

## Descrierea Logică a Codului

Programul implementează următoarele funcționalități logice: 

- Citirea Tastaturii Matriceale: Utilizează biblioteca Keypad pentru a detecta apăsările de taste și a construi codul introdus de utilizator. 
- Autentificarea RFID: Citește UID-ul cardurilor RFID detectate și îl compară cu un UID predefinit (authorizedCard1). 
- Controlul LED-urilor și Buzzer-ului: Aprinde LED-ul verde (greenLed) și activează buzzer-ul la acces permis, sau aprinde LED-ul albastru (blueLed) și activează buzzer-ul pentru acces respins. 
- Detectarea Mișcării cu PIR: Senzorul PIR monitorizează continuu zona și trimite semnale către Arduino pentru a activa/dezactiva sistemul, deși această parte specifică de integrare nu este detaliată în snippetul de cod furnizat, ci este menționată în descrierea generală. 
- Resetarea Codului: Codul introdus de utilizator poate fi resetat folosind tasta '*'. 

---

## Rulare și Utilizare

1. Instalați Arduino IDE: Asigurați-vă că aveți mediul de dezvoltare Arduino instalat.
2. Instalați Bibliotecile: Descărcați și instalați bibliotecile Keypad, MFRC522 și LiquidCrystal_I2C (și celelalte dacă sunt necesare pentru configurația completă a sistemului dvs.) prin Managerul de Biblioteci din Arduino IDE. 
3. Conectați Hardware-ul: Realizați conexiunile conform schemei electrice (Fritzing).
4. Încărcați Codul: Copiați codul sursă în Arduino IDE și încărcați-l pe placa Arduino Plusivo.
5. Monitor Serial: Deschideți Monitorul Serial în Arduino IDE (cu baud rate 9600) pentru a vizualiza mesajele de sistem și starea interfonului.
6. Interacțiune: Sistemul este activat la pornire. Puteți introduce un cod PIN (implicit 1234) sau utiliza un card RFID autorizat pentru a testa accesul. 

---

## Realizat de

Proiect realizat de [bogdanch7](https://github.com/bogdanch7)

An universitar 2024–2025

---

## Licență

Acest proiect este creat pentru uz educațional.
