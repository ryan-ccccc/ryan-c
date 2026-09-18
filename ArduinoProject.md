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

# Arduino Joystick Controller

## Project Idea

For this project, I chose to build on **analog input**. In class, we used a potentiometer to see how Arduino can read changing values. I wanted to use analog input in a more interactive way, so I chose a joystick.

My current goal is to use the joystick to control something on a webpage. Before building the final game, I first needed to make sure the Arduino could correctly read the joystick and send its directions to the website.

The system currently works like this:

**Joystick movement → Arduino reads the values → Arduino finds the direction → Arduino sends the direction through USB → webpage reads and displays the direction**

## New Component: Joystick

The new component I chose is a joystick module.

The joystick has five pins:

* **VCC** → power
* **GND** → ground
* **VRx** → left and right movement
* **VRy** → up and down movement
* **SW** → button when the joystick is pressed down

I connected VRx to **A0** and VRy to **A1** on the Arduino.

The joystick uses analog input because its position is not just on or off. The Arduino reads a range of values depending on how far and which direction I move it.

## Stage 1: Reading the Joystick

I first tested the joystick using the Arduino Serial Monitor.

I used:

```cpp
int xValue = analogRead(A0);
int yValue = analogRead(A1);
```

`analogRead()` gives a value from about **0 to 1023**. When the joystick is near the center, the value is usually around **512**.

I used values below 300 and above 700 to decide when the joystick was moved far enough in a direction.

The Serial Monitor could display:

```text
LEFT
RIGHT
UP
DOWN
CENTER
```

This helped me make sure the Arduino was correctly reading the joystick before I tried connecting it to a webpage.

<img width="1280" height="1707" alt="f1c29299bacbc1c1220f0f30186f55bc" src="https://github.com/user-attachments/assets/0d72f3d2-bd25-47a8-9a68-dfcf86f4f2ea" />

## Mistake 1: Left and Right Were Backwards

One of the first problems I found was that left and right were reversed.

When I moved the joystick left, the Serial Monitor printed:

```text
RIGHT
```

When I moved it right, it printed:

```text
LEFT
```

My original code was:

```cpp
if (xValue < 400) {
  Serial.println("LEFT");
}
else if (xValue > 600) {
  Serial.println("RIGHT");
}
```

After testing the joystick, I realized that its X-axis was oriented differently from what I expected.

I fixed it by switching the directions:

```cpp
if (xValue < 400) {
  Serial.println("RIGHT");
}
else if (xValue > 600) {
  Serial.println("LEFT");
}
```

After changing the code, the direction shown on the Serial Monitor matched the direction I actually moved the joystick.

## Mistake 2: The Center Was Not Exactly 512

At first, I thought I could use exactly **512** as the center of the joystick.

When I tested it, I noticed that the value changed slightly even when I was not touching the joystick. It might be close to 512, but it does not always stay exactly there.

Instead of using one exact value, I created a center range.

Values between about **400 and 600** count as the center. This stops small changes in the analog reading from being counted as movement.

## Stage 2: Sending Directions for the Website

After the joystick worked in the Serial Monitor, I changed the output so it would be easier for a webpage to read.

Instead of sending full words, the Arduino sends one letter for each direction:

```text
L = Left
R = Right
U = Up
D = Down
```

<img width="514" height="544" alt="Screenshot 2026-09-18 at 10 56 41 AM" src="https://github.com/user-attachments/assets/e409c03d-069f-4857-a38d-d4b3301af52f" />

My Arduino code checks the X and Y values and sends the correct letter.

For example:

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
```

I do not need to send a CENTER value because the webpage only needs to know when I move the joystick in a direction.

## Stage 3: Connecting the Arduino to GitHub

After the Arduino could send the directions correctly, I wanted to get that information onto my GitHub website.

I created a separate HTML page in my GitHub repository. The page uses JavaScript and the **Web Serial API** to communicate with the Arduino through the USB cable.

I first added a **Connect Arduino** button to the webpage. The website cannot automatically connect to any USB device, so I have to click the button and choose the Arduino from the list of available serial ports.

The Arduino code starts Serial communication with:

```cpp
Serial.begin(9600);
```

Because the Arduino uses a baud rate of 9600, I also had to set the website connection to the same baud rate:

```javascript
await port.open({
  baudRate: 9600
});
```

If the two sides used different speeds, the information would not be read correctly.

After the connection opens, the JavaScript continuously waits for information coming from the Arduino. The Arduino sends letters such as:

```text
L
R
U
D
```

The JavaScript reads the incoming data and checks which letter was sent.

For example:

```javascript
if (line === "L") {
  directionText.textContent = "LEFT";
}
else if (line === "R") {
  directionText.textContent = "RIGHT";
}
```

This means the Arduino does not directly control the webpage. Instead, the Arduino sends information through the USB cable, and the JavaScript decides what to do with that information.

The full connection works like this:

```text
I move the joystick
        ↓
Arduino reads A0 and A1
        ↓
Arduino decides the direction
        ↓
Arduino sends L, R, U, or D
        ↓
USB sends the Serial data to my computer
        ↓
Web Serial API reads the data
        ↓
JavaScript checks the letter
        ↓
GitHub webpage displays the direction
```

I tested the connection by opening my GitHub webpage, pressing **Connect Arduino**, and selecting my Arduino. After it connected, moving the physical joystick changed the text on the webpage between:

```text
LEFT
RIGHT
UP
DOWN
```

This was an important part of the project because I was able to connect hardware that I built on a breadboard to code running on a webpage.

## Mistake 3: The Website Would Not Connect While Serial Monitor Was Open

I ran into another problem when I tried connecting the Arduino to the webpage.

I still had the Arduino Serial Monitor open because I had been using it to test the joystick. When I pressed the **Connect Arduino** button on my website, the connection did not work correctly.

I realized that the Serial Monitor was already using the Arduino's serial connection. The website was trying to use the same connection at the same time.

To fix it, I closed the Serial Monitor before pressing the Connect Arduino button on the webpage.

My process became:

```text
1. Upload the Arduino code
2. Test it in Serial Monitor if needed
3. Close Serial Monitor
4. Open my GitHub webpage
5. Press Connect Arduino
6. Select the Arduino port
7. Move the joystick
```

After I closed the Serial Monitor, the webpage connected and started reading the joystick directions correctly.

This problem helped me understand that the Arduino's serial connection is being used by either the Serial Monitor or my webpage. I need to close one before I use the other.

