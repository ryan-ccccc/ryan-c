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

Later, we will attempt to add a stepper motor so the scanner can move in another direction and collect more measurements.

Starting with only the servo made the project easier to test because we could focus on one movement system before adding another motor.

---

## How We Used AI

John and I used ChatGPT with GPT-6 Astra to help develop the servo and ultrasonic scanner. We used it to generate starting code, explain unfamiliar functions, and understand how to make the movement and distance measurements happen in order.

My idea was to make the servo stop every 15 degrees so the sensor could take a reading at each position. Most of the starting program came from AI. John and I chose the movement interval, connected the components, mounted the sensor, and tested the setup.

### What We Asked AI

My first request was about combining the servo movement with distance detection. A shortened version of the prompt was:

> How do I make a servo stop every 15 degrees and use a distance sensor to detect an object and scan it?

AI suggested a positional servo and an HC-SR04 ultrasonic sensor. It generated a program that moved the servo from 15 to 165 degrees in 15-degree intervals, took a measurement at each position, and then scanned back.

The generated program included the movement loops, a function for reading distance, pauses between actions, and Serial output. This gave us a starting point for testing the sensor and servo together.

After receiving the code, we asked follow-up questions about what the functions did, why the servo needed a delay, and how the sensor converted echo time into centimetres. Our questions became more focused as we worked through the program.

### Understanding How the Servo Stops

One command we needed to understand was:

```cpp
scannerServo.write(angle);
```

This sends the positional servo a target angle. The servo still needs time to move there, so the code waits before measuring.

These settings control the spacing between positions and the waiting time:

```cpp
const int stepAngle = 15;
const int settleTime = 500;
```

The 15-degree interval came from my original request. The half-second settling time was included in the AI-generated code. It gives the servo time to reach its position before the sensor takes a reading. The program also waits another half second after measuring before moving again.

Understanding these settings helped me follow the sequence on the actual setup: the servo moves, pauses, the sensor measures, and then the servo continues.

### Understanding the Sensor Readings

We also wanted to understand how the sensor produced a distance. The program uses:

```cpp
unsigned long duration =
  pulseIn(echoPin, HIGH, 30000UL);

float distance = duration / 58.0;
```

The ultrasonic sensor sends out sound, and `pulseIn()` measures the length of the returning echo signal in microseconds. Dividing that time by `58.0` gives an approximate distance in centimetres, accounting for the sound travelling to the object and back.

The `30000UL` value sets a timeout. If no echo arrives within that time, the function returns zero rather than leaving the program waiting indefinitely.

AI also included checks for failed measurements:

```cpp
if (duration == 0) return -1;

if (distance < 2 || distance > 400) {
  return -1;
}
```

The program uses `-1` to identify an unusable reading and prints `no_reading` in the Serial Monitor. This prevents a failed reading from appearing as a real distance of zero.

The Serial Monitor prints the commanded angle followed by the measured distance. For example, `45,23.6` would mean a command of 45 degrees and a reading of 23.6 cm. This is an example of the format, not a recorded test result.

### Testing the Starting Program

John and I connected the sensor and servo and mounted the sensor so it turned with the servo arm. We tested whether the program could move through the positions and collect distance readings during the pauses.

The servo moved back and forth in 15-degree intervals while the program collected readings. This showed that the movement and measurement sequence worked together. We still needed to check the accuracy before treating the readings as a reliable outline of an object.

The angle printed by the program is the angle requested in the code. It does not independently measure the servo's actual position. Understanding that helped me separate what the program was commanding from what had been physically verified.

### Problems and Limits With Using AI

AI was useful, but it did not mean that the answers automatically worked with our physical project.

One issue was that AI had to make assumptions about our hardware. For example, some of the stepper code assumed a common 200-step motor and a certain driver setup. We still had to compare that information with the tutorial and the actual components in front of us.

