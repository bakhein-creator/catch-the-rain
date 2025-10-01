<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>빗물 받기 게임</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 폰트 설정 및 배경 */
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
            padding: 20px; /* 여백 추가 */
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


        /* 게임 영역: 반응성 개선 핵심 */
        #game-area {
            position: relative;
            width: 90vw; /* 뷰포트 너비의 90% 사용 */
            max-width: 900px; /* 데스크톱에서 최대 너비 900px 허용 */
            height: 80vh; /* 뷰포트 높이의 80% 사용 */
            max-height: 800px; /* 최대 높이 제한 */
            
            background: rgba(255, 255, 255, 0.5);
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4); /* 그림자 진하게 */
            overflow: hidden;
            border: 5px solid #4a90e2; /* 하늘색 테두리 */
        }


        /* 점수판 */
        #score-board-container {
            position: absolute;
            top: 15px;
            right: 15px;
            display: flex;
            gap: 10px;
            z-index: 10;
        }


        .stat-box {
            background: rgba(255, 255, 255, 0.9);
            padding: 8px 15px;
            border-radius: 12px;
            box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
            font-size: 1.2rem;
            color: #333;
            font-weight: bold;
        }


        /* 빗방울 스타일 */
        .raindrop {
            position: absolute;
            width: 20px;
            height: 20px;
            background: #4a90e2;
            border-radius: 50% 50% 50% 0;
            transform: rotate(-45deg);
            animation: fall linear infinite;
        }


        @keyframes fall {
            from { top: -20px; }
            to { top: 100%; }
        }


        /* 바구니 스타일 */
        #basket {
            position: absolute;
            bottom: 20px;
            width: 100px;
            height: 50px;
            background: #6a4c3e;
            border-radius: 10px 10px 50px 50px / 5px 5px 25px 25px;
            box-shadow: inset 0 -5px 10px rgba(0, 0, 0, 0.4), 0 5px 20px rgba(0, 0, 0, 0.4);
            transition: width 0.3s ease-in-out, background-color 0.1s;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0.8rem;
            color: white;
            z-index: 5;
            cursor: move; /* 이동 가능함을 표시 */
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
            box-shadow: inset 0 -3px 5px rgba(0, 0, 0, 0.4);
        }
        
        /* 게임 시작 버튼 및 메시지 */
        #start-button {
            padding: 12px 25px;
            font-size: 1.8rem;
            border-radius: 20px;
            background-color: #007bff; /* 파란색으로 변경 */
            color: white;
            border: none;
            cursor: pointer;
            box-shadow: 0 8px 15px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s ease, background-color 0.2s;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
        }
        
        #start-button:hover:not(:disabled) {
            transform: translateY(-5px);
            background-color: #0056b3;
        }
        
        #start-button:disabled {
            cursor: not-allowed;
            opacity: 0.7;
        }
        
        /* 게임 오버 메시지 */
        #game-message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 3rem;
            color: #d9534f;
            text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);
            z-index: 20;
            animation: pulse 1s infinite alternate;
            background: rgba(255, 255, 255, 0.9);
            padding: 20px 40px;
            border-radius: 15px;
            border: 5px dashed #d9534f;
            display: none;
        }

        @keyframes pulse {
            from { transform: translate(-50%, -50%) scale(1); opacity: 1; }
            to { transform: translate(-50%, -50%) scale(1.05); opacity: 0.9; }
        }


    </style>
