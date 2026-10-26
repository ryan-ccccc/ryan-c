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

[Play my mini-golf game that needs to be connected to a joystick to play](mini.html) 


## Connecting My Mini-Golf Game to an Arduino Controller

### Starting Point and Goal

For this part of the project, I continued working on the mini-golf game I created earlier. The game already had nine holes, obstacles, water hazards, scoring, and result animations. My new goal was to control its aiming, power, and putting with a physical joystick and button connected to an Arduino.

This connected the software project to the circuits and Arduino work from the previous unit. I wanted to understand how moving a physical component could change something inside a browser game. I kept the existing courses and game rules so I could focus on the controller connection.

<a href="{{ '/mini.html' | relative_url }}">Play my Arduino-controlled mini-golf game</a>

### Setting Up the Joystick and Button

My setup used a joystick with two analog outputs and a built-in button, plus a separate pushbutton. The joystick’s X and Y readings describe its position. The buttons provide pressed or released states.

The code used these connections:

| Input | Arduino pin | Purpose |
|---|---|---|
| Joystick X | A2 | Horizontal joystick position |
| Joystick Y | A0 | Vertical joystick position |
| Joystick button | D2 | Cancel the direction lock and return to aiming |
| Separate pushbutton | D3 | Lock direction, then confirm the shot |

The joystick button uses `INPUT_PULLUP`, so LOW means pressed. The separate button uses the original HIGH-means-pressed arrangement. That input needs a pull-down resistor to keep it LOW when the button is released instead of leaving the reading unstable.

<img width="480" height="384" alt="4a4121cb61985a3d8f5cfccd81dd7281" src="https://github.com/user-attachments/assets/bfaead4f-a3bb-4814-93bc-4b68724d9200" />


<img width="426" height="569" alt="c0cdd63692e6629682a39713204a9781" src="https://github.com/user-attachments/assets/de29c708-cea7-40a9-93b5-d241bddc2c68" />


### Testing the Inputs Before Connecting the Game

I started with a test sketch that printed directions such as UP, DOWN, LEFT, RIGHT, and CENTER, along with the state of each button. It used values below 400 or above 600 to identify joystick movement.

This separated the input test from the game. Seeing the readings change showed that information was being produced, but it did not yet prove that the game could use it. That distinction became important when I later connected the Arduino successfully but still could not move the aiming arrow.

### Sending the Readings to the Browser

I asked ChatGPT to connect the controller to the existing game using the Web Serial API. ChatGPT revised the Arduino sketch and added the browser connection code.

The revised sketch sends raw X and Y readings instead of only direction words. An example message is:

```text
GOLF,520,505,0,1
```

In this example, X is 520, Y is 505, the joystick button is released, and the separate button is pressed. The numbers are an example of the message format, not a recorded measurement from my test.

The sketch sends a line every 40 milliseconds. Both sides use a baud rate of 9600:

```cpp
Serial.begin(9600);
```

```javascript
await port.open({ baudRate: 9600 });
```

The browser collects incoming text until it has a complete line, then separates the readings. This matters because one serial read does not always contain exactly one complete message.

The connection happens between the Arduino and the browser on my laptop. GitHub Pages hosts the game file. The browser runs that file and reads the controller through USB.

### Understanding the Important Parts of the Arduino Code

ChatGPT revised the Arduino sketch, so I wanted to understand how it reads the controller and prepares information for the game.

The first section assigns names to the input pins:

```cpp
const int xPin = A2;
const int yPin = A0;
const int joystickButtonPin = 2;
const int pushButtonPin = 3;
```

These names connect the code to the physical wiring. For example, `analogRead(xPin)` reads A2. Using a name makes the purpose clearer than repeatedly writing the pin number.

**Handling the Two Buttons**

The buttons use different electrical states to mean pressed. The joystick button uses `INPUT_PULLUP`, so pressing it produces LOW. The separate button uses the existing pull-down arrangement, so pressing it produces HIGH.

```cpp
const bool PUSH_BUTTON_ACTIVE_HIGH = true;
```

This setting tells the code how to interpret the separate button. It must match the wiring.

