<!DOCTYPE html>
<html lang="ko">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>빗물 받기 게임</title>
   <script src="https://cdn.tailwindcss.com"></script>
   <style>
       @import url('https://fonts.googleapis.com/css2?family=Jua&display=swap');
       body {
           font-family: 'Jua', sans-serif;
           background: linear-gradient(to bottom, #87ceeb, #a2d2ff);
           overflow: hidden;
           height: 100vh;
           display: flex;
           justify-content: center;
           align-items: center;
           flex-direction: column;
       }


       #game-container {
           position: relative;
           width: 100%;
           height: 100%;
           display: flex;
           justify-content: center;
           align-items: center;
           flex-direction: column;
       }


       #game-area {
           position: relative;
           width: 80%;
           max-width: 600px;
           height: 70vh;
           background: rgba(255, 255, 255, 0.5);
           border-radius: 20px;
           box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
           overflow: hidden;
       }


       #score-board {
           position: absolute;
           top: 20px;
           right: 20px;
           background: rgba(255, 255, 255, 0.7);
           padding: 10px 20px;
           border-radius: 15px;
           box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
           font-size: 1.5rem;
           color: #333;
           z-index: 10;
       }


       .raindrop {
           position: absolute;
           width: 20px;
           height: 20px;
           background: #4a90e2;
           border-radius: 50% 50% 50% 0;
           transform: rotate(-45deg);
           animation: fall linear infinite;
       }


       .raindrop::before {
           content: '';
           position: absolute;
           width: 100%;
           height: 100%;
           background: #4a90e2;
           border-radius: 50% 50% 50% 0;
           opacity: 0.5;
           top: 5px;
           left: 5px;
           filter: blur(5px);
       }


       @keyframes fall {
           from { top: -20px; }
           to { top: 100%; }
       }


       #basket {
           position: absolute;
           bottom: 20px;
           width: 100px;
           height: 50px;
           background: #6a4c3e;
           border-radius: 10px 10px 50px 50px / 5px 5px 25px 25px;
           box-shadow: inset 0 -5px 10px rgba(0, 0, 0, 0.3), 0 5px 15px rgba(0, 0, 0, 0.3);
           transition: width 0.3s ease-in-out;
           left: 50%;
           transform: translateX(-50%);
           display: flex;
           justify-content: center;
           align-items: center;
           font-size: 0.8rem;
           color: white;
           z-index: 5;
       }
      
       #basket::before {
           content: '';
           position: absolute;
           top: -10px;
           left: 5px;
           right: 5px;
           height: 15px;
           background: #8e6c5c;
           border-radius: 50%;
           filter: blur(2px);
           box-shadow: inset 0 -3px 5px rgba(0, 0, 0, 0.3);
       }


       #start-button {
           padding: 10px 20px;
           font-size: 1.5rem;
           border-radius: 15px;
           background-color: #4CAF50;
           color: white;
           border: none;
           cursor: pointer;
           box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
           transition: transform 0.2s ease;
       }
      
       #start-button:hover {
           transform: translateY(-3px);
       }


   </style>
</head>
<body class="bg-gray-100 flex flex-col items-center justify-center h-screen">


   <div id="game-container" class="relative w-full h-full flex flex-col items-center justify-center">
       <div id="game-area" class="relative">
           <div id="score-board">물방울: <span id="score">0</span></div>
           <div id="basket">바구니</div>
       </div>
       <button id="start-button" class="mt-8">게임 시작</button>
   </div>


   <script>
       const gameArea = document.getElementById('game-area');
       const basket = document.getElementById('basket');
       const scoreDisplay = document.getElementById('score');
       const startButton = document.getElementById('start-button');
      
       let score = 0;
       let isGameRunning = false;
       let basketSizeIndex = 0;
       const basketSizes = [100, 120, 140, 160, 180, 200];
       const initialBasketWidth = 100;


       function updateScore() {
           scoreDisplay.textContent = score;
           if (score > 0 && score % 10 === 0 && basketSizeIndex < basketSizes.length - 1) {
               basketSizeIndex++;
               basket.style.width = `${initialBasketWidth + (basketSizeIndex * 20)}px`;
           }
       }


       function createRaindrop() {
           if (!isGameRunning) return;


           const raindrop = document.createElement('div');
           raindrop.classList.add('raindrop');
          
           const startX = Math.random() * (gameArea.clientWidth - 20);
           raindrop.style.left = `${startX}px`;
          
           const fallDuration = Math.random() * 2 + 3; // 3~5 seconds
           raindrop.style.animationDuration = `${fallDuration}s`;
          
           gameArea.appendChild(raindrop);
          
           // 빗방울이 떨어지는 동안 지속적으로 충돌을 확인합니다.
           const collisionCheckInterval = setInterval(() => {
               // 게임이 멈췄으면 확인을 중단합니다.
               if (!isGameRunning) {
                   clearInterval(collisionCheckInterval);
                   return;
               }


               const basketRect = basket.getBoundingClientRect();
               const raindropRect = raindrop.getBoundingClientRect();


               // 충돌 감지 로직: 빗방울의 바닥이 바구니의 윗부분에 닿고, 좌우 위치가 겹칠 때
               if (
                   raindropRect.bottom >= basketRect.top &&
                   raindropRect.left < basketRect.right &&
                   raindropRect.right > basketRect.left
               ) {
                   score++;
                   updateScore();
                   clearInterval(collisionCheckInterval); // 충돌 후 확인 중단
                   raindrop.remove(); // 빗방울 제거
               }


               // 빗방울이 바구니를 놓치고 화면 아래로 벗어난 경우
               if (raindropRect.top > gameArea.clientHeight) {
                   clearInterval(collisionCheckInterval); // 확인 중단
                   raindrop.remove(); // 빗방울 제거
               }
           }, 20); // 20ms마다 충돌을 확인합니다.
       }


       function gameLoop() {
           if (isGameRunning) {
               createRaindrop();
               setTimeout(gameLoop, Math.random() * 500 + 500); // Create a new raindrop every 0.5~1 seconds
           }
       }


       // Mouse movement for the basket
       document.addEventListener('mousemove', (e) => {
           if (isGameRunning) {
               const gameAreaRect = gameArea.getBoundingClientRect();
               const basketWidth = basket.offsetWidth;
               let newX = e.clientX - gameAreaRect.left - (basketWidth / 2);
              
               // Keep the basket within the game area boundaries
               if (newX < 0) {
                   newX = 0;
               }
               if (newX > gameAreaRect.width - basketWidth) {
                   newX = gameAreaRect.width - basketWidth;
               }
              
               basket.style.left = `${newX}px`;
           }
       });


       // Touch movement for the basket
       document.addEventListener('touchmove', (e) => {
           if (isGameRunning) {
               const touch = e.touches[0];
               const gameAreaRect = gameArea.getBoundingClientRect();
               const basketWidth = basket.offsetWidth;
               let newX = touch.clientX - gameAreaRect.left - (basketWidth / 2);


               if (newX < 0) {
                   newX = 0;
               }
               if (newX > gameAreaRect.width - basketWidth) {
                   newX = gameAreaRect.width - basketWidth;
               }


               basket.style.left = `${newX}px`;
               e.preventDefault();
           }
       }, { passive: false });




       startButton.addEventListener('click', () => {
           if (!isGameRunning) {
               isGameRunning = true;
               startButton.textContent = "게임 중...";
               startButton.disabled = true;
               gameLoop();
           }
       });
   </script>
</body>
</html>