</head>
<body class="bg-gray-100 flex flex-col items-center justify-center h-screen">


    <div id="game-container" class="relative w-full h-full flex flex-col items-center justify-center">
        <div id="game-area" class="relative">
            <div id="score-board-container">
                <div class="stat-box">물방울: <span id="score">0</span></div>
                <div class="stat-box bg-red-100 text-red-600">놓침: <span id="missed">0</span> / 10</div>
            </div>
            <div id="basket">바구니</div>
            <div id="game-message">GAME OVER</div>
        </div>
        <button id="start-button" class="mt-8">게임 시작</button>
    </div>


    <script>
        const gameArea = document.getElementById('game-area');
        const basket = document.getElementById('basket');
        const scoreDisplay = document.getElementById('score');
        const missedDisplay = document.getElementById('missed');
        const startButton = document.getElementById('start-button');
        const gameMessage = document.getElementById('game-message');
        
        const MAX_MISSED = 10; // 놓칠 수 있는 최대 빗방울 개수
        let score = 0;
        let missedCount = 0;
        let isGameRunning = false;
        let gameLoopTimeout;
        const collisionIntervals = []; // 충돌 확인 인터벌을 저장할 배열


        // 바구니 크기 변화 관련 설정
        let basketSizeIndex = 0;
        const initialBasketWidth = 100;


        // --- 게임 상태 관리 함수 ---


        function resetGame() {
            // 모든 빗방울과 인터벌 제거
            document.querySelectorAll('.raindrop').forEach(drop => drop.remove());
            collisionIntervals.forEach(clearInterval);
            collisionIntervals.length = 0; // 배열 비우기

            score = 0;
            missedCount = 0;
            basketSizeIndex = 0;

            // UI 초기화
            updateScore();
            updateMissedCount();
            basket.style.width = `${initialBasketWidth}px`;
            gameMessage.style.display = 'none';

            startButton.textContent = "게임 시작";
            startButton.disabled = false;
        }

        function startGame() {
            if (isGameRunning) return;

            resetGame(); // 게임 재시작 시 초기화
            isGameRunning = true;
            startButton.textContent = "게임 중...";
            startButton.disabled = true;
            gameLoop();
        }

        function gameOver() {
            isGameRunning = false;
            clearTimeout(gameLoopTimeout);
            collisionIntervals.forEach(clearInterval);
            collisionIntervals.length = 0;
            
            startButton.textContent = "다시 시작";
            startButton.disabled = false;

            gameMessage.textContent = `GAME OVER! 점수: ${score}점`;
            gameMessage.style.display = 'block';
        }


        // --- UI 업데이트 함수 ---

        function updateScore() {
            scoreDisplay.textContent = score;
            
            // 10점마다 바구니 크기 증가 (최대 크기까지)
            if (score > 0 && score % 10 === 0 && basket.offsetWidth < 200) {
                // 바구니 너비 최대 200px까지 20px씩 증가
                const newWidth = initialBasketWidth + (Math.floor(score / 10) * 20);
                basket.style.width = `${Math.min(newWidth, 200)}px`;
            }
        }

        function updateMissedCount() {
            missedDisplay.textContent = `${missedCount} / ${MAX_MISSED}`;
            if (missedCount >= MAX_MISSED) {
                gameOver();
            }
        }


        // --- 빗방울 생성 및 충돌 로직 ---

        function createRaindrop() {
            if (!isGameRunning) return;

            const raindrop = document.createElement('div');
            raindrop.classList.add('raindrop');
            
            const areaWidth = gameArea.clientWidth;
            const startX = Math.random() * (areaWidth - 20); // 빗방울 너비 20px 고려
            raindrop.style.left = `${startX}px`;
            
            const fallDuration = Math.random() * 2 + 3; // 3~5 seconds
            raindrop.style.animationDuration = `${fallDuration}s`;
            
            gameArea.appendChild(raindrop);
            
            // 빗방울이 떨어지는 동안 지속적으로 충돌을 확인합니다.
            const collisionCheckInterval = setInterval(() => {
                if (!isGameRunning) {
                    clearInterval(collisionCheckInterval);
                    return;
                }

                const basketRect = basket.getBoundingClientRect();
                const raindropRect = raindrop.getBoundingClientRect();
                const gameAreaRect = gameArea.getBoundingClientRect();


                // 충돌 감지 (빗방울의 바닥이 바구니의 윗부분에 닿고, 좌우 위치가 겹칠 때)
                if (
                    raindropRect.bottom >= basketRect.top + 5 && // 5px 오차 허용
                    raindropRect.bottom <= basketRect.bottom + 5 && // 바구니 바닥을 넘기 전에 잡기
                    raindropRect.left < basketRect.right - 10 && // 바구니 테두리 안쪽으로 잡기
                    raindropRect.right > basketRect.left + 10
                ) {
                    score++;
                    updateScore();
                    basket.style.backgroundColor = '#4CAF50'; // 잡았을 때 초록색 반짝
                    setTimeout(() => basket.style.backgroundColor = '#6a4c3e', 100);
                    clearInterval(collisionCheckInterval);
                    raindrop.remove();
                }

                // 빗방울이 바구니를 놓치고 게임 영역 아래로 벗어난 경우 (놓침 처리)
                if (raindropRect.top > gameAreaRect.bottom) {
                    missedCount++;
                    updateMissedCount();
                    clearInterval(collisionCheckInterval);
                    raindrop.remove();
                }

            }, 20); // 20ms마다 충돌을 확인

            collisionIntervals.push(collisionCheckInterval);
        }


        function gameLoop() {
            if (isGameRunning) {
                createRaindrop();
                // 난이도 조절: 시간이 지날수록 빗방울 생성 간격 줄이기
                const baseDelay = 500;
                const minDelay = 200;
                // 점수가 높을수록 딜레이를 줄여서 난이도 증가
                const delayReduction = Math.min(score * 5, baseDelay - minDelay);
                const delay = Math.random() * 500 + (baseDelay - delayReduction); 

                gameLoopTimeout = setTimeout(gameLoop, delay);
            }
        }


        // --- 이벤트 리스너 ---

        // 마우스 이동 (데스크톱)
        gameArea.addEventListener('mousemove', (e) => {
            if (isGameRunning) {
                // gameArea 내부에서의 마우스 상대 X 좌표
                const gameAreaRect = gameArea.getBoundingClientRect();
                const basketWidth = basket.offsetWidth;
                
                // 마우스 포인터의 X 위치를 기준으로 바구니 중앙을 맞춥니다.
                let newX = e.clientX - gameAreaRect.left - (basketWidth / 2);
                
                // 경계선 처리
                if (newX < 0) newX = 0;
                if (newX > gameAreaRect.width - basketWidth) newX = gameAreaRect.width - basketWidth;
                
                // transform 대신 left 속성을 사용하고, CSS에서 transform: translateX(-50%)를 제거하여 정확한 위치 설정
                basket.style.left = `${newX}px`;
            }
        });


        // 터치 이동 (모바일)
        gameArea.addEventListener('touchmove', (e) => {
            if (isGameRunning) {
                const touch = e.touches[0];
                const gameAreaRect = gameArea.getBoundingClientRect();
                const basketWidth = basket.offsetWidth;
                
                // 터치 포인터의 X 위치를 기준으로 바구니 중앙을 맞춥니다.
                let newX = touch.clientX - gameAreaRect.left - (basketWidth / 2);

                // 경계선 처리
                if (newX < 0) newX = 0;
                if (newX > gameAreaRect.width - basketWidth) newX = gameAreaRect.width - basketWidth;

                basket.style.left = `${newX}px`;
                e.preventDefault(); // 스크롤 방지
            }
        }, { passive: false });


        startButton.addEventListener('click', startGame);

        // 초기 설정
        resetGame();

    </script>
</body>
</html>
