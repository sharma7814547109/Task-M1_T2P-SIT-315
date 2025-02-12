#define TRIG_PIN 2  // Trigger pin for HC-SR04
#define ECHO_PIN 3  // Echo pin (must be an interrupt pin)
#define LED_PIN 13  // Built-in LED on Arduino
#define EXT_LED 7   // External LED

volatile bool pulseReceived = false;
volatile long startTime = 0;
volatile long endTime = 0;

void echoISR() {
    if (digitalRead(ECHO_PIN) == HIGH) {
        startTime = micros();  // Capture pulse start time
    } else {
        endTime = micros();    // Capture pulse end time
        pulseReceived = true;
    }
}

void setup() {
    pinMode(TRIG_PIN, OUTPUT);
    pinMode(ECHO_PIN, INPUT);
    pinMode(LED_PIN, OUTPUT);
    pinMode(EXT_LED, OUTPUT);
    Serial.begin(9600);

    attachInterrupt(digitalPinToInterrupt(ECHO_PIN), echoISR, CHANGE);
}

void loop() {
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);

    delay(50);  // Give time for the echo

    if (pulseReceived) {
        long duration = endTime - startTime;
        int distance = duration / 29 / 2;

        Serial.print("Distance: ");
        Serial.print(distance);
        Serial.println(" cm");

        if (distance > 0 && distance < 100) {
            digitalWrite(LED_PIN, HIGH);
            digitalWrite(EXT_LED, HIGH);
            Serial.println("Object Detected! LEDs ON");
        } else {
            digitalWrite(LED_PIN, LOW);
            digitalWrite(EXT_LED, LOW);
            Serial.println("No Object Nearby. LEDs OFF");
        }

        pulseReceived = false;
    }
    delay(500);
}
