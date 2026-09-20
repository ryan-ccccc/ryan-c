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

# Arduino Joystick Snake Controller

## Project Overview

For this project, I chose to build on **analog input**. In class, we used a potentiometer to see how an Arduino can read a range of values instead of only HIGH or LOW. I wanted to use analog input for something more interactive, so I chose a joystick.

My final goal was to use a physical joystick connected to an Arduino to control a Snake game on my GitHub website.

The project developed in stages. I first got the Arduino to read the joystick and recognize left, right, up, down, and center. Then I changed those directions into simple letters that could be sent through Serial. After that, I made a test webpage that connected to the Arduino with Web Serial and displayed the direction it received. Once that worked, I found a basic Snake game online and connected my joystick input system to the game's movement controls.

The final system works like this:

**Joystick → Arduino → Serial through USB → Web Serial API → JavaScript → Snake movement**

---

## Choosing and Wiring the Joystick

The new component I chose was a joystick module.

It has five pins:

- **VCC** → power
- **GND** → ground
- **VRx** → horizontal movement
- **VRy** → vertical movement
- **SW** → button when the joystick is pressed down

I connected:

```text
VCC → 5V
GND → GND
VRx → A0
VRy → A1
SW  → Digital Pin 2
```

The joystick uses both analog and digital input. VRx and VRy are analog because the joystick can be in many positions. SW is digital because the button is either pressed or not pressed.

<img width="450px" height="auto" alt="f1c29299bacbc1c1220f0f30186f55bc" src="https://github.com/user-attachments/assets/8fb742e6-7460-4304-9f83-f819f84f6f58" />

**Figure 1.** My joystick connected to the Arduino. VRx and VRy connect to the analog input pins so the Arduino can read the joystick's position.

---

## Reading the Joystick

I started by reading the X and Y axes:

```cpp
int xValue = analogRead(xPin);
int yValue = analogRead(yPin);
```

Earlier in the code, I defined:

```cpp
const int xPin = A0;
const int yPin = A1;
```

This means `xValue` stores the horizontal position and `yValue` stores the vertical position.

`analogRead()` gives a value from about **0 to 1023**. The middle is around 512, so when the joystick is resting near the center, both values are usually somewhere around that number.

The Arduino does not automatically know that a number means "left" or "up." My code has to interpret the numbers and turn them into directions.

My final direction code is:

```cpp
if (xValue < 400) {
  Serial.println("R");
}
else if (xValue > 600) {
  Serial.println("L");
}
else if (yValue < 400) {
  Serial.println("D");
}
else if (yValue > 600) {
  Serial.println("U");
}
else {
  Serial.println("C");
}
```

The letters mean:

```text
R = Right
L = Left
D = Down
U = Up
C = Center
```

I used single letters because they are simple for the webpage to read later.

---

## Debugging the Joystick Directions

One of my first problems was that left and right were backwards.

My original idea was:

```cpp
if (xValue < 400) {
  Serial.println("L");
}
else if (xValue > 600) {
  Serial.println("R");
}
```

When I actually tested the joystick, moving it left produced the value I had assigned to RIGHT, and moving it right produced LEFT.

At first, I thought I had wired VRx incorrectly. After checking the readings, I realized the wiring was working. The X-axis on the joystick was simply oriented differently from what I expected.

I fixed it by switching what the values meant:

```cpp
if (xValue < 400) {
  Serial.println("R");
}
else if (xValue > 600) {
  Serial.println("L");
}
```

This helped me understand that the Arduino only sees numbers. My code decides what those numbers mean physically.

---

## Creating a Center Range

Another problem was that the joystick did not return exactly 512 every time I let go of it.

At first, I thought I could treat one exact value as the center. When I watched the readings, I saw that they moved slightly even when the joystick looked centered.

Instead of checking for one exact number, I created a **dead zone**:

```text
0–399      = direction
400–600    = center
601–1023   = opposite direction
```

This is why the program checks whether the value is below 400 or above 600.

Small changes inside the center range are ignored. This became important for Snake because I did not want tiny movements around the center to accidentally change the snake's direction.

The code checks X before Y using `else if`. This also means that if I push the joystick diagonally, the X direction gets priority. That is fine for Snake because the game only uses four directions and does not need diagonal movement.

---

## Testing the Output in Serial Monitor

Before trying to connect the Arduino to a website, I tested the directions in the Arduino Serial Monitor.

When I moved the joystick, the Arduino printed only:

```text
L
R
U
D
C
```

This was useful because it separated the project into smaller tests. If the letters were wrong in Serial Monitor, I knew I needed to fix the Arduino code before working on the webpage.

<!-- VIDEO: Put the video of you moving the joystick and the Arduino producing L/R/U/D/C here. -->

