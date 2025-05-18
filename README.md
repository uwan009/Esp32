# Esp32
Monitoring Antar Komponen
#include <PZEM004Tv30.h>
#include <HardwareSerial.h>

// ----------------- PZEM Configuration -----------------
HardwareSerial pzemSerial(1);
PZEM004Tv30 pzem(pzemSerial, 16, 17);

// ----------------- Rain Sensor & Relay -----------------
const int sensorPin = 33;       // A0 untuk sensor hujan
const int relayPLN = 26;        // Relay PLN (aktif LOW)
const int relayPLTS = 27;       // Relay PLTS (aktif LOW)
const int threshold = 1500;     // Ambang sensor hujan

void setup() {
  Serial.begin(115200);
  delay(1000);

  // Inisialisasi UART1 untuk PZEM
  pzemSerial.begin(9600, SERIAL_8N1, 16, 17);
  Serial.println("Monitoring PLN via PZEM-004T v3.0");

  // Konfigurasi relay
  pinMode(relayPLN, OUTPUT);
  pinMode(relayPLTS, OUTPUT);

  // Awal: aktifkan PLTS, matikan PLN
  digitalWrite(relayPLN, HIGH);   // Relay PLN OFF
  digitalWrite(relayPLTS, LOW);   // Relay PLTS ON
}

void loop() {
  // ----------------- Cek Sensor Hujan -----------------
  int nilaiSensor = analogRead(sensorPin);
  Serial.print("Sensor Hujan: ");
  Serial.println(nilaiSensor);

  if (nilaiSensor < threshold) {
    // Hujan terdeteksi: pindah ke PLN
    digitalWrite(relayPLN, LOW);    // Relay PLN ON
    digitalWrite(relayPLTS, HIGH);  // Relay PLTS OFF
    Serial.println("HUJAN TERDETEKSI → SUMBER DI PLN");
  } else {
    // Tidak hujan: tetap di PLTS
    digitalWrite(relayPLN, HIGH);   // Relay PLN OFF
    digitalWrite(relayPLTS, LOW);   // Relay PLTS ON
    Serial.println("TIDAK HUJAN → SUMBER DI PLTS");
  }

  // ----------------- Monitoring PZEM -----------------
  float voltage = pzem.voltage();
  float current = pzem.current();
  float power   = pzem.power();
  float energy  = pzem.energy();
  float frequency = pzem.frequency();
  float pf = pzem.pf();

  Serial.println("----- Data PZEM -----");

  if (!isnan(voltage)) Serial.print("Voltage: "), Serial.print(voltage), Serial.println(" V");
  else Serial.println("Voltage: Read failed");

  if (!isnan(current)) Serial.print("Current: "), Serial.print(current), Serial.println(" A");
  else Serial.println("Current: Read failed");

  if (!isnan(power)) Serial.print("Power: "), Serial.print(power), Serial.println(" W");
  else Serial.println("Power: Read failed");

  if (!isnan(energy)) Serial.print("Energy: "), Serial.print(energy), Serial.println(" Wh");
  else Serial.println("Energy: Read failed");

  if (!isnan(frequency)) Serial.print("Frequency: "), Serial.print(frequency), Serial.println(" Hz");
  else Serial.println("Frequency: Read failed");

  if (!isnan(pf)) Serial.print("Power Factor: "), Serial.println(pf);
  else Serial.println("Power Factor: Read failed");

  Serial.println("----------------------\n");

  delay(2000); // delay sebelum loop ulang
}
