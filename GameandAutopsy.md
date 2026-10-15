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

# One-shot Game and Autopsy

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

2. **Create a game that is more advanced than Pong.** I will attempt to create a new game that has more controls than pong.

3. **Improve how I report problems.** Instead of only saying “it doesnt work,” I will describe the steps I took, what I expected, what happened, and any error message. I will ask ChatGPT to explain its suggested fix so I can connect the problem to the code.

## Developing My Own Game: Mini Golf

[Play my mini-golf game](minigolf.html)

### Choosing a Game and Starting Small

After the Pong and Tag experiments, I chose mini golf for my main game. It connected to my interest in sports and gave me specific mechanics to investigate: aiming, choosing power, moving the ball, hitting obstacles, and reaching a cup.

My first request to ChatGPT was:

> try the minigolf game and then explain to me how the main mechanics work

I wanted the explanation alongside the game because my goal was to understand what the code was doing. The first version contained one hole, one rectangular obstacle, a power slider, and a stroke counter. This gave me a smaller starting point for trying the controls and reading the main functions.

ChatGPT generated the file and explained the relationship between aiming, velocity, friction, collisions, and drawing. I then used the game as a starting point for deciding what needed to change.

<img width="1372" height="781" alt="Screenshot 2026-10-14 at 6 48 23 PM" src="https://github.com/user-attachments/assets/a770640f-7798-42e8-8afa-2395f7b362ca" />

*Caption: I started with one hole so I could investigate the main mechanics before expanding the game.*

### Connecting Input, State, Updates, and Rendering

The main relationship I am learning to follow is:

**Player input → changes to stored values → game updates → drawing the result**

In my game, pressing Space runs `shoot()`. This function reads the selected power and aiming angle, changes the ball’s velocity, adds one stroke, and sets `shotInProgress` to `true`.

The game loop then calls `update()` to move the balls, slow them down, and check collisions. After the updates, `draw()` displays the current positions on the canvas.

This explains why the ball’s movement involves more than one function. The keyboard listener detects the input. The shot function changes the state. The update function applies the movement. The drawing function makes that movement visible.

The important state values include:

| State | What it remembers |
| --- | --- |
| Each ball’s `x` and `y` | Its position on the course. |
| Each ball’s `vx` and `vy` | Its horizontal and vertical velocity. |
| `activePlayer` | Whose turn it is. |
| `holeIndex` | Which course is loaded. |
| Each player’s `scores` and `done` arrays | Their strokes and completion status for each hole. |
| `shotInProgress` | Whether another shot should be blocked while the current shot is resolving. |

These values continue to exist between frames. The game does not recreate the score or forget the ball’s position every time it draws the screen.

### Understanding the Interface and Canvas

The file is called `minigolf.html`, but it contains HTML, CSS, and JavaScript.

HTML creates elements such as the canvas, power slider, buttons, and score table. CSS controls their appearance and layout. JavaScript handles the game rules, updates the interface, and draws the course.

For example, the power slider is created in HTML. CSS controls how it looks, while JavaScript reads its value when I shoot. This gives me a concrete example of the three languages working together.

The first course uses a canvas size of 900 by 520 pixels. Later courses have larger internal dimensions, reaching 1,500 by 860 on hole 7. CSS scales the canvas to fit the page.

That creates an important issue for mouse input. The pointer’s screen coordinates must be converted into canvas coordinates. If a 1,500-pixel-wide canvas is displayed at 750 pixels wide, moving 100 screen pixels represents 200 canvas pixels. Without that conversion, the aim could become inaccurate when the display size changed.

### Improving Aiming and Power Controls

After trying the first version, I felt that adjusting the direction was too slow. I asked for mouse aiming and a specific key for putting. Later, I requested arrow-key power control so I could adjust strength without moving my mouse away from the course.

The current controls are:

- Mouse movement to aim.
- Up and down arrows to change power by 5%.
- Space to putt.
- A slider as another way to adjust power.

`updateAim()` finds the direction from the ball to the pointer:

