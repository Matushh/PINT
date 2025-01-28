<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Typewriter ASCII Fire Effect</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: white;
      font-family: monospace;
      color: grey; /* Grey color for flames */
    }
    canvas {
      display: block;
    }
    #hiddenMessage {
      position: absolute;
      bottom: 10%;
      width: 100%;
      text-align: center;
      font-size: 48px;
      font-weight: bold;
      color: grey;
      opacity: 0;
      transition: opacity 3s ease-in-out;
      z-index: 10;
    }
    #hiddenMessage.show {
      opacity: 1;
    }
    #controlPanel {
      position: absolute;
      bottom: 5%;
      right: 5%;
      background: rgba(255, 255, 255, 0.8);
      padding: 10px;
      border-radius: 8px;
      font-family: monospace;
      z-index: 20;
    }
    #controlPanel input {
      font-size: 16px;
      padding: 5px;
      margin-top: 5px;
      width: 100%;
    }
    #centerText {
      position: absolute;
      top: 5%;
      left: 50%;
      transform: translateX(-50%);
      text-align: center;
      font-size: 96px;
      font-weight: bold;
      font-family: "Courier New", Courier, monospace;
      text-shadow: 3px 3px 5px rgba(0, 0, 0, 0.5);
      color: black;
      z-index: 15;
    }
  </style>
</head>
<body>
  <canvas id="fireCanvas"></canvas>
  <div id="hiddenMessage">PLEASE HIRE ME</div>
  <div id="centerText">WELCOME</div>
  <div id="controlPanel">
    <label for="wordInput">Enter Words (comma-separated):</label>
    <input type="text" id="wordInput" value="PINTS?,BEER,PUB?">
    <label for="centerTextInput" style="margin-top: 10px;">Center Top Text:</label>
    <input type="text" id="centerTextInput" value="WELCOME">
  </div>

  <script>
    const canvas = document.getElementById("fireCanvas");
    const ctx = canvas.getContext("2d");
    const wordInput = document.getElementById("wordInput");
    const centerTextInput = document.getElementById("centerTextInput");
    const centerTextDiv = document.getElementById("centerText");

    centerTextInput.addEventListener("input", () => {
      centerTextDiv.textContent = centerTextInput.value;
    });

    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      centerTextDiv.style.top = `${window.innerHeight * 0.05}px`;
    }

    resizeCanvas();

    const chars = ["#", "|", "/", "\\", "-", ".", "*"]; // Flame-like symbols
    const flames = [];
    const hiddenMessage = document.getElementById("hiddenMessage");
    let showMessage = false;

    const texts = []; // Store active texts

    class Flame {
      constructor(x, y, char, intensity, delay) {
        this.x = x;
        this.y = y;
        this.char = char;
        this.intensity = intensity; // Brightness factor
        this.speed = 0.1; // Uniform slower upward speed
        this.delay = delay; // Delay for typewriter effect
      }

      draw() {
        if (this.delay <= 0) {
          ctx.fillStyle = `rgba(128, 128, 128, ${this.intensity / 255})`; // Grey flames
          ctx.fillText(this.char, this.x, this.y);
        }
      }

      update() {
        if (this.delay > 0) {
          this.delay -= 1; // Reduce delay over time
        } else {
          this.y -= this.speed; // Upward movement
          this.intensity -= 0.05; // Very slow fading
          if (this.y < -20 || this.intensity <= 0) {
            this.reset();
          }
        }
      }

      reset() {
        this.y = canvas.height + Math.random() * 30; // Reset below the screen
        this.x = Math.random() * canvas.width;
        this.char = chars[Math.floor(Math.random() * chars.length)];
        this.intensity = Math.random() * 255 + 100; // Reset intensity
        this.delay = Math.random() * 300; // Increased delay for more staggered effect
      }
    }

    class Text {
      constructor(text, x, y, opacity, size) {
        this.text = text;
        this.x = x;
        this.y = y;
        this.opacity = opacity;
        this.size = size; // Random size
        this.lifespan = 300; // Text stays for 300 frames
      }

      draw() {
        ctx.font = `bold ${this.size}px monospace`;
        ctx.fillStyle = `rgba(128, 128, 128, ${this.opacity})`;
        ctx.fillText(this.text, this.x, this.y);
      }

      update() {
        this.lifespan -= 1;
        if (this.lifespan < 0) {
          return false; // Indicate text is expired
        }
        return true;
      }
    }

    function createFlames() {
      const cols = Math.floor(canvas.width / 14); // Adjust column spacing
      const rows = Math.floor(canvas.height / 20); // Adjust density

      flames.length = 0;
      for (let row = rows - 1; row >= 0; row--) {
        for (let col = 0; col < cols; col++) {
          const x = col * 14;
          const y = canvas.height - row * 20; // Start at the bottom, row by row
          const char = chars[Math.floor(Math.random() * chars.length)];
          const intensity = Math.random() * 255 + 100;
          const delay = row * 50 + Math.random() * 150; // Staggered row-by-row delay
          flames.push(new Flame(x, y, char, intensity, delay));
        }
      }
    }

    function drawRandomText() {
      const randomX = Math.random() * canvas.width * 0.8; // Ensure text stays on screen
      const randomY = Math.random() * canvas.height * 0.8;
      const randomSize = Math.random() * 190 + 20; // Random size between 20px and 210px
      const words = wordInput.value.split(",").map(word => word.trim());
      const randomText = words[Math.floor(Math.random() * words.length)]; // Random text from user input
      const newText = new Text(randomText, randomX, randomY, 1, randomSize);
      texts.push(newText);
    }

    function animate() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.font = "14px monospace";

      flames.forEach((flame) => {
        flame.update();
        flame.draw();
      });

      if (Math.random() < 0.01) { // Random chance to draw text
        drawRandomText();
      }

      for (let i = texts.length - 1; i >= 0; i--) {
        const text = texts[i];
        text.draw();
        if (!text.update()) {
          texts.splice(i, 1); // Remove expired text
        }
      }

      // Reveal the hidden message once the flames are stable
      if (!showMessage && flames.every(flame => flame.delay <= 0)) {
        hiddenMessage.classList.add("show");
        showMessage = true;
      }

      requestAnimationFrame(animate);
    }

    window.addEventListener("resize", () => {
      resizeCanvas();
      createFlames();
    });

    createFlames();
    animate();
  </script>
</body>
</html>