<video width="700" controls style="max-width: 100%;">
  <source src="videos/joystick-serial-test.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

## Testing the Joystick Button

The joystick also has a built-in button when I press directly down on it.

I connected SW to digital pin 2 and used:

```cpp
pinMode(swPin, INPUT_PULLUP);
```

With `INPUT_PULLUP`, the button works like this:

```text
HIGH = not pressed
LOW  = pressed
```

This seemed backwards at first, but the internal pull-up resistor keeps the pin HIGH until the button connects it to ground.

I also reused the debounce code from earlier in the unit. A physical button can briefly switch between HIGH and LOW several times during one press. The debounce code waits until the signal has stayed stable for 50 milliseconds before accepting the change.

```cpp
if ((millis() - lastDebounceTime) > debounceDelay) {
```

When a real press is detected, I toggle the state:

```cpp
toggleState = !toggleState;
```

The `!` means the opposite, so:

```text
0 → 1
1 → 0
```

The joystick button was not needed for controlling Snake, but testing it let me apply the digital-input and debounce work from earlier in the unit to the new component.

---

## Sending the Directions Through Serial

The Arduino and website needed a simple way to communicate.

The Arduino starts Serial communication with:

```cpp
Serial.begin(9600);
```

The number `9600` is the baud rate, or the communication speed.

Instead of sending a long message, the Arduino sends one command:

```cpp
Serial.println("L");
```

The Arduino's job is:

```text
Read joystick
↓
Interpret analog values
↓
Send a simple direction command
```

The website does not need to know whether the X value was 250 or 800. It only needs to know the direction the Arduino already decided.

That separation made the project easier to understand and debug.

---

## Connecting the Arduino to My GitHub Website

Before working with Snake, I created a simple webpage whose only job was to connect to the Arduino and display the direction it received.

I used the **Web Serial API** in JavaScript.

The webpage has a button:

```html
<button id="connectButton">Connect Arduino</button>
```

JavaScript finds that button with:

```javascript
const connectButton =
  document.getElementById("connectButton");
```

Then an event listener waits for me to click it:

```javascript
connectButton.addEventListener("click", async () => {
```

The connection does not happen automatically. I have to click the button and choose the Arduino serial port.

The code that opens the device chooser is:

```javascript
port = await navigator.serial.requestPort();
```

After selecting the Arduino, the website opens the connection:

```javascript
await port.open({
  baudRate: 9600
});
```

The website uses `9600` because it has to match:

```cpp
Serial.begin(9600);
```

on the Arduino.

<!-- PHOTO: Put the photo showing the browser asking you to choose cu.usbmodem101 here. -->

<img width="1706" height="1279" alt="7fe20a6f929e09e06bd69410d32fe0b1" src="https://github.com/user-attachments/assets/1ab20620-73f6-4bdb-900f-155965ccf166" />

---

## Reading Serial Data in JavaScript

Opening the port only creates the connection. The webpage still needs to continuously read the information coming from the Arduino.

I created:

```javascript
async function readArduino()
```

Inside it, I use:

```javascript
const decoder = new TextDecoder();
```

The serial connection sends data as bytes. `TextDecoder` converts those bytes into normal text that JavaScript can compare with `"L"`, `"R"`, `"U"`, or `"D"`.

The webpage gets access to the incoming stream with:

```javascript
reader = port.readable.getReader();
```

Then this line waits for new information:

```javascript
const { value, done } = await reader.read();
```

Because Serial is a continuous stream, data is not guaranteed to arrive as perfectly separated messages. I used a buffer:

```javascript
buffer += decoder.decode(value, { stream: true });
```

Then I split the text whenever Arduino's `Serial.println()` created a new line:

```javascript
const lines = buffer.split("\n");
```

This line:

```javascript
buffer = lines.pop();
```

keeps any unfinished piece of data for the next read.

For each completed line, I remove extra spaces:

```javascript
line = line.trim();
```

Then I can compare it with a direction:

```javascript
if (line === "L") {
  directionText.textContent = "LEFT";
}
```

I repeated the same check for R, U, and D.

At this stage, moving the physical joystick changed the direction text on my GitHub webpage. Snake was not involved yet. I wanted to prove that the Arduino-to-browser connection worked on its own first.

The system at that point was:

```text
Joystick
↓
Arduino
↓
L / R / U / D
↓
USB Serial
↓
Web Serial
↓
JavaScript
↓
Direction displayed on webpage
```

---

## Problem: Serial Monitor Blocked the Website

One problem happened when I tried to connect the webpage while the Arduino Serial Monitor was still open.

The webpage would not connect correctly.

I realized that Serial Monitor was already using the Arduino's serial port. The webpage was trying to access the same port at the same time.

I fixed it by closing Serial Monitor before pressing **Connect Arduino**.