`aimAngle = Math.atan2(y - b.y, x - b.x)`

The subtraction gives the horizontal and vertical differences. `Math.atan2()` converts those differences into an angle.

The power listener checks for `ArrowUp` and `ArrowDown`. It adds or subtracts five, then limits the value to between 5 and 100. It also uses `event.preventDefault()` so the arrow keys control power instead of scrolling the page.

This is an example of how an input can affect state without immediately moving the ball. Changing power updates the selected strength. The ball only receives that strength as velocity when a shot is taken.

<video controls playsinline preload="metadata" style="width: 100%;">
  <source src="videos/power-controls.mp4" type="video/mp4">
  Your browser does not support embedded video.
</video>

<p><em>The arrow keys change shot power by 5% while the mouse keeps the aiming direction in place.</em></p>

### Following a Shot Through the Code

In `shoot()`, power becomes speed:

`const speed = Number(powerInput.value) / 100 * MAX_SPEED`

Since `MAX_SPEED` is 620 pixels per second, 50% power produces a starting speed of 310 pixels per second.

The aiming angle separates this speed into horizontal and vertical velocity:

`b.vx = Math.cos(aimAngle) * speed`

`b.vy = Math.sin(aimAngle) * speed`

A shot directly right has horizontal movement and no vertical movement. A diagonal shot uses both components.

The update function changes the position:

`b.x += b.vx * dt`

`b.y += b.vy * dt`

Here, `dt` represents elapsed time for a physics step. The game uses steps of `1 / 120` of a second. At 240 pixels per second, the ball would move two pixels during one step before accounting for collisions and friction.

Friction reduces the speed while preserving its direction. Eventually, the code sets very slow movement to zero so the ball comes to a complete stop.

The loop uses `requestAnimationFrame(loop)` to schedule another frame. It can run several small physics updates before drawing once. This separates the timing of the movement calculations from the browser’s display rate.

### Expanding the Courses and Finding Repetition

I next requested five larger holes with increasing difficulty and a two-player mode. However, the first expanded version used too many similar straight barriers. I told ChatGPT:

> the holes are just kinda like repeating

This helped me identify a difference between adding more content and adding different decisions. A larger course could still feel repetitive if the player kept navigating the same type of obstacle.

I asked for curved obstacles and places where the ball could fall off the map. The next version introduced circular bumpers, crescent-shaped walls, a narrow crossing, and winding platforms.

The courses are stored in the `COURSES` array. Each course contains its dimensions, starting position, cup, obstacles, and any water hazards. `loadHole()` selects that information and resets the players’ positions for the new hole.

This allows the game to reuse its movement and drawing functions across different layouts.

### Revising the Water Hazards

The first falling-hazard version became too frustrating because too much of the surrounding area was a drop. I asked for hazards only in certain places and protective walls along some edges.

The revised version used grass as the main surface, solid outer walls, and separate blue ponds. Entering water returned the affected ball to that hole’s tee with one penalty stroke.

I later felt that too many walls protected the ponds, so I asked for some rails to be shortened or removed. This kept water relevant without making every missed shot leave the course.

These revisions were gameplay adjustments rather than fixes for syntax errors. The code could run, but its behavior did not match the experience I wanted.

`isInWater()` checks whether the ball’s center is inside a pond. Rectangular ponds use comparisons with their four edges. Circular ponds use the distance from the ball to the pond’s center.

The pond data is also used to draw the water. This connects the visible hazard to the detection area.

<img width="885" height="796" alt="Screenshot 2026-10-14 at 6 56 35 PM" src="https://github.com/user-attachments/assets/ed90b212-c504-4ffc-a858-cf63a8c923b4" />

*Caption: I changed the hazard layout through several revisions to balance risk with room to recover.*

### Adding and Adjusting Ball Collisions

The first two-player version tracked separate balls without making them collide. I requested a small bump so players could affect each other’s positions.

That first collision version was too weak for what I wanted. I then asked for stronger collisions while keeping a limit on the resulting speed.

ChatGPT wrote and revised the collision code. I directed the changes based on how the interaction felt and used the code to identify the settings responsible for its strength.

