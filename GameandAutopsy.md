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

# Arcade Unit: First Experiments with Browser Games

## Starting Point

For this unit, I started using WebbGPT to generate browser games and explain their code. My goal was to begin understanding how a game’s graphical user interface, or GUI, connects to the code behind it. Pong and Tag were starting experiments, not my final project. I will probably work on a different game after getting more familiar with the basic structure.

## Creating Pong

My first prompt to WebbGPT was:

> make me a game i can play on my device laptop, make the game pong

WebbGPT generated `pong.html`, which included a computer opponent, a two-player mode, scoring, and pause and restart buttons. My prompt explained the game I wanted, but it did not ask for simple code or limit the number of features.

After receiving the game, I asked:

> can you explain this code to me?

WebbGPT explained how HTML, CSS, and JavaScript worked together in the file. It also explained the variables, controls, game loop, collisions, and scoring. This gave me a starting point for understanding how the code produced the game.

<img width="838" height="925" alt="Screenshot 2026-10-13 at 7 14 48 PM" src="https://github.com/user-attachments/assets/ee79c4a0-7b16-4959-a3fc-03d34fa1c3f1" />

## Trying Tag and Fixing a Problem

Next, I asked:

> now make me a tag game

WebbGPT generated `tag.html` with two modes. Survive mode involved avoiding taggers, while You're It mode involved chasing runners within a time limit. It also added a safe zone and saved high scores.

The first version did not work, so I sent:

> it doesnt work

WebbGPT suggested that `localStorage`, which saves high scores in the browser, might be causing an error. It provided a revised file, and the revised game worked when I tried it.

The revised code used `try/catch` around the storage commands. This allowed the game to handle a storage error instead of letting it stop the script. WebbGPT also added temporary storage as a backup. If this backup was used, scores would remain available while the page stayed open but would disappear after reloading or closing it.

It also added an error handler intended to show an error message and line number on the canvas. This could make another problem easier to investigate. The revision fixed the issue I experienced, although I did not collect an error message to confirm that `localStorage` was the original cause.

<img width="826" height="809" alt="Screenshot 2026-10-13 at 7 15 40 PM" src="https://github.com/user-attachments/assets/129d98ae-8c44-45ae-b9a8-0e838f81f862" />

<img width="793" height="678" alt="Screenshot 2026-10-13 at 7 16 01 PM" src="https://github.com/user-attachments/assets/0c7572b8-71f7-448f-82f4-ce023439bf5b" />

### How I Could Have Asked More Clearly

Although WebbGPT fixed the game, my message did not give it much information. I could have used a more specific prompt like this:

> The Tag game you generated does not work when I try to play it. Before changing the code, ask me which browser I am using, how I opened the file, what appears on screen, and whether the buttons and keyboard controls respond. Show me how to check the browser console for errors. Use that information to suggest a fix, explain the code you changed, and tell me how to test it.

This would be better because it asks WebbGPT to gather evidence before choosing a cause. It would also help me understand the change rather than only receiving a replacement file.

For example, a blank canvas and a visible game that does not respond to movement keys are different problems. A blank canvas could mean an error stopped the game from drawing, while unresponsive movement could involve the keyboard listener or movement code. Describing which situation happened would help narrow down what to inspect. These are examples of useful details, not a record of the exact symptoms I saw.

## Initial Code Autopsy

I used Pong for these first observations because its code has fewer systems to follow than Tag. I am identifying the main sections without trying to understand every line yet.

### The Canvas

The canvas appears inside the HTML `<body>`:

`<canvas id="gameCanvas" width="800" height="500"></canvas>`

It creates an 800-by-500-pixel drawing area. JavaScript finds it using `document.getElementById('gameCanvas')` and uses its 2D drawing context to display the paddles, ball, and score.

If I changed its width or height in the HTML and reloaded the page, I would expect the playing area to change. The code stores these dimensions as `W` and `H`, so positions calculated from them would also change. The paddles and ball have separate size settings, so they would not automatically become larger.

### State Variables

Near the beginning of the `<script>` section, the code declares variables such as `leftScore`, `rightScore`, `paused`, `gameOver`, and `twoPlayerMode`. These store information about the current game.

The paddle and ball objects contain their positions, sizes, and speeds. `initGame()` sets up the paddles and resets the scores. `resetBall()` places the ball near the center and gives it its starting movement values.

### The Game Loop

The `loop()` function repeatedly calls `update()` and `draw()`.

`update()` changes positions and checks collisions and scoring. `draw()` paints the background and draws the game using the current values. This separation helps me understand the difference between changing a position in the code and displaying that position on screen.

`requestAnimationFrame(loop)` schedules the next frame. It follows the browser’s rendering schedule, often around 60 frames per second, rather than guaranteeing an exact rate.

### Input Handling

Pong uses W and S to control the left paddle. In two-player mode, the up and down arrow keys control the right paddle.

The `keydown` event listener sets `keys[e.key] = true`, while `keyup` sets it to `false`. These listeners record which keys are held. The movement code inside `update()` then checks those values and changes the paddle’s position.

For example, `leftPaddle.y -= leftPaddle.speed` moves the left paddle upward by reducing its vertical coordinate. The buttons have their own click listeners. Pause changes the `paused` value, and Restart calls `initGame()`.

### Collision and Win-Lose Logic

The game checks the ball’s position against the walls and paddles. For example, `ball.y <= 0` detects when the ball reaches the top. The code then reverses its vertical direction using `ball.speedY *= -1`.

The left-paddle collision includes the comparison `ball.x <= leftPaddle.x + leftPaddle.w`. Other comparisons check the ball’s vertical position and whether it is moving toward the paddle. The game needs these conditions together to decide whether a bounce should happen.

The winning condition is:

`leftScore >= WIN_SCORE || rightScore >= WIN_SCORE`

`WIN_SCORE` is set to `7`, and `||` means “or.” If either player reaches seven points, the game sets `gameOver = true` and displays the winner.

### Scoring

The scores are stored in `leftScore` and `rightScore`.

When `ball.x < 0`, the ball has passed the left side, and `rightScore++` adds one point to the right player. When `ball.x > W`, `leftScore++` adds one point to the left player. The drawing function displays those stored values on the canvas.

## Outcome and Understanding

I finished these starting experiments with Pong and a working revised version of Tag. WebbGPT generated the code, explained Pong, and made the changes that fixed Tag. My role was choosing the game ideas, asking for an explanation, trying Tag, and reporting the problem. I have not yet made my own deliberate code change.

I now have a clearer starting understanding of the three languages inside the files. HTML creates elements such as the canvas and buttons, CSS controls their appearance and layout, and JavaScript controls the game’s behavior and draws the moving objects.

I also noticed that an AI’s description does not always match its code. WebbGPT described Tag’s safe zone as temporary, but the code does not include a time limit for staying inside it. That is something I could test further rather than assuming the description is correct.

## Goals for This Week

1. **Understand how the GUI connects to the code.** In my chosen game, I will identify an HTML element, a CSS rule, and a JavaScript function and explain how each affects what I see or control.

2. **Make and test one change myself.** I will change a value such as player speed or object size, predict the result before running the game, and document what actually happens with before-and-after screenshots.

3. **Improve how I report problems.** Instead of only saying “it doesnt work,” I will describe the steps I took, what I expected, what happened, and any error message. I will ask WebbGPT to explain its suggested fix so I can connect the problem to the code.
