play games in your consol:(function() {
  const canvas = document.createElement('canvas');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  canvas.style.position = 'fixed';
  canvas.style.top = '0';
  canvas.style.left = '0';
  canvas.style.zIndex = '999999';
  canvas.style.backgroundColor = '#000';
  canvas.style.margin = '0';
  canvas.style.padding = '0';
  document.body.style.margin = '0';
  document.body.style.padding = '0';
  document.body.style.overflow = 'hidden';
  document.body.appendChild(canvas);
  
  const ctx = canvas.getContext('2d');
  const gridSize = 20;
  
  let gameState = 'mainMenu';
  let gameMode = 'single';
  let snake1 = [{x: 10, y: 10}];
  let snake2 = [{x: Math.floor(canvas.width / gridSize) - 20, y: Math.floor(canvas.height / gridSize) - 20}];
  let apples = [{x: 15, y: 15}, {x: 20, y: 20}, {x: 25, y: 25}];
  let bananas = [{x: 12, y: 12}, {x: 18, y: 18}];
  let enemies = [{x: 30, y: 30}, {x: 40, y: 40}];
  let dx1 = 1, dy1 = 0;
  let nextDx1 = 1, nextDy1 = 0;
  let dx2 = -1, dy2 = 0;
  let nextDx2 = -1, nextDy2 = 0;
  let score1 = 0;
  let score2 = 0;
  let finalScore1 = 0;
  let finalScore2 = 0;
  let costume1 = 'default';
  let costume2 = 'default';
  let enemyMoveCounter = 0;
  let enemyMoveSpeed = 3;
  let gameOverTimer = 0;
  
  window.addEventListener('resize', () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  });
  
  function requestFullscreen() {
    if (canvas.requestFullscreen) {
      canvas.requestFullscreen().catch(err => console.log('Fullscreen request failed:', err));
    } else if (canvas.webkitRequestFullscreen) {
      canvas.webkitRequestFullscreen();
    } else if (canvas.mozRequestFullScreen) {
      canvas.mozRequestFullScreen();
    } else if (canvas.msRequestFullscreen) {
      canvas.msRequestFullscreen();
    }
  }
  
  window.addEventListener('keydown', (e) => {
    if (gameState === 'mainMenu') {
      if (e.key === '1') selectMode('single');
      if (e.key === '2') selectMode('multi');
    } else if (gameState === 'playing') {
      if (e.key === 'ArrowUp' && dy1 === 0) { nextDx1 = 0; nextDy1 = -1; }
      if (e.key === 'ArrowDown' && dy1 === 0) { nextDx1 = 0; nextDy1 = 1; }
      if (e.key === 'ArrowLeft' && dx1 === 0) { nextDx1 = -1; nextDy1 = 0; }
      if (e.key === 'ArrowRight' && dx1 === 0) { nextDx1 = 1; nextDy1 = 0; }
      
      if (gameMode === 'multi') {
        if (e.key === 'w' && dy2 === 0) { nextDx2 = 0; nextDy2 = -1; }
        if (e.key === 's' && dy2 === 0) { nextDx2 = 0; nextDy2 = 1; }
        if (e.key === 'a' && dx2 === 0) { nextDx2 = -1; nextDy2 = 0; }
        if (e.key === 'd' && dx2 === 0) { nextDx2 = 1; nextDy2 = 0; }
      }
    } else if (gameState === 'gameOver') {
      if (e.key === 'ArrowDown') goToMainMenu();
    }
  });
  
  function selectMode(mode) {
    gameMode = mode;
    gameState = 'playing';
    resetGame();
    requestFullscreen();
    gameLoop();
  }
  
  function resetGame() {
    snake1 = [{x: 10, y: 10}];
    snake2 = [{x: Math.floor(canvas.width / gridSize) - 20, y: Math.floor(canvas.height / gridSize) - 20}];
    apples = [{x: 15, y: 15}, {x: 20, y: 20}, {x: 25, y: 25}];
    bananas = [{x: 12, y: 12}, {x: 18, y: 18}];
    enemies = [{x: 30, y: 30}, {x: 40, y: 40}];
    dx1 = 1; dy1 = 0;
    nextDx1 = 1; nextDy1 = 0;
    dx2 = -1; dy2 = 0;
    nextDx2 = -1; nextDy2 = 0;
    score1 = 0;
    score2 = 0;
    costume1 = 'default';
    costume2 = 'default';
    enemyMoveCounter = 0;
    gameOverTimer = 0;
  }
  
  function spawnApple() {
    let x = Math.floor(Math.random() * (canvas.width / gridSize));
    let y = Math.floor(Math.random() * (canvas.height / gridSize));
    apples.push({x: x, y: y});
  }
  
  function spawnBanana() {
    let x = Math.floor(Math.random() * (canvas.width / gridSize));
    let y = Math.floor(Math.random() * (canvas.height / gridSize));
    bananas.push({x: x, y: y});
  }
  
  function spawnEnemy() {
    let x = Math.floor(Math.random() * (canvas.width / gridSize));
    let y = Math.floor(Math.random() * (canvas.height / gridSize));
    enemies.push({x: x, y: y});
  }
  
  function moveEnemies() {
    enemyMoveCounter++;
    if (enemyMoveCounter < enemyMoveSpeed) {
      return;
    }
    enemyMoveCounter = 0;
    
    for (let enemy of enemies) {
      let dirX = 0, dirY = 0;
      
      if (Math.random() > 0.3) {
        if (enemy.x < snake1[0].x) dirX = 1;
        else if (enemy.x > snake1[0].x) dirX = -1;
        
        if (enemy.y < snake1[0].y) dirY = 1;
        else if (enemy.y > snake1[0].y) dirY = -1;
      } else {
        dirX = Math.floor(Math.random() * 3) - 1;
        dirY = Math.floor(Math.random() * 3) - 1;
      }
      
      enemy.x += dirX;
      enemy.y += dirY;
      
      const maxX = Math.floor(canvas.width / gridSize);
      const maxY = Math.floor(canvas.height / gridSize);
      enemy.x = (enemy.x + maxX) % maxX;
      enemy.y = (enemy.y + maxY) % maxY;
    }
  }
  
  function updateCostume(score, isPlayer2 = false) {
    if (score >= 500) {
      if (isPlayer2) costume2 = 'neon';
      else costume1 = 'neon';
    } else if (score >= 300) {
      if (isPlayer2) costume2 = 'cyber';
      else costume1 = 'cyber';
    } else if (score >= 150) {
      if (isPlayer2) costume2 = 'golden';
      else costume1 = 'golden';
    }
  }
  
  function gameLoop() {
    if (gameState !== 'playing') return;
    
    dx1 = nextDx1;
    dy1 = nextDy1;
    
    let head1 = {
      x: snake1[0].x + dx1,
      y: snake1[0].y + dy1
    };
    
    const maxX = Math.floor(canvas.width / gridSize);
    const maxY = Math.floor(canvas.height / gridSize);
    head1.x = (head1.x + maxX) % maxX;
    head1.y = (head1.y + maxY) % maxY;
    
    for (let i = 0; i < snake1.length; i++) {
      if (head1.x === snake1[i].x && head1.y === snake1[i].y) {
        endGame();
        return;
      }
    }
    
    for (let enemy of enemies) {
      if (head1.x === enemy.x && head1.y === enemy.y) {
        endGame();
        return;
      }
    }
    
    if (gameMode === 'multi') {
      for (let segment of snake2) {
        if (head1.x === segment.x && head1.y === segment.y) {
          endGame();
          return;
        }
      }
    }
    
    snake1.unshift(head1);
    
    let ate1 = false;
    for (let i = 0; i < apples.length; i++) {
      if (head1.x === apples[i].x && head1.y === apples[i].y) {
        score1 += 10;
        apples.splice(i, 1);
        spawnApple();
        updateCostume(score1, false);
        ate1 = true;
        break;
      }
    }
    
    if (!ate1) {
      for (let i = 0; i < bananas.length; i++) {
        if (head1.x === bananas[i].x && head1.y === bananas[i].y) {
          score1 += 25;
          bananas.splice(i, 1);
          spawnBanana();
          updateCostume(score1, false);
          ate1 = true;
          break;
        }
      }
    }
    
    if (!ate1) {
      snake1.pop();
    }
    
    if (gameMode === 'multi') {
      dx2 = nextDx2;
      dy2 = nextDy2;
      
      let head2 = {
        x: snake2[0].x + dx2,
        y: snake2[0].y + dy2
      };
      
      head2.x = (head2.x + maxX) % maxX;
      head2.y = (head2.y + maxY) % maxY;
      
      for (let i = 0; i < snake2.length; i++) {
        if (head2.x === snake2[i].x && head2.y === snake2[i].y) {
          endGame();
          return;
        }
      }
      
      for (let enemy of enemies) {
        if (head2.x === enemy.x && head2.y === enemy.y) {
          endGame();
          return;
        }
      }
      
      for (let segment of snake1) {
        if (head2.x === segment.x && head2.y === segment.y) {
          endGame();
          return;
        }
      }
      
      snake2.unshift(head2);
      
      let ate2 = false;
      for (let i = 0; i < apples.length; i++) {
        if (head2.x === apples[i].x && head2.y === apples[i].y) {
          score2 += 10;
          apples.splice(i, 1);
          spawnApple();
          updateCostume(score2, true);
          ate2 = true;
          break;
        }
      }
      
      if (!ate2) {
        for (let i = 0; i < bananas.length; i++) {
          if (head2.x === bananas[i].x && head2.y === bananas[i].y) {
            score2 += 25;
            bananas.splice(i, 1);
            spawnBanana();
            updateCostume(score2, true);
            ate2 = true;
            break;
          }
        }
      }
      
      if (!ate2) {
        snake2.pop();
      }
    }
    
    moveEnemies();
    
    let totalScore = gameMode === 'multi' ? score1 + score2 : score1;
    if (totalScore > 0 && totalScore % 100 === 0 && enemies.length < 5) {
      spawnEnemy();
    }
    
    draw();
    setTimeout(gameLoop, 100);
  }
  
  function endGame() {
    gameState = 'gameOver';
    finalScore1 = score1;
    finalScore2 = score2;
    gameOverTimer = 0;
    draw();
  }
  
  function goToMainMenu() {
    gameState = 'mainMenu';
    if (document.fullscreenElement) {
      document.exitFullscreen().catch(err => console.log('Exit fullscreen failed:', err));
    } else if (document.webkitFullscreenElement) {
      document.webkitExitFullscreen();
    } else if (document.mozFullScreenElement) {
      document.mozCancelFullScreen();
    } else if (document.msFullscreenElement) {
      document.msExitFullscreen();
    }
    draw();
  }
  
  function draw() {
    ctx.fillStyle = '#000';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    if (gameState === 'mainMenu') {
      drawMainMenu();
    } else if (gameState === 'playing') {
      drawGame();
    } else if (gameState === 'gameOver') {
      gameOverTimer++;
      drawGame();
      drawGameOver();
    }
  }
  
  function drawMainMenu() {
    ctx.fillStyle = '#00FF00';
    ctx.font = 'bold 80px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('SNAKE GAME', canvas.width / 2, canvas.height / 2 - 200);
    
    ctx.fillStyle = '#FFFFFF';
    ctx.font = '40px Arial';
    ctx.fillText('Select Game Mode', canvas.width / 2, canvas.height / 2 - 50);
    
    ctx.fillStyle = '#00AA00';
    ctx.fillRect(canvas.width / 2 - 250, canvas.height / 2 + 50, 200, 80);
    ctx.fillStyle = '#000000';
    ctx.font = 'bold 28px Arial';
    ctx.fillText('Press 1', canvas.width / 2 - 150, canvas.height / 2 + 105);
    ctx.fillStyle = '#FFFFFF';
    ctx.font = '20px Arial';
    ctx.fillText('SINGLE PLAYER', canvas.width / 2 - 150, canvas.height / 2 + 135);
    
    ctx.fillStyle = '#FF00FF';
    ctx.fillRect(canvas.width / 2 + 50, canvas.height / 2 + 50, 200, 80);
    ctx.fillStyle = '#000000';
    ctx.font = 'bold 28px Arial';
    ctx.fillText('Press 2', canvas.width / 2 + 150, canvas.height / 2 + 105);
    ctx.fillStyle = '#FFFFFF';
    ctx.font = '20px Arial';
    ctx.fillText('MULTIPLAYER', canvas.width / 2 + 150, canvas.height / 2 + 135);
    
    ctx.fillStyle = '#FFFF00';
    ctx.font = '18px Arial';
    ctx.fillText('Player 1: Arrow Keys | Player 2: WASD', canvas.width / 2, canvas.height / 2 + 250);
  }
  
  function drawGame() {
    function drawSnake(snake, costume) {
      let headColor, bodyColor;
      if (costume === 'golden') {
        headColor = '#FFD700';
        bodyColor = '#FFA500';
      } else if (costume === 'cyber') {
        headColor = '#00FFFF';
        bodyColor = '#0099FF';
      } else if (costume === 'neon') {
        headColor = '#FF00FF';
        bodyColor = '#00FF00';
      } else {
        headColor = '#00FF00';
        bodyColor = '#00AA00';
      }
      
      ctx.fillStyle = bodyColor;
      for (let i = 1; i < snake.length; i++) {
        ctx.fillRect(snake[i].x * gridSize, snake[i].y * gridSize, gridSize - 2, gridSize - 2);
      }
      
      ctx.fillStyle = headColor;
      ctx.fillRect(snake[0].x * gridSize, snake[0].y * gridSize, gridSize - 2, gridSize - 2);
    }
    
    drawSnake(snake1, costume1);
    if (gameMode === 'multi') {
      drawSnake(snake2, costume2);
    }
    
    ctx.fillStyle = '#FF0000';
    for (let a of apples) {
      ctx.fillRect(a.x * gridSize + 2, a.y * gridSize + 2, gridSize - 4, gridSize - 4);
    }
    
    ctx.fillStyle = '#FFFF00';
    for (let b of bananas) {
      ctx.fillRect(b.x * gridSize + 2, b.y * gridSize + 2, gridSize - 4, gridSize - 4);
    }
    
    ctx.fillStyle = '#FF00FF';
    for (let enemy of enemies) {
      ctx.beginPath();
      ctx.arc(enemy.x * gridSize + gridSize / 2, enemy.y * gridSize + gridSize / 2, gridSize / 2 - 2, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = '#FF0000';
      ctx.lineWidth = 2;
      ctx.stroke();
    }
    
    ctx.fillStyle = '#FFFFFF';
    ctx.font = '20px Arial';
    ctx.textAlign = 'left';
    ctx.fillText('P1 Score: ' + score1, 10, 30);
    if (gameMode === 'multi') {
      ctx.fillText('P2 Score: ' + score2, 10, 60);
      ctx.fillText('P1 Costume: ' + costume1, 10, 90);
      ctx.fillText('P2 Costume: ' + costume2, 10, 120);
    } else {
      ctx.fillText('Costume: ' + costume1, 10, 60);
    }
  }
  
  function drawGameOver() {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.fillStyle = '#FF0000';
    ctx.font = 'bold 70px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('GAME OVER', canvas.width / 2, canvas.height / 2 - 100);
    
    ctx.fillStyle = '#FFFFFF';
    ctx.font = 'bold 40px Arial';
    if (gameMode === 'multi') {
      ctx.fillText('P1 Final Score: ' + finalScore1, canvas.width / 2, canvas.height / 2 + 30);
      ctx.fillText('P2 Final Score: ' + finalScore2, canvas.width / 2, canvas.height / 2 + 80);
    } else {
      ctx.fillText('Final Score: ' + finalScore1, canvas.width / 2, canvas.height / 2 + 50);
    }
    
    ctx.fillStyle = '#00FF00';
    ctx.font = 'bold 30px Arial';
    ctx.fillText('Press DOWN for Main Menu', canvas.width / 2, canvas.height / 2 + 150);
  }
  
  draw();
})();