My testing process became:

```text
Upload Arduino code
↓
Test in Serial Monitor if needed
↓
Close Serial Monitor
↓
Open GitHub webpage
↓
Press Connect Arduino
↓
Select Arduino port
```

After that, the webpage connected normally.

This helped me understand that the serial connection is not only something inside my code. The computer also has to manage which program is using the Arduino's USB connection.

---

## Original Arduino-to-Website Test Code

I kept my original test code because it shows the step between Serial Monitor and the final Snake project.

<details>
<summary><strong>Show Original Web Serial Test Code</strong></summary>

```html
<button id="connectButton">Connect Arduino</button>

<h2>Direction:</h2>
<p id="direction">Not connected</p>

<script>
let port;
let reader;
let buffer = "";

const connectButton = document.getElementById("connectButton");
const directionText = document.getElementById("direction");

connectButton.addEventListener("click", async () => {

  try {

    port = await navigator.serial.requestPort();

    await port.open({
      baudRate: 9600
    });

    directionText.textContent = "Connected!";

    readArduino();

  } catch (error) {

    console.log(error);

    directionText.textContent = "Connection failed.";
  }
});

async function readArduino() {

  const decoder = new TextDecoder();

  while (port.readable) {

    reader = port.readable.getReader();

    try {

      while (true) {

        const { value, done } = await reader.read();

        if (done) {
          break;
        }

        buffer += decoder.decode(value, { stream: true });

        const lines = buffer.split("\n");

        buffer = lines.pop();

        for (let line of lines) {

          line = line.trim();

          if (line === "L") {
            directionText.textContent = "LEFT";
          }

          else if (line === "R") {
            directionText.textContent = "RIGHT";
          }

          else if (line === "U") {
            directionText.textContent = "UP";
          }

          else if (line === "D") {
            directionText.textContent = "DOWN";
          }
        }
      }

    } finally {

      reader.releaseLock();

    }
  }
}
</script>
```

</details>

---

## Adding the Snake Game

Once the test webpage worked, I moved to the final game.

I found a **basic Snake game online** and used it as the starting point rather than writing the whole game from scratch.

**Original Snake game source:**  
[ADD THE ACTUAL LINK OR NAME OF THE SNAKE GAME HERE]

The base game already included the main mechanics:

- the game board
- snake movement
- food
- scoring
- collision detection
- keyboard controls

My work was understanding how its movement system worked and connecting my Arduino/Web Serial input to it.

<!-- PHOTO: Put the wider photo showing the laptop, Arduino, joystick, and Snake page here. -->

**Figure 3.** The complete setup with the Arduino and joystick connected to the laptop running the Snake webpage.

---

## Understanding the Snake Movement

The Snake game uses an HTML canvas:

```html
<canvas id="gameCanvas" width="400" height="400"></canvas>
```

JavaScript gets access to the canvas with:

```javascript
const canvas =
  document.getElementById("gameCanvas");

const ctx =
  canvas.getContext("2d");
```

The game uses `ctx` to draw the background, snake, food, and game-over screen.

The snake itself is stored as an array:

```javascript
snake = [
  { x: 10, y: 10 },
  { x: 9, y: 10 },
  { x: 8, y: 10 }
];
```

Each object is one section of the snake. `snake[0]` is the head.

The game keeps track of two direction variables:

```javascript
let direction;
let nextDirection;
```

`nextDirection` stores the newest input. During the next game update:

```javascript
direction = nextDirection;
```

The snake's head then moves one grid square.

For example:

```javascript
if (direction === "left") {
  head.x--;
}
```

Right adds to X, up subtracts from Y, and down adds to Y.

---

## Connecting the Joystick to Snake

This was the main change I made to the Snake game.

Before this, my Web Serial code could receive `"L"` and display LEFT:

```javascript
if (line === "L") {
  directionText.textContent = "LEFT";
}
```

I needed the same Serial command to control the game's movement.

I created a function that handles Arduino commands:

```javascript
function handleArduinoDirection(command) {

  if (command === "L") {
    directionText.textContent = "LEFT";
    changeDirection("left");
  }

  else if (command === "R") {
    directionText.textContent = "RIGHT";
    changeDirection("right");
  }

  else if (command === "U") {
    directionText.textContent = "UP";
    changeDirection("up");
  }

  else if (command === "D") {
    directionText.textContent = "DOWN";
    changeDirection("down");
  }
}
```

The important change is:

```javascript
changeDirection("left");
```

Before, `"L"` only changed text on the screen. Now `"L"` is passed into the same direction system used by the Snake game.

The final path for one joystick movement is:

```text
Move joystick left
↓
Arduino reads X value
↓
Arduino sends "L"
↓
Web Serial reads "L"
↓
handleArduinoDirection("L")
↓
changeDirection("left")
↓
nextDirection becomes "left"
↓
Game updates
↓
Snake head moves left
```