The `DebouncedButton` structure groups the information needed to track one button. Its main values are:

| Value | What it remembers |
|---|---|
| `pin` | Which input to read |
| `activeHigh` | Whether HIGH means pressed |
| `lastRaw` | The most recent unfiltered reading |
| `pressed` | The accepted, stable button state |
| `changedAt` | When the raw reading last changed |

The code creates two separate button objects:

```cpp
DebouncedButton joystickButton;
DebouncedButton puttButton;
```

They use the same logic but remember their states independently.

**Preventing Unstable Button Readings**

A physical button can briefly switch between HIGH and LOW as its contacts settle. The `update()` function checks for changes and records their time:

```cpp
if (raw != lastRaw) {
  lastRaw = raw;
  changedAt = now;
}

if (now - changedAt >= DEBOUNCE_MS) {
  pressed = lastRaw;
}
```

Since `DEBOUNCE_MS` is 25, the reading must remain unchanged for at least 25 milliseconds before becoming the accepted state. This filters short fluctuations. It is separate from the browser’s new-press detection, which prevents holding the button from repeatedly firing shots.

**Starting Communication and Controlling Timing**

`setup()` starts serial communication and initializes the buttons:

```cpp
Serial.begin(9600);
joystickButton.begin(joystickButtonPin, false);
puttButton.begin(pushButtonPin, PUSH_BUTTON_ACTIVE_HIGH);
```

The `false` argument means the joystick button is active LOW. The separate button uses the active-HIGH setting.

Inside `loop()`, `millis()` provides the elapsed time since the Arduino started:

```cpp
unsigned long now = millis();
joystickButton.update(now);
puttButton.update(now);
```

The code keeps checking the buttons, but only sends a message when 40 milliseconds have passed:

```cpp
if (now - lastSend >= SEND_INTERVAL_MS) {
  lastSend = now;
```

This produces approximately 25 messages per second. Unlike the original `delay(200)`, this timing check does not deliberately pause the whole loop between messages.

**Reading and Sending the Controller State**

The joystick readings come from:

```cpp
int x = analogRead(xPin);
int y = analogRead(yPin);
```

The sketch sends these values along with the two stable button states. In this expression:

```cpp
joystickButton.pressed ? 1 : 0
```

the `? :` operator chooses `1` when the button is pressed and `0` when it is released.

`Serial.print()` builds the message piece by piece. The final `Serial.println()` adds a line ending, marking the end of one complete reading:

```text
GOLF,512,512,0,0
```

This gives the browser a consistent format to interpret.

### How Web Serial Connects the HTML Game

The Web Serial code is JavaScript inside the HTML file. Clicking **Connect Arduino** runs:

```javascript
port = await navigator.serial.requestPort();
await port.open({ baudRate: 9600 });
```

`requestPort()` asks me to select the Arduino’s serial port. `open()` starts the connection at the same baud rate as the sketch. `await` lets the code wait for these operations without stopping the browser’s other work.

The game then reads incoming bytes:

```javascript
const { value, done } = await session.reader.read();
```

A `TextDecoder` converts those bytes into text. The code keeps unfinished text in a buffer until it finds a newline. This prevents part of a message from being mistaken for a complete reading.

`receiveControllerLine()` checks the message format, extracts X, Y, and both buttons, and updates the stored controller state. The game then uses those values for aiming, adjusting power, and detecting confirmation presses.

For example, this comparison detects a new press of the separate button:

```javascript
const puttEdge =
  fresh &&
  putt === 1 &&
  controller.putt === 0;
```

It checks that the previous reading was released and the new reading is pressed. The first press locks direction in AIM mode. A later press in POWER mode calls `shoot()`. Repeated messages saying “pressed” do not count as additional presses.

The connection code also checks whether readings are recent. If no valid message arrives for more than 700 milliseconds, the game stops using that old input. Disconnecting cancels the reader, releases its lock, and closes the port.

Understanding this helped me separate the jobs of the two programs. The Arduino measures and reports the physical controls. The browser decides what those readings mean for the current step of the game.

### Choosing How to Aim and Set Power

I considered making the joystick control direction and power at the same time, with greater distance from the center producing more power. I also suggested a two-step system where I could choose direction first and power afterward.

