# SMART-CAR-PARKING-SYSTEM
#include <EEPROM.h>       // For storing the slot status
#include <LiquidCrystal.h> // For the LCD display
#include <SoftwareSerial.h> // For GSM module communication

// Define LCD pins
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Define pins for ultrasonic sensors
const int trigPin1 = 9;  
const int echoPin1 = 8;
const int trigPin2 = 7;
const int echoPin2 = 6;

// Define pins for LEDs
const int ledPin1Green = 10;
const int ledPin1Red = 13;
const int ledPin2Green = A1;
const int ledPin2Red = A2;

// GSM module pins and serial communication
SoftwareSerial gsmSerial(2, 3);  // RX, TX for GSM

// Parking slot distance threshold
const int distanceThreshold = 10;

// Function to measure the distance using ultrasonic sensor
long measureDistance(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  long duration = pulseIn(echoPin, HIGH);
  long distance = (duration * 0.034) / 2;
  return distance;
}

// Send SMS via GSM module
void sendSMS(const char* message) {
  gsmSerial.println("AT+CMGF=1");  // Set GSM to text mode
  delay(100);
  gsmSerial.println("AT+CMGS=\"+91xxxxxxxxxx\""); // Your phone number
  delay(100);
  gsmSerial.println(message);
  delay(100);
  gsmSerial.write(26); // End the SMS
  delay(1000);
}

void setup() {
  // Initialize serial communication
  Serial.begin(9600);
  gsmSerial.begin(9600);
  
  // Initialize LCD
  lcd.begin(16, 2);
  
  // Set the pins for the ultrasonic sensors, LEDs
  pinMode(trigPin1, OUTPUT);
  pinMode(echoPin1, INPUT);
  pinMode(trigPin2, OUTPUT);
  pinMode(echoPin2, INPUT);
  
  pinMode(ledPin1Green, OUTPUT);
  pinMode(ledPin1Red, OUTPUT);
  pinMode(ledPin2Green, OUTPUT);
  pinMode(ledPin2Red, OUTPUT);
  
  // Display initial message on LCD
  lcd.setCursor(0, 0);
  lcd.print("Parking System");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Measure distance for parking slot 1
  long distance1 = measureDistance(trigPin1, echoPin1);
  long distance2 = measureDistance(trigPin2, echoPin2);

  // Check slot 1 availability and update LEDs
  if (distance1 < distanceThreshold) {
    digitalWrite(ledPin1Red, HIGH);
    digitalWrite(ledPin1Green, LOW);
    lcd.setCursor(0, 0);
    lcd.print("Slot 1: OCCUPIED ");
    EEPROM.write(0, 1);  // Store slot 1 status
  } else {
    digitalWrite(ledPin1Green, HIGH);
    digitalWrite(ledPin1Red, LOW);
    lcd.setCursor(0, 0);
    lcd.print("Slot 1: AVAILABLE");
    EEPROM.write(0, 0);  // Store slot 1 status
  }

  // Check slot 2 availability and update LEDs
  if (distance2 < distanceThreshold) {
    digitalWrite(ledPin2Red, HIGH);
    digitalWrite(ledPin2Green, LOW);
    lcd.setCursor(0, 1);
    lcd.print("Slot 2: OCCUPIED ");
    EEPROM.write(1, 1);  // Sto
