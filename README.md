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