I chose the two-step system because changing strength would be less likely to disturb my aim:

1. Tilt the joystick toward the target.
2. Press the separate button to lock the direction.
3. Return the joystick to center, then tilt up or down to adjust power.
4. Release and press the separate button again to putt.

Clicking the joystick’s built-in button cancels the lock and returns to aiming.

The game stores the current step in `controller.mode`. When I lock the direction, it saves the angle:

```javascript
controller.lockedAngle = aimAngle;
controller.mode = 'POWER';
```

While I adjust power, the saved angle stays unchanged. The same separate button therefore performs different actions depending on the current mode.

### Troubleshooting: Readings Worked, but the Game Did Not

The first connection was only partly successful. The Arduino connected, and the displayed information changed when I moved the joystick, but the aiming arrow did not move. Pressing the button also produced no response in the game.

I reported those specific symptoms to ChatGPT. This was more useful than my earlier “it doesnt work” message during the Tag experiment because it identified which part was responding and which part was not.

ChatGPT found two conditions in the browser code that could block gameplay even while readings were displayed. The game required manual calibration, and it could wait for both buttons to read released before allowing aiming. I did not isolate which condition caused the problem on my setup, but they were both revised.

The updated version automatically calculates the joystick’s center from 20 stable readings. It also includes a **Set center now** button as a manual option. A small dead zone around the center stops minor fluctuations from changing the aim.

The button-release requirement no longer blocks ordinary aiming. The game still detects a new button press instead of treating every message with a pressed button as another action. The Arduino also uses a 25-millisecond debounce period to filter brief changes caused by the button contacts.

I requested that mouse and keyboard shot controls be removed and that the game become single-player only. This made the controller test clearer. The joystick and separate button now handle every shot, while the mouse is used for connection, calibration, and round setup.

The changing readings helped narrow down the problem. They showed that input data was reaching the browser, so the USB connection was not completely failing. The next step was to investigate how the game used those readings. I reported that the displayed values changed while the aiming arrow and shot controls did not respond, which gave ChatGPT specific behavior to investigate.

ChatGPT identified calibration and button-release conditions that could block the controls and revised them. I then tested the updated game with the physical joystick and button and recorded the working result. Because several changes were made together, I cannot say which condition caused the original failure. If I troubleshoot this again, I would change and test one condition at a time to identify the cause more precisely.

### Understanding How Movement Reaches the Game

I can now follow the connection through the main parts of the program:

**Joystick movement → Arduino readings → USB serial message → browser state → game update → drawing**

For aiming, the browser subtracts the calibrated center from the current readings. It then converts that displacement into an angle:

```javascript
aimAngle = Math.atan2(v.y, v.x);
```

When I confirm a shot in POWER mode, the controller code calls the existing `shoot()` function. That function sets the ball’s velocity and adds a stroke. The game loop continues calling `update()` to move the ball and check collisions, while `draw()` displays the result.

This helped me understand that the controller supplies input to the game’s existing systems. It does not need a separate version of the movement, collision, or scoring code.

### Working Result

After the revisions, I confirmed that the joystick and button worked with the actual game. I recorded one demonstration of the controller in use.

<video controls playsinline preload="metadata"
       style="width: 100%; max-width: 600px; display: block; margin: 0 auto;">
  <source src="videos/minigolf-joystick-demo.mp4" type="video/mp4">
  Your browser does not support embedded video.
</video>

The current result is the original nine-hole game adapted for a single player using the Arduino controller. The successful physical test provides evidence beyond the simulated checks ChatGPT ran while revising the code.

### AI Use and Reflection

ChatGPT revised the Arduino sketch and implemented the Web Serial connection and controller logic. I provided the starting sketch and pin assignments, proposed the control options, reported the problem with the unresponsive game, and tested the updated version. I also chose to remove mouse and keyboard shot controls and simplify the game to one player.

The main thing I learned was that receiving information and using it are separate steps. Changing live readings showed that data was reaching the page, but the game’s conditions still had to allow that data to affect aiming and shooting. Describing that difference helped narrow down the problem.
