<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>Jeu de Tir - Guerre</title>
  <style>
    canvas {
      background: #333;
      display: block;
      margin: 0 auto;
    }
  </style>
</head>
<body>
  <canvas id="game" width="800" height="600"></canvas>
  <audio id="gunshot" src="https://cdn.pixabay.com/audio/2021/11/04/audio_2657f9fa96.mp3"></audio>
  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const player = {
      x: 400,
      y: 300,
      size: 20,
      speed: 4,
      color: "#0f0",
      hp: 100,
      score: 0,
      showFlash: false,
      flashTimer: 0,
      angle: 0
    };

    let bullets = [];
    let enemies = [];
    let keys = {};
    let wave = 1;
    let spawnTimer = 0;
    const gunshotSound = document.getElementById("gunshot");

    window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
    window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

    canvas.addEventListener("click", e => {
      const angle = Math.atan2(
        e.clientY - canvas.offsetTop - player.y,
        e.clientX - canvas.offsetLeft - player.x
      );
      player.angle = angle;
      bullets.push({
        x: player.x,
        y: player.y,
        dx: Math.cos(angle) * 7,
        dy: Math.sin(angle) * 7
      });
      gunshotSound.currentTime = 0;
      gunshotSound.play();
      player.showFlash = true;
      player.flashTimer = 5;
    });

    function spawnEnemies(n) {
      for (let i = 0; i < n; i++) {
        enemies.push({
          x: Math.random() * canvas.width,
          y: -20,
          size: 20,
          speed: 1 + Math.random(),
          color: "#f00"
        });
      }
    }

    function update() {
      if (keys["z"]) player.y -= player.speed;
      if (keys["s"]) player.y += player.speed;
      if (keys["q"]) player.x -= player.speed;
      if (keys["d"]) player.x += player.speed;

      bullets.forEach((b, i) => {
        b.x += b.dx;
        b.y += b.dy;
        if (b.x < 0 || b.y < 0 || b.x > canvas.width || b.y > canvas.height) bullets.splice(i, 1);
      });

      enemies.forEach((e, ei) => {
        const angle = Math.atan2(player.y - e.y, player.x - e.x);
        e.x += Math.cos(angle) * e.speed;
        e.y += Math.sin(angle) * e.speed;

        bullets.forEach((b, bi) => {
          const dx = e.x - b.x;
          const dy = e.y - b.y;
          const dist = Math.sqrt(dx * dx + dy * dy);
          if (dist < e.size) {
            enemies.splice(ei, 1);
            bullets.splice(bi, 1);
            player.score += 10;
          }
        });

        const dx = e.x - player.x;
        const dy = e.y - player.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < e.size) {
          enemies.splice(ei, 1);
          player.hp -= 10;
          if (player.hp <= 0) {
            alert("Game Over ! Score: " + player.score);
            document.location.reload();
          }
        }
      });

      if (enemies.length === 0) {
        spawnTimer++;
        if (spawnTimer > 60) {
          wave++;
          spawnEnemies(wave * 2);
          spawnTimer = 0;
        }
      }

      if (player.showFlash) {
        player.flashTimer--;
        if (player.flashTimer <= 0) {
          player.showFlash = false;
        }
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Joueur
      ctx.fillStyle = player.color;
      ctx.fillRect(player.x - player.size / 2, player.y - player.size / 2, player.size, player.size);

      // Flamme du tir
      if (player.showFlash) {
        const muzzleX = player.x + Math.cos(player.angle) * (player.size / 2 + 5);
        const muzzleY = player.y + Math.sin(player.angle) * (player.size / 2 + 5);
        ctx.fillStyle = "orange";
        ctx.beginPath();
        ctx.moveTo(muzzleX, muzzleY);
        ctx.lineTo(muzzleX + Math.cos(player.angle + 0.2) * 10, muzzleY + Math.sin(player.angle + 0.2) * 10);
        ctx.lineTo(muzzleX + Math.cos(player.angle - 0.2) * 10, muzzleY + Math.sin(player.angle - 0.2) * 10);
        ctx.closePath();
        ctx.fill();
      }

      // Balles
      ctx.fillStyle = "#fff";
      bullets.forEach(b => {
        ctx.beginPath();
        ctx.arc(b.x, b.y, 5, 0, Math.PI * 2);
        ctx.fill();
      });

      // Ennemis
      enemies.forEach(e => {
        ctx.fillStyle = e.color;
        ctx.fillRect(e.x - e.size / 2, e.y - e.size / 2, e.size, e.size);
      });

      // Barre de vie
      ctx.fillStyle = "red";
      ctx.fillRect(10, 30, 100, 10);
      ctx.fillStyle = "lime";
      ctx.fillRect(10, 30, player.hp, 10);
      ctx.strokeStyle = "#fff";
      ctx.strokeRect(10, 30, 100, 10);

      // Infos
      ctx.fillStyle = "#fff";
      ctx.fillText("Vague: " + (wave - 1), 10, 20);
      ctx.fillText("Score: " + player.score, 10, 55);
    }

    function loop() {
      update();
      draw();
      requestAnimationFrame(loop);
    }

    spawnEnemies(3);
    loop();
  </script>
</body>
</html>
