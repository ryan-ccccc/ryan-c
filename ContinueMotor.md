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


## Day 1 — Restoring the 3D Scanner

### Context

Kaydon, Leo, and I are continuing the 3D scanner project that Kaydon and Thomas started last year. Our goal is to restore their scanner and develop software that uses its measurements to display the scanned object. This builds on my previous experience with Arduino inputs and connecting hardware to software, but introduces the challenge of turning distance readings into a representation of an object's shape.

[Kaydon's original Innovator's Journal](https://kfidak13.github.io/3d%20Scanner.html) provides background on the original project, including its testing, hardware, and problems.

### Understanding the Existing Project

The original project used a distance sensor mounted on a bracket attached to a stepper motor. The photos show an Arduino with a motor-driver shield, an external power supply, and the sensor assembly. The motor changes the direction the sensor faces so it can take measurements from different angles.

Kaydon's journal describes inconsistent distance readings and experiments with motor steps and delays. The previous team also found that securely mounting the sensor and keeping its angle consistent improved its behavior. These details give us specific things to investigate as we restore the scanner.

One useful piece of evidence is their graph comparing real-world distances with reported distances. The readings followed a trend, but the fitted line did not match perfect agreement. This means that getting a number from the sensor does not automatically mean the number is accurate. We will need to check the restored scanner's readings against known distances before relying on them to draw an object.

### Our Progress

Our first stage was restoring the existing scanner. We now have the hardware restored, shown in the photo below. However, having the assembly back together does not yet demonstrate that its readings are accurate or that it can generate a complete 3D scan.

<img width="569" height="427" alt="baee17b15a28f5b4b7bdb3fd0012b15c" src="https://github.com/user-attachments/assets/bd0cabc5-5767-4d07-8d95-598216c6d974" />

*Our restored scanner at the end of Day 1. This records the physical setup before further testing and development of the visualization software.*

We are building on Kaydon and Thomas's hardware and previous work rather than starting with a new scanner design. Our next task is to establish what the restored system can do reliably and then connect its measurements to software that displays them.

### How the Scanner Works

My current understanding is that the Arduino coordinates the motor and the distance sensor. The stepper motor moves the sensor through small increments. At each position, the sensor measures the distance to a surface. The program needs to associate each reading with the direction the sensor was facing.

The visualization software will then use the distance and angle together to calculate where a measured point belongs. Repeating this process produces a collection of points describing the surfaces the sensor can detect.

A single sweep only records one cross-section. To create a full 3D representation, we also need measurements at different heights or another controlled scanning angle. We still need to establish how our restored mechanism will collect those additional measurements. The old photos alone do not confirm that this part was completed.

### Testing and Goals for the Next Session

My next goal is to test the distance sensor with an object at several known distances and record repeated readings at each position. This will help us see whether errors are consistent or whether readings fluctuate while the object stays still.

We also need to test the motor separately and check that it moves in repeatable increments. After that, we can combine movement and measurement, checking whether a pause after each movement improves the readings. Changing one setting at a time will make it easier to identify what affects the results.

For the software, our first target is to display one sweep of measured points. Once that works, we can work toward collecting and displaying multiple layers. This gives us a smaller, testable goal before attempting a complete 3D object.

## Day 2 — Testing the Sensor and Creating the Scanning Software

### Getting Distance Readings

Today, Kaydon, Leo, and I got the sensor working with the Arduino code. We tested it using Kaydon's hand as the target and watched the reported distance change as he moved his hand closer to and farther from the scanner.

This showed that the sensor was responding to changes in distance. However, we have not yet compared those readings with a ruler, so this test confirmed that it responds rather than proving that its measurements are accurate.

<video controls playsinline preload="metadata"
       style="width: 100%; max-width: 600px; display: block; margin: 0 auto;">
  <source src="videos/scanner-distance-test-day2.mp4" type="video/mp4">
  Your browser does not support embedded video.
</video>

*Testing the sensor with Kaydon's hand. The Arduino code reports distance readings as he changes the distance between his hand and the scanner.*

### Using Claude to Create the Software

Once we had the sensor responding, we used Claude to create software intended to control the scanner and display its measurements in 3D. Claude generated the scanning software and Arduino firmware. Our role was to choose what we wanted the scanner to do and test whether the result works with our physical setup.

[Our scanner code repository](https://github.com/kfidak13/RLK)

The software includes controls to connect the Arduino, move the motors, adjust calibration, and start a scan. It also includes a 3D viewer and options to save and load scan data. We did not have enough time in class to test this software with the scanner, so we cannot yet say that it successfully produces a scan.

### How the Code Is Intended to Work

The browser connects to the Arduino through the Web Serial API. It sends movement commands so the sensor can scan across different horizontal and vertical positions. At each position, the Arduino pauses and collects several sensor readings. The firmware sorts those readings and selects a middle value to reduce the effect of unusually high or low readings.

The browser then combines the distance reading with the sensor's pan and tilt angles to calculate a point's position in 3D. Repeating this across the scanning area should build a point cloud, which is a collection of dots representing the surfaces detected by the sensor.

This helped me understand why a distance reading alone is not enough to create a 3D scan. The software also needs to know which direction the sensor was facing. If the angle calibration is wrong, the points could appear in the wrong places even when the distance readings are reasonable.

The current software is intended to display a point cloud. It does not yet create a solid, finished model that we could immediately 3D print.

### Understanding the Important Arduino Code

Claude created the firmware for the scanning software. This is the code we still need to test with the complete scanner, rather than the earlier code used to show the distance of Kaydon's hand.

#### Connecting the Code to the Hardware

```cpp
const int ENABLE_PIN = 8;
const int X_STEP_PIN = 2;
const int X_DIR_PIN = 5;
const int Y_STEP_PIN = 3;
const int Y_DIR_PIN = 6;
const int SENSOR_PIN = A3;
```

These constants identify the Arduino pins connected to the motor drivers and sensor. Each motor uses a STEP pin to tell it when to move and a DIR pin to choose the direction. The sensor sends its analog reading through A3. These assignments match the CNC shield's connections.

#### Receiving Commands from the Browser

```cpp
Serial.begin(115200);
Serial.println("ready");
```

The Arduino and browser communicate at 115200 baud. The Arduino sends `ready` when it starts so the browser knows the firmware is responding.

The firmware waits for a complete line of text and checks which command it received. For example, `M 3 0` requests three pan steps and no tilt movement, while `R` requests a sensor reading. This means the browser decides the scanning path, and the Arduino carries out each movement and measurement.

#### Moving the Motors

```cpp
void pulse(int stepPin) {
  digitalWrite(stepPin, HIGH);
  delayMicroseconds(2);
  digitalWrite(stepPin, LOW);
}
```

This function produces a short electrical pulse for a motor driver. The movement function repeats it for the requested number of steps, with a delay between pulses to control speed.

The code also tracks position using `panPos` and `tiltPos`. These values count the steps the program commanded. They do not measure the actual position, so if a motor jams or skips steps, the stored position could become incorrect.

#### Limiting the Tilt Movement

```cpp
const long newTilt = tiltPos + tilt;

if ((tilt > 0 && newTilt > tiltMax) ||
    (tilt < 0 && newTilt < tiltMin)) {
  Serial.println("err limit");
  return;
}
```

Before moving, the Arduino calculates where the tilt would end up. It refuses movement farther beyond the configured limits and sends `err limit` to the browser.

This matters because the screw linkage has limited travel. However, these software limits depend on us choosing a correct reference position and measuring the available movement. The Arduino cannot detect the physical end of the linkage by itself.

#### Reducing Unusual Sensor Readings

```cpp
const int SETTLE_MS = 100;
const int READINGS = 10;
const int READING_GAP_MS = 40;
```

Before taking a measurement, the code waits 100 milliseconds to let the mechanism settle. It then collects ten readings, spaced 40 milliseconds apart, and sorts them from lowest to highest.

```cpp
Serial.print("d ");
Serial.println(readings[READINGS / 2]);
```

The Arduino sends the upper middle reading from that sorted group. This reduces the influence of an unusually high or low reading compared with taking a simple average. It can reduce sudden spikes, but it does not correct a sensor that consistently measures the wrong distance.

The value sent is still a raw sensor reading. The browser converts it into centimeters and combines it with the pan and tilt angles to calculate a 3D point. Understanding this separation helps us troubleshoot: we can check the raw readings first, then the distance conversion, and finally where the points appear.

### What We Have Confirmed So Far

Our hand test showed that the sensor and Arduino can produce changing distance readings. The new scanning software has been created, but we have not tested the complete process of moving the motors, collecting readings, and displaying the resulting points on our hardware.

The repository includes simulated tests, but those cannot establish whether our physical scanner moves accurately or whether its linkage introduces errors. That is why I would describe the software as ready for testing rather than say that the scanner is finished.

### Goals for Next Class

Next class, our main goal is to connect the scanner to the new software and attempt a small scan. We will first check which motor controls pan and which controls tilt, then establish the tilt movement limits and measure how much the sensor's angle changes per motor step.

Before attempting a complicated object, we plan to test a flat surface at a known distance. This will give us something clear to compare with the displayed points. We will also compare the sensor's distance readings with a ruler.

If the result looks wrong, we will check the distance readings and motor movement separately, then change one setting at a time and repeat the test. I want to record the first result even if it is incomplete, because it will help us explain what failed and whether later changes actually improved it.

## Day 3: Getting the Scanner Moving Again

On Day 2, we saw the distance reading change when Kaydon moved his hand in front of the sensor. Today, Kaydon, Leo, and I tested whether the motors could move the scanner. We used Arduino test code that Kaydon generated with Claude.

At first, only one motor responded. Since we were picking up a build from last year, we needed to check which parts still worked. We also ran into trouble with the power supply. One thing I learned is that the Arduino can send sensor readings over USB even when the motors aren't getting the power they need. That gave us a reason to check the motor supply, although we couldn't say for sure that it caused the unresponsive motor.

By the end of class, we had both motors responding and the scanner moving again.

<video controls playsinline preload="metadata" style="display: block; width: 100%; max-width: 600px; margin: 0 auto;">
  <source src="videos/scanner-movement-day3.mp4" type="video/mp4">
</video>

*The restored scanner moving during our Arduino motor test.*

### How Our Motor Test Worked

Kaydon used Claude to generate the Arduino sketch we ran. The goal was to see whether both motors could move and reverse. This sketch runs on the Arduino by itself. It doesn't read the distance sensor or need the scanning software, so we could test the movement separately.

In `setup()`, the Arduino sets the motor control pins to `OUTPUT` and writes `LOW` to `ENABLE_PIN` (pin 8). That line stood out to me because `LOW` actually enables the drivers on our CNC shield. It gives them permission to move the motors, but they still need power from the motor supply.

The part that explains the movement in our video is the loop:

```cpp
void loop() {
  moveBoth(HIGH);
  moveBoth(LOW);
}
```

Here, `HIGH` and `LOW` are direction settings. Inside `moveBoth()`, the code sets the direction for both motors, then pulses both `STEP` pins 100 times. Each pulse tells each driver to move its motor one step. The 10,000-microsecond delay spaces the pulses about 10 milliseconds apart, so each movement takes roughly one second. The scanner pauses for half a second, reverses for 100 steps, and pauses again. That pattern repeats while the sketch is running.

One detail I learned is that 100 full steps is half a turn of our motor shaft, but it doesn't mean the sensor itself tilts 180 degrees. The screw linkage turns the motor's rotation into a smaller tilt movement and has limited travel. The sketch always sends the number of steps set in `STEPS`; it has no way to detect when the linkage reaches its limit.

This also helped me understand our troubleshooting. The sketch sends step commands to both drivers during the same loop. When only one motor moved, we knew the loop was running, but we couldn't tell from the code alone why the other motor stayed still. Seeing both motors move and reverse gave us evidence that both responded to the test. Checking whether the software can use those movements to build a scan is still a separate task.

### What comes next

We now have evidence that the sensor can detect a change in distance and that the motors can move the scanner. We still need to find out whether the software can match each reading to the direction the sensor was facing and plot it in the right place.

One option for next class is to scan something and see if the software works or not. The other is to improve the original design, starting with a base that can hold the scanner steady on its own. We will decide what direction to move towards next class.

## Day 4: Getting the Scanner Working

### From testing the motors to scanning

Today, we got the scanner movement and distance readings working together. The previous code was only a test to check whether the motors and their connections worked. It moved the motors back and forth, but it did not scan an object.

The new code actually performs a scan. X moves across a horizontal row while the sensor measures distance. After each row, Y moves upward by 10 microsteps, and X scans back in the opposite direction.

### Adjusting the movement

We tested the Y movement and chose the range from −90 to 360 microsteps because it worked best for our mechanism. This gives 450 microsteps of travel. With 10 microsteps between rows, the scanner takes 46 rows, including the starting row.

We also changed the X sweep from 90° to 45°. This required changing the movement count from 800 to 400 microsteps and updating the angle calculation displayed in the Serial Monitor. The Y values are only motor positions. We have not measured the actual tilt angle produced by the screw linkage.

### Fixing the sensor code

At first, the Serial Monitor kept displaying “No echo detected.” The AI-generated code assumed we were using an ultrasonic sensor, but our actual sensor was the Sharp GP2Y0A21YK0F infrared sensor. It was connected to CoolEn, 5V, and GND.

After we supplied the correct sensor model, the code was changed to read an analog voltage through A3 instead of waiting for an ultrasonic echo. This fixed the reading method, and we were able to get distance measurements in the Serial Monitor.

### How I understood the code

The pin settings at the beginning tell the Arduino where the motors and sensor are connected. Each motor has a direction pin and a step pin. The direction pin controls which way it turns, and pulses sent to the step pin control how far it moves. With our driver settings, each pulse represents one microstep.

The `moveAxis()` function sends these pulses and counts the commanded movement. I understood that this count does not prove the motor reached that position. If the motor gets stuck, the code will still count the pulses because there is no position sensor checking it.

The `runScan()` function controls the scanning sequence. X moves 16 microsteps between readings, pauses for a measurement, and continues until it finishes the row. Y then moves upward by 10 microsteps. X changes direction for the next row, creating a back-and-forth scanning pattern.

The `printDistance()` function takes five analog readings and averages them. It converts the average into a voltage, then uses an approximate formula to calculate distance in centimetres. These measurements still need to be checked against a ruler.

I also understood how to adjust the scanning range. Assuming a 200-step motor with 1/16 microstepping, one full rotation takes 3,200 microsteps. A 45° rotation is one eighth of a turn, so we set the X travel to 400 microsteps.

### Code That Controls the Scan

This is the section of our working code that controls the scanning sequence. The earlier code only tested whether the motors worked. This code moves X across each row and calls `recordPoint()` to take distance readings. After each row, it moves Y upward and reverses X for the next row. It uses the movement and sensor-reading functions defined elsewhere in the full program.

These settings give us a 45° X sweep and 10 microsteps of upward Y movement between rows:

```cpp
const long X_SWEEP_STEPS = 400;
const int X_SAMPLE_STEPS = 16;
const int Y_TRAVEL_STEPS = 450;
const int Y_ROW_STEPS = 10;
const int ROW_COUNT = Y_TRAVEL_STEPS / Y_ROW_STEPS + 1;
```

The function below uses those settings to complete the scan:

```cpp
void runScan() {
  Serial.println("Scan started.");

  for (int row = 0; row < ROW_COUNT; row++) {
    // Alternate X direction for each row.
    int direction = (row % 2 == 0) ? 1 : -1;

    // Take a reading at the beginning of the row.
    if (!recordPoint(row)) return;

    long remaining = X_SWEEP_STEPS;

    while (remaining > 0) {
      long amount = X_SAMPLE_STEPS;
      if (amount > remaining) amount = remaining;

      // Move X to the next measurement position.
      if (!moveAxis(
            X_STEP_PIN, X_DIR_PIN, X_FORWARD,
            direction, amount, xPosition)) {
        return;
      }

      remaining -= amount;

      // Read the sensor and print the measurement.
      if (!recordPoint(row)) return;
    }

    // Raise Y if another row remains.
    if (row < ROW_COUNT - 1) {
      if (!moveAxis(
            Y_STEP_PIN, Y_DIR_PIN, Y_UP,
            1, Y_ROW_STEPS, yPosition)) {
        return;
      }
    }
  }

  Serial.println("Scan complete: 46 rows.");
}
```

I understood that the outer `for` loop controls the rows, while the inner `while` loop controls movement across each row. `remaining` tracks how many X microsteps are left before reaching the end. The direction switches between `1` and `-1` on alternating rows. The final condition prevents Y from moving upward again after the last row.

<video controls playsinline preload="metadata" style="display: block; width: 100%; max-width: 600px; margin: 0 auto;">
  <source src="videos/scanner-working.mp4" type="video/mp4">
</video>

### Using AI and testing the result

We used ChatGPT to help turn the motor test into scanning code. We provided the pin connections, tested movement range, and the number of microsteps we wanted between rows. We tested the code on the hardware and corrected the sensor assumption when the distance readings did not work.

The commands made testing easier: `t` checks the distance without moving, `s` starts the scan, and `q` stops it. We still need to position the scanner at its starting point before running it because it cannot automatically find that position.

### Next class

Next class, our goal is to test the software Kaydon created earlier. Now that the scanner movement and distance readings work, we need to check whether his software can receive and use the data from this code.
