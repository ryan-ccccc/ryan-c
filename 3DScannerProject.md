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


# 3D Scanner Project

## Starting the Project

For this project, John and I wanted to work toward making a simple 3D scanner with Arduino.

Our final idea is to have an ultrasonic distance sensor move in more than one direction so it can collect distance measurements from different positions. Instead of trying to build the entire scanner at once, we decided to first get one part working.

Our first goal was:

**Servo moves the sensor → stops at an angle → measures distance → moves again**

Later, we plan to add a **stepper motor** so the scanner can move in another direction and collect more measurements.

Starting with only the servo made the project easier to test because we could focus on one movement system before adding another motor.

---

## Using AI

AI was a major part of how we got started with the code.

We explained that we wanted a servo to stop every 15 degrees and an ultrasonic sensor to measure distance at each position. AI gave us most of the first version of the Arduino code and helped us understand how the servo and distance sensor could work together.

We did not understand every line immediately, so we went through the code and figured out what the important parts were.

For example:

```cpp
scannerServo.write(angle);
```

tells the servo which angle to move to.

```cpp
const int stepAngle = 15;
```

controls how far the servo moves each time.

We also learned that:

```cpp
pulseIn(echoPin, HIGH, 30000UL);
```

measures how long the ultrasonic echo takes to return.

The distance is then calculated with:

```cpp
float distance = duration / 58.0;
```

AI helped us create the starting code, while John and I were responsible for wiring the parts, testing the system, deciding how we wanted the scanner to move, and understanding what the code was doing.

---

## Building the First Version

We connected an HC-SR04 ultrasonic sensor to the Arduino and mounted it so it could move with the servo.

Our current setup uses:

```text
HC-SR04 VCC  → 5V
HC-SR04 GND  → GND
HC-SR04 TRIG → Pin 10
HC-SR04 ECHO → Pin 11

Servo signal → Pin 9
```

<!-- PHOTO: Add a photo of the whole scanner setup here -->

<img width="820" height="1080" alt="93fe708cf5f01cc86b67540fb686ab30" src="https://github.com/user-attachments/assets/5e53eb87-e2c1-492c-afcd-d93bbabb872d" />

<img width="820" height="1080" alt="ce49f9da8a1fa438a202cf5a1ccbbaed" src="https://github.com/user-attachments/assets/b2635698-0a25-4f9e-b16e-2c45e636c6f8" />


**Figure 1.** Our current servo and ultrasonic sensor setup.

---

## Figuring Out the Movement

At first, our main focus was just getting the servo to move to specific positions instead of continuously moving.

We used:

```cpp
scannerServo.write(angle);
```

and changed the angle by 15 degrees each time.

The movement is:

```text
15°
30°
45°
60°
75°
...
165°
```

We also realized that the sensor should not measure immediately after the servo starts moving. The servo needs time to reach its position first.

That is why the code includes:

```cpp
delay(settleTime);
```

The delay gives the servo time to stop before the distance measurement is taken.

This was important because the sensor would be less useful if it was taking measurements while it was still moving.

---

## Distance Measurements

After the servo reaches an angle, the ultrasonic sensor sends out a pulse.

The Arduino measures how long the echo takes to return and converts that time into centimeters.

The Serial Monitor gives us results like:

```text
15,35.2
30,30.8
45,23.6
```

For example:

```text
45,23.6
```

means the servo was around **45 degrees** and the detected object was about **23.6 cm away**.

This gives us two pieces of information for every measurement:

**angle + distance**

---

## Troubleshooting the Scan

One thing we had to think about was the timing between moving the servo and measuring the object.

If the program moves too quickly, the sensor can take a measurement before the servo has fully reached its angle.

We used:

```cpp
const int settleTime = 500;
```

to make the Arduino wait half a second.

This made the scan slower, but it also made each measurement happen after the servo had time to stop.

We also added a check for readings outside the useful range:

```cpp
if (distance < 2 || distance > 400) {
  return -1;
}
```

Instead of treating a bad measurement as a real distance, the program prints:

```text
no_reading
```

This helped us separate useful measurements from ones where the ultrasonic sensor did not receive a good echo.

---

## Current Code

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

At this point, the scanner can move the ultrasonic sensor back and forth in 15-degree steps and collect a distance measurement at each angle.

<video width="700" controls style="max-width: 100%;">
  <source src="videos/servo-scanner-test.mp4" type="video/mp4">
</video>

**Video 1.** John and I testing the servo scanner as it stops at different angles and takes distance measurements.

---

## Next Step

This is still only one part of the scanner.

Our next goal is to add a **stepper motor**. The servo will control the angle of the ultrasonic sensor, while the stepper motor will move the scanner through another direction.

Before adding that, we wanted to make sure we understood how the current servo movement and distance measurements worked on their own.


