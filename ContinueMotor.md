<style>
  /* Scanner project theme */
  :root {
    --background: #0b1118;
    --panel: #121d28;
    --text: #dce6ef;
    --muted: #98acbd;
    --cyan: #63e6e2;
    --orange: #ffb36b;
    --border: #273b4c;
  }

  body {
    background-color: var(--background);
    background-image:
      linear-gradient(rgba(99, 230, 226, 0.035) 1px, transparent 1px),
      linear-gradient(90deg, rgba(99, 230, 226, 0.035) 1px, transparent 1px);
    background-size: 32px 32px;
    color: var(--text);
    font-family: system-ui, sans-serif;
    line-height: 1.8;
  }

  h1, h2, h3, h4 {
    color: var(--cyan);
    line-height: 1.3;
  }

  h2 {
    margin-top: 2.5rem;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--border);
  }

  h3 {
    margin-top: 2rem;
    color: var(--orange);
  }

  p, li {
    color: var(--text);
  }

  a {
    color: var(--cyan);
    text-decoration: underline;
    text-underline-offset: 4px;
  }

  a:hover {
    color: var(--orange);
  }

  strong {
    color: #ffffff;
  }

  img, video {
    display: block;
    max-width: 100%;
    height: auto;
    margin: 24px auto 12px;
    border: 1px solid var(--border);
    border-radius: 12px;
  }

  em {
    color: var(--muted);
  }

  blockquote {
    margin: 24px 0;
    padding: 16px 20px;
    background: var(--panel);
    border-left: 3px solid var(--orange);
    color: var(--text);
  }

  pre, code {
    background: var(--panel);
    color: var(--cyan);
    border-radius: 6px;
  }

  pre {
    padding: 18px;
    border: 1px solid var(--border);
    overflow-x: auto;
  }

  th, td {
    border-bottom: 1px solid var(--border);
    color: var(--text);
  }

  th {
    color: var(--cyan);
  }

  .scanner-banner {
    margin-bottom: 36px;
    padding: 30px;
    background: linear-gradient(135deg, #15303b, #121d28);
    border: 1px solid #315766;
    border-radius: 16px;
  }

  .scanner-label {
    color: var(--cyan);
    font-family: monospace;
    font-size: 12px;
    letter-spacing: 2px;
  }

  .scanner-banner h1 {
    margin: 12px 0;
    color: #ffffff;
    font-size: clamp(30px, 5vw, 44px);
  }

  .scanner-banner p {
    margin: 0;
    color: var(--muted);
  }
</style>

<div class="scanner-banner">
  <div class="scanner-label">TECH & INNOVATION / PROJECT JOURNAL</div>
  <h1>From Distance to Dimension</h1>
  <p>Restoring a 3D scanner and turning measurements into a visible shape.</p>
</div>

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