AI could also explain how something should be connected in theory, but it could not see every detail of our real wiring unless we gave it that information. This became especially important when we started dealing with the stepper motor power supply and current limit.

There were also times when an AI response gave us much more code or information than we actually needed. We had to narrow down our questions and ask about one stage at a time. That is one reason we tested the project in separate parts instead of trying to build the full scanner immediately.

Our process became:

```text
Come up with the next goal
↓
Ask AI or use the tutorial for a starting point
↓
Read and understand the important parts
↓
Build or wire it ourselves
↓
Test it on the real hardware
↓
Find what does not work
↓
Ask more specific questions or change the setup
↓
Test again
```

AI did most of the initial coding for the servo scanner and helped us develop the combined scanner code, but John and I still had to physically build the system, choose how we wanted it to move, test each component, compare the AI suggestions with the tutorial, and decide what changes actually worked.

The biggest thing we learned from using AI was that getting code from AI is different from having a working project. We still needed to understand enough of the code to test it, recognize when something did not match our hardware, and explain what each major part was doing.

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

<img width="410" height="540" alt="93fe708cf5f01cc86b67540fb686ab30" src="https://github.com/user-attachments/assets/5e53eb87-e2c1-492c-afcd-d93bbabb872d" />

<img width="410" height="540" alt="ce49f9da8a1fa438a202cf5a1ccbbaed" src="https://github.com/user-attachments/assets/b2635698-0a25-4f9e-b16e-2c45e636c6f8" />


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

John and I testing the servo scanner as it stops at different angles and takes distance measurements.

---

## Adding the Stepper Motor

After getting the servo and distance sensor working, John and I moved on to the next part of our plan: adding a stepper motor.

The goal of the stepper motor is to eventually move the scanner in another direction while the servo changes the angle of the distance sensor. This should let us collect measurements from more positions instead of only sweeping side to side.

We had not worked with this type of stepper motor and driver before, so we followed this tutorial:

