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

## Arcade Unit: Getting Started

### My Goal for This Week

My goal this week is to understand how a game’s graphical user interface, or GUI, works in a browser. I want to learn how HTML creates the page structure, CSS changes its appearance, and JavaScript makes it respond to the player. When I ask AI to create a game, I want to understand enough of the code to explain important sections and make small changes myself.

By the end of the week, I want to explain at least three sections of my chosen game’s code and predict what a small change will do before testing it.

### Starting Point

I started by asking another AI to generate Pong and Tag as examples of browser games. These are starting experiments, and I have not chosen my actual game project yet. I will probably work on a different game, but these examples give me an introduction to what a complete game file looks like.

After generating Pong, I asked the AI to explain its code. This introduced me to the different roles of HTML, CSS, and JavaScript within one `.html` file. For now, my focus is getting familiar with the basic structure rather than studying every part of these two games.

### Initial Observations

- Both games contain HTML, CSS, and JavaScript in one file. Even though the file ends in `.html`, much of the game’s behavior comes from JavaScript.
- Both use an `800 × 500` canvas for the playing area. The buttons are separate HTML elements, while JavaScript draws the moving objects inside the canvas.
- Both have an `update()` function and a `draw()` function. One changes values such as positions and scores, and the other displays the current game.
- Variables such as `paused`, `gameOver`, and `score` keep track of what is happening. Keyboard and button event listeners let the player affect those values.
- The AI added several features beyond my basic requests. For my actual project, I want to give clearer limits so the first version is small enough for me to understand.

### Beginning to Debug

When Tag did not work, my first message to the AI was too vague to identify the problem and the AI didn't give me the response I wanted. I then used the prompt of “The Tag game you generated is not working as expected. Help me identify the cause before rewriting it. Ask me which browser I am using, how I opened the file, what appears on the screen, and whether the buttons or keyboard controls respond. Show me how to find any error message in the browser console. Then explain what the error means, identify the relevant code, and suggest the smallest change needed. Explain how I can test whether the change fixes the problem.” The AI then suggested that localStorage, which saves high scores in the browser, might be causing an error. It added try/catch around the storage commands so that a storage error would not stop the game from running. If browser storage is unavailable, the code uses a temporary storage object instead. This keeps scores available while the page stays open, but they would be lost after reloading or closing it.

For my actual game, I want to use this process to understand what failed and why a change helps.