`collideBalls()` calculates the distance between the ball centers using `Math.hypot(dx, dy)`. Each ball has a radius of 10 pixels. When their centers are less than 20 pixels apart, the balls overlap.

The function first separates them. It then checks whether they are moving toward each other before applying a push. If they are already separating, it returns without applying another impulse. This prevents the same contact from repeatedly adding movement.

The current settings are:

| Setting | Value | Role |
| --- | --- | --- |
| `BALL_RESTITUTION` | `0.5` | Affects how strongly the balls separate during the impact calculation. |
| `IMPACT_DAMPING` | `0.8` | Reduces the resulting velocity components after impact. |
| `MAX_BUMP_SPEED` | `240` | Limits each ball’s speed after the collision. |

The first gentle version used a speed cap of 110. Increasing that cap to 240 allowed a more noticeable bump. The effect also depends on damping, the impact direction, and the incoming speed, so the cap alone does not determine every collision.

A simplified straight-on example helps me follow the calculation. If one ball approaches a stationary ball at 300 pixels per second, the current collision calculation gives speeds of 75 and 225 before damping. Multiplying by 0.8 reduces them to 60 and 180. Neither reaches the speed cap.

<video controls playsinline preload="metadata" style="width: 100%;">
  <source src="videos/ball-collisions.mp4" type="video/mp4">
  Your browser does not support embedded video.
</video>

<p><em>One ball hits the other, transferring movement. The game waits for both balls to stop before switching turns.</em></p>

### Understanding Why Both Balls Must Finish Moving

Ball collisions changed more than the bounce calculation. A player’s shot could now move the opponent’s ball, so the game needed to update both balls.

The current code waits until both unfinished balls have stopped and any water-reset animation has finished before switching turns. This is controlled by the `stillMoving` check.

Each player has their own scores and completion flags. If a bumped ball enters water, the penalty belongs to that ball’s owner. If a player finishes a hole, the turn-selection code skips them until the next hole.

This shows how adding one feature can affect several sections of a program. Ball collisions required changes to movement, penalties, and turn handling as well as contact detection.

### Refining the Cup and Reaching Seven Holes

The original cup required a slow arrival. I asked to remove the speed restriction because I wanted accurate shots to count at any speed.

Later, I requested a smaller cup and hitbox. The current comparison checks whether the ball’s center is within the cup’s radius:

`Math.hypot(b.x - course.cup.x, b.y - course.cup.y) <= (course.cup.r || CUP_RADIUS)`

Most cups have a radius of 14 pixels. Hole 7 uses 12. The earlier, more generous check also included the ball’s radius, so the outside edge of the ball could touch the cup and count.

This helped me distinguish a visible shape from its hitbox. Making the drawing smaller would not make the game more precise unless the detection area also changed.

I expanded the game to seven holes and requested that hole 7 be the hardest. It combines the largest course area, exposed ponds, several obstacles, and a small cup inside a curved barrier. That is the intended difficulty progression, but it still needs to be evaluated through complete playthroughs.

<img width="885" height="835" alt="Screenshot 2026-10-14 at 7 01 43 PM" src="https://github.com/user-attachments/assets/679eef89-b2bc-49c3-b2cc-cee1b1fdf759" />

*Caption: The final course combines earlier mechanics and uses a smaller target, while still allowing successful shots at any speed.*


### Outcome, AI Use, and Scope

The current result is a seven-hole mini-golf game with one-player and two-player modes, mouse aiming, arrow-key power control, curved obstacles, water penalties, ball collisions, and a scorecard. I downloaded `minigolf.html` and added it to my GitHub project.

ChatGPT generated the initial code and implemented the revisions. I selected the game, evaluated versions, identified problems, requested changes, and asked for explanations. My prompts became more specific as I had actual results to respond to.

The project also grew substantially from the first one-hole version. Although I kept aiming and putting as the core mechanic, multiplayer and hazards added more state and interactions. I now need to pause expansion and focus on understanding and verifying the existing game.

