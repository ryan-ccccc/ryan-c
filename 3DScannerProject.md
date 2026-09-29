<style>
header {
  display: none;
}

footer {
  display: none;
}

.wrapper {
  width: 95%;
  max-width: none;
  margin: 0 auto;
}

section {
  width: 100%;
  float: none;
}

body {
  padding: 20px;
}
</style>


# Servo Distance Scanner

## Project Idea

For this stage of the project, John and I are building a simple scanner using a servo motor and an ultrasonic distance sensor.

The servo moves the sensor in 15° intervals. At each angle, it stops, waits, and measures how far away an object is.

The current system works like this:

**Servo moves → stops → sensor measures distance → Arduino prints angle and distance**

Our next goal is to add a stepper motor so the scanner can move in another direction and collect more measurements.

---

## Current Setup

We are using:

- Arduino Uno
- Servo motor
- HC-SR04 ultrasonic distance sensor
- Jumper wires
- External power for the servo

The distance sensor is mounted so it turns with the servo.

<img width="1280" height="1707" alt="93fe708cf5f01cc86b67540fb686ab30" src="https://github.com/user-attachments/assets/5e53eb87-e2c1-492c-afcd-d93bbabb872d" />

<img width="1280" height="1707" alt="ce49f9da8a1fa438a202cf5a1ccbbaed" src="https://github.com/user-attachments/assets/b2635698-0a25-4f9e-b16e-2c45e636c6f8" />

**Figure 1.** Our current scanner setup.

---

## How the Code Works

The servo is controlled using:

```cpp
#include <Servo.h>
```

The main command is:

```cpp
scannerServo.write(angle);
```

This moves the servo to a certain angle.

We set:

```cpp
const int stepAngle = 15;
```

so the servo moves like:

```text
15°
30°
45°
60°
...
165°
```

After moving, the code waits:

```cpp
delay(settleTime);
```

so the servo can stop before the ultrasonic sensor takes a measurement.

The sensor measures how long it takes for an ultrasonic pulse to return:

```cpp
unsigned long duration =
  pulseIn(echoPin, HIGH, 30000UL);
```

Then the time is converted into distance:

```cpp
float distance = duration / 58.0;
```

The Arduino prints both the angle and distance, for example:

```text
45,23.6
```

This means the sensor was at about 45° and detected something about 23.6 cm away.

---

## Current Scanner Code

```cpp
#include <Servo.h>

Servo scannerServo;

const int servoPin = 9;
const int trigPin = 10;
const int echoPin = 11;

const int stepAngle = 15;
const int settleTime = 500;

float readDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  unsigned long duration =
    pulseIn(echoPin, HIGH, 30000UL);

  if (duration == 0) return -1;

  float distance = duration / 58.0;

  if (distance < 2 || distance > 400) {
    return -1;
  }

  return distance;
}

void scanAtAngle(int angle) {
  scannerServo.write(angle);
  delay(settleTime);

  float distance = readDistance();

  Serial.print(angle);
  Serial.print(",");

  if (distance < 0) {
    Serial.println("no_reading");
  }
  else {
    Serial.println(distance, 1);
  }

  delay(500);
}

void setup() {
  Serial.begin(9600);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  scannerServo.attach(servoPin);

  scannerServo.write(90);
  delay(1000);
}

void loop() {

  for (int angle = 15; angle <= 165; angle += stepAngle) {
    scanAtAngle(angle);
  }

  for (int angle = 150; angle > 15; angle -= stepAngle) {
    scanAtAngle(angle);
  }
}
```

---

## Current Test

<!-- VIDEO: Add video of the scanner moving here -->

<video width="700" controls style="max-width: 100%;">
  <source src="videos/servo-scanner-test.mp4" type="video/mp4">
</video>

**Video 1.** John and I testing the servo as it moves the distance sensor through different angles.

---

## Next Step

The next part of the project is to add a **stepper motor**. The servo will keep changing the sensor's angle, while the stepper motor will move the scanner in another direction.

This should let us collect measurements from more positions instead of only scanning across one line.