[Control a NEMA 17 Stepper Motor with A4988 Driver and Arduino](https://www.youtube.com/watch?v=wcLeXXATCR4)

The tutorial showed us how to connect the NEMA 17 stepper motor, A4988 motor driver, Arduino, and external power supply.

For this stage, the tutorial was our main source rather than AI. We followed the wiring shown in the video and stopped at different points to make sure we understood what each connection was doing before continuing.

---

## Stepper Motor Setup

One thing we learned was that the stepper motor is not controlled directly from the Arduino. The A4988 driver is between the Arduino and the motor.

The Arduino sends the driver instructions such as which direction to move and when to take a step. The driver handles the higher current needed by the stepper motor.


<img width="410" height="540" alt="1e07e59390015b27927fce45a7b183a8" src="https://github.com/user-attachments/assets/fea6032e-a2e3-46b5-8117-0685baaddb4f" />

<img width="410" height="540" alt="61abceb8b2d4e09d34a9bf9b8b7f6a9c" src="https://github.com/user-attachments/assets/d9cfdedf-a3e4-46d2-84a8-5f3d8db144c6" />

---

## Setting the Current Limit

Before running the stepper motor, we needed to set the current limit on the A4988 driver.

The small potentiometer on the driver controls how much current the motor is allowed to receive, so we used a multimeter while adjusting it instead of guessing.

This part took longer than we expected because we ran into a few problems. At first, the multimeter stopped working because its battery ran out. After replacing the battery, it still did not seem to give us the reading we expected, so we had to stop and check the setup again.

We checked the multimeter settings, where the probes were connected, and how we were measuring the driver. Once we got the multimeter working again, we continued adjusting the small potentiometer on the A4988 while checking the reading.


<video width="700" controls style="max-width: 100%;">
  <source src="videos/stepper-current-limit.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

This was one of the messier parts of the project because the problem was not with our code. It was with the tool we were using to test the hardware, so we had to figure that out before we could keep working on the stepper motor. This step helped us understand that connecting a motor is not only about getting the wires in the correct places. We also had to make sure the driver was set correctly for the motor before we started controlling its movement.

---

## Current Progress

At this point, we have started setting up the stepper motor and driver and have adjusted the current limit. Our next step is to get the Arduino controlling the stepper motor and then figure out how to combine its movement with the servo and distance sensor.

## Connecting and Testing the Stepper Motor

After setting the current limit, John and I connected the stepper motor to the A4988 driver.

The stepper motor has four wires that connect to the motor outputs on the driver. This part was a little confusing because the wires are connected in pairs inside the motor, and they have to go to the correct motor terminals on the driver. We had to pay attention to which wires belonged together instead of just connecting them randomly.

The Arduino also connects to the driver using two important control pins:

```text
DIR  → Arduino Pin 2
STEP → Arduino Pin 3
```

`DIR` controls which direction the motor turns. `STEP` controls when the motor moves one step.

At this point, our goal was not to connect the stepper motor to the whole scanner yet. We first wanted to see if we could make the motor turn by itself.

---

## Making the Stepper Motor Turn

For our first test, we copied the basic stepper motor code from the YouTube tutorial we were following.

```cpp
const int dirPin = 2;
const int stepPin = 3;

void setup() {
  pinMode(dirPin, OUTPUT);
  pinMode(stepPin, OUTPUT);

  delay(2000);
  digitalWrite(dirPin, HIGH);
}

void loop() {
  digitalWrite(stepPin, HIGH);
  delayMicroseconds(5000);

  digitalWrite(stepPin, LOW);
  delayMicroseconds(5000);
}
```

Even though we copied this starting code from the tutorial, we went through it so we understood what it was doing.

These lines:

```cpp
const int dirPin = 2;
const int stepPin = 3;
```

tell the Arduino which pins are connected to the `DIR` and `STEP` inputs on the motor driver.

In `setup()`, these lines:

```cpp
pinMode(dirPin, OUTPUT);
pinMode(stepPin, OUTPUT);
```

make both pins outputs because the Arduino is sending signals to the driver.

The line:

```cpp
digitalWrite(dirPin, HIGH);
```

sets the direction of the motor. Changing it from `HIGH` to `LOW` would make the motor turn in the opposite direction.

The most important part is:

```cpp
digitalWrite(stepPin, HIGH);
delayMicroseconds(5000);
digitalWrite(stepPin, LOW);
delayMicroseconds(5000);
```

The STEP pin repeatedly switches between HIGH and LOW. Each pulse tells the driver to move the stepper motor another step.

The delays also affect how quickly the pulses are sent. With a `5000` microsecond delay between changes, the motor turns relatively slowly. A shorter delay would send the pulses faster and make the motor turn faster.

This first test helped us separate the project into smaller pieces. Before trying to combine the stepper motor with the servo and ultrasonic sensor, we wanted to make sure the motor, driver, wiring, and basic code could work on their own.


<video width="700" controls style="max-width: 100%;">
  <source src="videos/stepper-motor-test.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

John and I testing the stepper motor after connecting it to the A4988 driver and Arduino.

## Problem: Powering the Stepper Motor

When we tried to add the stepper motor to the scanner, we ran into a problem with the power setup. The voltage and power needed for the stepper motor were much higher than what we were already using for the servo and sensor, and we were worried that connecting it incorrectly could damage the scanner or the Arduino.

We started trying to connect the stepper motor setup, but during the process John received a small electrical shock. At that point, we stopped working with the powered circuit instead of continuing to experiment with it.

This made us realize that we needed to understand the power requirements and wiring more clearly before combining the stepper motor with the rest of the scanner. Rather than risk damaging the components or getting shocked again, we decided to pause this part and check the power setup before continuing.

Before testing it again, we plan to have the wiring and power supply checked and make sure the circuit is powered off while making connections.

When we reached this point, we ran out of time to rewire and reconnect everything so we only could end at this point but we still think that we achieved and learned a lot during this process.