That connection between the hardware input and the existing game controls was the main coding change I made to the base Snake game.

---

## Why Center Does Not Stop the Snake

The Arduino sends:

```text
C
```

when the joystick returns to its center range.

The Snake code does not have a command for `C`.

I left it this way on purpose.

Snake should continue moving after I release the joystick. Returning the joystick to center should not stop the game.

For example:

```text
Joystick moved UP
↓
Arduino sends U
↓
Snake starts moving up
↓
Joystick released
↓
Arduino sends C
↓
C is ignored
↓
Snake keeps moving up
```

The snake only changes direction when the webpage receives L, R, U, or D.

---

## Preventing the Snake From Reversing

The game also checks whether a requested move is allowed.

For example, if the snake is moving right, it cannot instantly turn left:

```javascript
if (
  newDirection === "left" &&
  direction === "right"
) {
  return;
}
```

`return` stops the direction change.

The same check exists for every opposite direction.

This means the Arduino is responsible for telling the game which direction I moved the joystick, while the Snake code is still responsible for deciding whether that move is legal.

---

## Game Speed

The snake moves repeatedly using:

```javascript
gameLoop = setInterval(
  updateGame,
  130
);
```

The `130` is the delay in milliseconds between movements.

A smaller number makes the game faster. A larger number makes it slower.

For example:

```text
100 = faster
130 = current speed
180 = slower
```

This matters more with a physical controller because I need enough time to move the joystick before the next game update.

---

## Keeping the Keyboard Controls

I kept the original arrow-key controls even after the joystick worked.

They were useful for debugging.

If the keyboard worked but the joystick did not, I knew the Snake game itself was still working. I could then focus on the joystick, Arduino, Serial data, or Web Serial connection.

If both the keyboard and joystick failed, the problem was more likely inside the game code.

Keeping the keyboard input gave me a second way to test the movement system.

---

## Final Test

Once the Web Serial code and Snake code were combined, the physical joystick controlled the game.

<!-- VIDEO: Put the video of you controlling Snake with the joystick here. -->

<video width="700" controls style="max-width: 100%;">
  <source src="videos/joystick-snake-final.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

<p><strong>Video 2.</strong> Using the physical joystick to control the Snake game.</p>

The completed system is:

```text
Physical joystick
↓
Analog X and Y readings
↓
Arduino direction logic
↓
L / R / U / D
↓
USB Serial
↓
Web Serial API
↓
JavaScript
↓
changeDirection()
↓
Snake movement
```

The project started with a joystick printing letters in Serial Monitor. By the end, those same letters were controlling movement inside a game running on my GitHub website.

---
## Final Arduino Code

```cpp
const int xPin = A0;
const int yPin = A1;
const int swPin = 2;

int toggleState = 0;

int buttonState = HIGH;
int lastButtonState = HIGH;

unsigned long lastDebounceTime = 0;
unsigned long debounceDelay = 50;

void setup() {
  Serial.begin(9600);

  // Joystick button is connected to GND when pressed
  pinMode(swPin, INPUT_PULLUP);
}

void loop() {

  // Read joystick position
  int xValue = analogRead(xPin);
  int yValue = analogRead(yPin);

  // Send joystick direction
  if (xValue < 400) {
    Serial.println("R");
  }
  else if (xValue > 600) {
    Serial.println("L");
  }
  else if (yValue < 400) {
    Serial.println("D");
  }
  else if (yValue > 600) {
    Serial.println("U");
  }
  else {
    Serial.println("C");
  }

  // Read joystick button
  int reading = digitalRead(swPin);

  // Debounce
  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {

    if (reading != buttonState) {
      buttonState = reading;

      // LOW means the joystick button was pressed
      if (buttonState == LOW) {

        toggleState = !toggleState;

        Serial.print("BUTTON: ");
        Serial.println(toggleState);
      }
    }
  }

  lastButtonState = reading;

  // Slow down Serial output
  delay(100);
}
```

## Final Snake Game

The final Snake game uses the Arduino joystick as the controller through Web Serial.

[Open the Arduino Joystick Snake Game](snake.html)
## Final Snake Game

The final Snake game uses the Arduino joystick as the controller through Web Serial.

[Open the final Arduino Joystick Snake Game](snake.html)



## Reflection

The skill I relied on most was **debugging**. I tested the project in separate stages instead of connecting everything at once. I first checked the joystick, then Serial output, then Web Serial, and finally Snake.

That made problems easier to locate. I could tell whether an issue came from the physical input, Arduino code, serial connection, or game code.

If I continued the project, I would build a case around the Arduino and joystick so it works more like a real controller and the wires are protected.
