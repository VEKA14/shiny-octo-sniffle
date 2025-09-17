# shiny-octo-sniffle
from flask import Flask, render_template_string

app = Flask(__name__)

HTML_PAGE = """
<!DOCTYPE html>
<html>
<head>
  <title>Snake Game</title>
  <style>
    body { text-align: center; background: #111; color: white; }
    canvas { background: black; display: block; margin: auto; }
    #restartBtn { 
      display: none; 
      margin-top: 10px; 
      padding: 10px 20px; 
      font-size: 16px; 
      border: none; 
      border-radius: 8px; 
      background: limegreen; 
      color: white; 
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h1>Snake Game</h1>
  <canvas id="gameCanvas" width="1280" height="500"></canvas>
  <p>Use Arrow Keys to Play</p>
  <button id="restartBtn" onclick="resetGame()">Play Again</button>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    const box = 10;
    let snake, direction, food, score;
    let gameOver = false;
    let gameLoop;

    // Reset game state
    function resetGame() {
      snake = [{x: 300, y: 200}];
      direction = null;
      score = 0;
      gameOver = false;
      food = {
        x: Math.floor(Math.random() * (canvas.width / box)) * box,
        y: Math.floor(Math.random() * (canvas.height / box)) * box
      };

      document.getElementById("restartBtn").style.display = "none";

      if (gameLoop) clearInterval(gameLoop); // clear old loop
      gameLoop = setInterval(draw, 100); // start new loop
    }

    // Controls
    document.addEventListener("keydown", event => {
      if (event.key === "ArrowLeft" && direction !== "RIGHT") direction = "LEFT";
      if (event.key === "ArrowUp" && direction !== "DOWN") direction = "UP";
      if (event.key === "ArrowRight" && direction !== "LEFT") direction = "RIGHT";
      if (event.key === "ArrowDown" && direction !== "UP") direction = "DOWN";
    });

    function draw() {
      if (gameOver) return;

      ctx.fillStyle = "black";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Draw snake
      ctx.fillStyle = "lime";
      snake.forEach(part => ctx.fillRect(part.x, part.y, box, box));

      // Draw food
      ctx.fillStyle = "red";
      ctx.fillRect(food.x, food.y, box, box);

      // Movement
      let head = {x: snake[0].x, y: snake[0].y};
      if (direction === "LEFT") head.x -= box;
      if (direction === "RIGHT") head.x += box;
      if (direction === "UP") head.y -= box;
      if (direction === "DOWN") head.y += box;

      // Game over check
      if (
        head.x < 0 || head.y < 0 ||
        head.x >= canvas.width || head.y >= canvas.height ||
        snake.some((part, index) => index !== 0 && part.x === head.x && part.y === head.y)
      ) {
        alert("Game Over! Final Score: " + score);
        gameOver = true;
        document.getElementById("restartBtn").style.display = "inline-block";
        clearInterval(gameLoop); // stop loop
        return;
      }

      // Eat food
      if (head.x === food.x && head.y === food.y) {
        score++;
        food = {
          x: Math.floor(Math.random() * (canvas.width / box)) * box,
          y: Math.floor(Math.random() * (canvas.height / box)) * box
        };
      } else {
        snake.pop();
      }

      snake.unshift(head);

      // Score display
      ctx.fillStyle = "white";
      ctx.font = "20px Arial";
      ctx.fillText("Score: " + score, 10, 20);
    }

    // Start the first game
    resetGame();
  </script>
</body>
</html>
"""


@app.route("/")
def index():
    return render_template_string(HTML_PAGE)


if __name__ == "__main__":
    app.run(debug=True)
