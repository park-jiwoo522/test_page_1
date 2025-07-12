<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>방탈출: 안전한 물놀이 교실</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Jua&display=swap" rel="stylesheet">

    <style>
        /* --- 기본 스타일 --- */
        :root {
            --primary-color: #0d6efd;
            --dark-color: #212529;
            --light-color: #f8f9fa;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: 'Jua', sans-serif;
            display: flex; justify-content: center; align-items: center;
            min-height: 100vh; background-color: var(--dark-color);
        }

        /* --- 게임 컨테이너 --- */
        #game-container {
            width: 100%; max-width: 1000px; margin: 1rem;
            background: white; border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5); overflow: hidden;
        }
        #main-game, #success-screen { padding: 1rem; }
        #success-screen { text-align: center; padding: 4rem 1rem; }
        #success-screen h1 { color: var(--primary-color); font-size: 2.5rem; }
        #final-time-display { font-size: 1.5rem; color: var(--dark-color); margin-top: 1rem; }

        /* --- 방 이미지 및 클릭 오브젝트 --- */
        #room { position: relative; }
        #room img { width: 100%; display: block; border-radius: 10px; }

        .clickable-object {
            position: absolute; cursor: pointer;
            border: 3px dashed transparent;
            transition: background-color 0.3s, border 0.3s;
            border-radius: 10px;
        }
        @media (hover: hover) {
            .clickable-object:hover {
                background-color: rgba(255, 255, 0, 0.4);
                border-color: red;
            }
        }

        /* ★★★ 이미지에 맞춘 오브젝트 좌표 ★★★ */
        #obj-monitor { top: 41%; left: 35.5%; width: 19%; height: 20%; }
        #obj-projector { top: 8%; left: 63.5%; width: 7%; height: 10%; }
        #obj-clock { top: 9%; left: 74.5%; width: 8%; height: 14%; }
        #obj-pencilcase { top: 62.5%; left: 47.5%; width: 5%; height: 9%; }
        #obj-door { top: 12%; left: 2.5%; width: 22%; height: 85%; }

        /* --- 하단 UI (메시지 & 타이머) --- */
        #bottom-ui {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 1rem;
            gap: 1rem;
        }
        #message-box {
            flex-grow: 1;
            padding: 1rem;
            background-color: var(--light-color);
            text-align: center; border-radius: 10px; border: 1px solid #ddd;
            font-size: 1.2rem;
        }
        #timer-display {
            padding: 1rem;
            background-color: var(--dark-color);
            color: white;
            border-radius: 10px;
            font-size: 1.5rem;
            min-width: 100px;
            text-align: center;
        }

        /* --- 모달 공통 스타일 --- */
        .hidden { display: none !important; }
        .modal-container {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-color: rgba(0, 0, 0, 0.75);
            display: flex; justify-content: center; align-items: center;
            padding: 1rem; z-index: 100;
        }
        .modal-content {
            background: white; padding: 2rem; border-radius: 15px; text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3); width: 90%; max-width: 500px;
        }
        .modal-content h2 { margin-bottom: 1rem; font-size: 1.8rem; color: var(--primary-color); }
        .modal-content p { margin-bottom: 1.5rem; font-size: 1.1rem; line-height: 1.6; }

        /* --- Form 스타일 (사용자 정보 입력) --- */
        #user-info-form { display: flex; flex-direction: column; gap: 1rem; }
        .form-group { text-align: left; }
        .form-group label { font-weight: bold; margin-bottom: 0.5rem; display: block; }
        .form-group input {
            width: 100%; padding: 0.75rem; border-radius: 5px;
            border: 2px solid #ccc; font-family: 'Jua', sans-serif; font-size: 1rem;
        }
        .form-row { display: flex; gap: 1rem; }
        .form-row .form-group { flex: 1; }

        /* --- 메모리 카드 게임 --- */
        #memory-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 0.75rem; }
        .card { width: 100%; aspect-ratio: 1 / 1; perspective: 1000px; cursor: pointer; }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flip .card-inner { transform: rotateY(180deg); }
        .card-front, .card-back {
            position: absolute; width: 100%; height: 100%; backface-visibility: hidden;
            display: flex; justify-content: center; align-items: center;
            border-radius: 10px; font-size: clamp(2rem, 8vw, 3rem);
        }
        .card-back { background-color: var(--primary-color); }
        .card-front { background-color: #e9ecef; transform: rotateY(180deg); }

        /* --- 비밀번호 입력 & 버튼 --- */
        #passcode-area { display: flex; justify-content: center; gap: 0.5rem; }
        #passcode-input {
            padding: 0.75rem; border-radius: 5px; border: 2px solid #ccc;
            width: 180px; text-align: center; font-size: 1.5rem; font-family: 'Jua', sans-serif;
        }
        input[type=number]::-webkit-inner-spin-button { display: none; -moz-appearance: textfield; }

        button {
            padding: 0.75rem 1.5rem; border: none; border-radius: 8px;
            background-color: var(--primary-color); color: white;
            font-size: 1.1rem; font-family: 'Jua', sans-serif; cursor: pointer;
            transition: background-color 0.2s, transform 0.2s;
        }
        button:hover { background-color: #0b5ed7; transform: scale(1.05); }

        /* --- 작은 화면 대응 --- */
        @media (max-width: 600px) {
            #bottom-ui { flex-direction: column; }
            .modal-content { padding: 1.5rem; }
            #passcode-area { flex-direction: column; align-items: center; }
            #passcode-input { width: 100%; margin-bottom: 0.5rem; }
            button { width: 100%; }
        }

        /* --- 애니메이션 --- */
        @keyframes shake {
          0%, 100% { transform: translateX(0); }
          25% { transform: translateX(-5px); }
          75% { transform: translateX(5px); }
        }
    </style>
</head>
<body>
    <!-- 게임 컨테이너 (시작 시 숨김) -->
    <div id="game-container" class="hidden">
        <div id="main-game">
            <div id="room">
                <img src="https://i.ibb.co/R4TxwD7/Whisk-f2c2894a06.jpg" alt="교실 이미지">
                <div id="obj-monitor" class="clickable-object" title="모니터를 확인해보자"></div>
                <div id="obj-projector" class="clickable-object" title="천장의 빔프로젝터를 보자"></div>
                <div id="obj-clock" class="clickable-object" title="시계를 살펴보자"></div>
                <div id="obj-pencilcase" class="clickable-object" title="필통에 무언가 들어있다"></div>
                <div id="obj-door" class="clickable-object" title="이 문으로 탈출해야 한다"></div>
            </div>
            <div id="bottom-ui">
                <div id="message-box">
                    <p>물놀이 안전 수칙을 모두 배우고 교실을 탈출하자!</p>
                </div>
                <div id="timer-display">00:00</div>
            </div>
        </div>
        <div id="success-screen" class="hidden">
            <h1>탈출 성공!</h1>
            <p id="final-time-display">소요 시간: 00:00</p>
        </div>
    </div>

    <!-- 사용자 정보 입력 모달 (시작 시 보임) -->
    <div id="user-info-modal" class="modal-container">
        <div class="modal-content">
            <h2>게임 시작 전 정보 입력</h2>
            <p>방탈출 게임을 시작하기 위해 정보를 입력해주세요.</p>
            <form id="user-info-form">
                <div class="form-group">
                    <label for="user-name">이름</label>
                    <input type="text" id="user-name" placeholder="이름을 입력하세요" required>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label for="user-class">반</label>
                        <input type="text" id="user-class" placeholder="예) 3반" required>
                    </div>
                    <div class="form-group">
                        <label for="user-id">학번</label>
                        <input type="text" id="user-id" placeholder="예) 14번" required>
                    </div>
                </div>
                <button type="submit">게임 시작</button>
            </form>
        </div>
    </div>
    
    <!-- 각종 게임 모달들... -->
    <div id="info-modal" class="modal-container hidden">
        <div class="modal-content">
            <h2 id="info-title"></h2><p id="info-text"></p><button class="close-btn">확인</button>
        </div>
    </div>
    <div id="memory-game-modal" class="modal-container hidden">
        <div class="modal-content">
            <h2>미니게임: 안전 수칙 기억하기</h2><div id="memory-grid"></div><button class="close-btn">그만하기</button>
        </div>
    </div>
    <div id="password-modal" class="modal-container hidden">
        <div class="modal-content">
            <h2>교실 문 잠금장치</h2><p>네 자리 비밀번호를 입력하여 탈출하세요.</p>
            <div id="passcode-area">
                <input type="number" id="passcode-input" placeholder="****" maxlength="4" oninput="this.value=this.value.slice(0,this.maxLength)">
                <button id="escape-btn">탈출</button>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- 1. HTML 요소 가져오기 ---
            const gameContainer = document.getElementById('game-container');
            const messageBox = document.querySelector('#message-box p');
            const mainGame = document.getElementById('main-game');
            const successScreen = document.getElementById('success-screen');
            const timerDisplay = document.getElementById('timer-display');
            const allObjects = {
                monitor: document.getElementById('obj-monitor'), projector: document.getElementById('obj-projector'),
                clock: document.getElementById('obj-clock'), pencilcase: document.getElementById('obj-pencilcase'),
                door: document.getElementById('obj-door'),
            };
            const allModals = {
                userInfo: document.getElementById('user-info-modal'), info: document.getElementById('info-modal'),
                memory: document.getElementById('memory-game-modal'), password: document.getElementById('password-modal'),
            };
            const userInfoForm = document.getElementById('user-info-form');
            const infoTitle = document.getElementById('info-title');
            const infoText = document.getElementById('info-text');
            const memoryGrid = document.getElementById('memory-grid');
            const passcodeInput = document.getElementById('passcode-input');
            const escapeBtn = document.getElementById('escape-btn');
            const finalTimeDisplay = document.getElementById('final-time-display');

            // --- 2. 게임 상태 및 변수 ---
            const FINAL_PASSWORD = "2025";
            const memoryCardIcons = ['🏊', '❤️', '⚠️', '🚑', '🏊', '❤️', '⚠️', '🚑'];
            let isBoardLocked = false;
            let flippedCards = [];
            let timerInterval;
            let startTime;
            let userInfo = {};

            // --- 3. 핵심 기능 함수 ---

            // 타이머 업데이트 함수
            function updateTimer() {
                const elapsedTime = Math.floor((Date.now() - startTime) / 1000);
                const minutes = String(Math.floor(elapsedTime / 60)).padStart(2, '0');
                const seconds = String(elapsedTime % 60).padStart(2, '0');
                timerDisplay.textContent = `${minutes}:${seconds}`;
            }

            // 게임 시작 처리
            function startGame(event) {
                event.preventDefault();

                userInfo = {
                    name: document.getElementById('user-name').value,
                    class: document.getElementById('user-class').value,
                    id: document.getElementById('user-id').value,
                };
                
                // 사용자 정보 모달 숨기고 게임 화면 보이기
                allModals.userInfo.classList.add('hidden');
                gameContainer.classList.remove('hidden');

                // 타이머 시작
                startTime = Date.now();
                timerInterval = setInterval(updateTimer, 1000);

                // 환영 메시지
                messageBox.textContent = `${userInfo.name} 학생, 환영합니다! 교실을 탐험해 안전 수칙을 배우고 탈출하세요.`;
            }

            // 정보 모달 열기
            function showInfoModal(title, text) {
                infoTitle.textContent = title;
                infoText.innerHTML = text;
                allModals.info.classList.remove('hidden');
            }

            // 메모리 카드 게임 시작
            function startMemoryGame() {
                memoryGrid.innerHTML = ''; isBoardLocked = false; flippedCards = [];
                let shuffledIcons = [...memoryCardIcons].sort(() => 0.5 - Math.random());
                shuffledIcons.forEach(icon => {
                    const card = document.createElement('div'); card.classList.add('card'); card.dataset.icon = icon;
                    card.innerHTML = `<div class="card-inner"><div class="card-back"></div><div class="card-front">${icon}</div></div>`;
                    card.addEventListener('click', handleCardClick); memoryGrid.appendChild(card);
                });
                allModals.memory.classList.remove('hidden');
            }

            // 카드 클릭 처리
            function handleCardClick() {
                if (isBoardLocked || this.classList.contains('flip')) return;
                this.classList.add('flip'); flippedCards.push(this);
                if (flippedCards.length === 2) {
                    isBoardLocked = true; const [card1, card2] = flippedCards;
                    if (card1.dataset.icon === card2.dataset.icon) {
                        card1.removeEventListener('click', handleCardClick); card2.removeEventListener('click', handleCardClick);
                        flippedCards = []; isBoardLocked = false;
                        if (document.querySelectorAll('.card.flip').length === memoryCardIcons.length) {
                            setTimeout(() => {
                                alert("미니게임 성공! '수영 가능' 구역과 '안전요원'이 있는 곳에서 물놀이해야 한다는 사실을 배웠다.");
                                allModals.memory.classList.add('hidden');
                            }, 500);
                        }
                    } else {
                        setTimeout(() => {
                            card1.classList.remove('flip'); card2.classList.remove('flip');
                            flippedCards = []; isBoardLocked = false;
                        }, 1000);
                    }
                }
            }

            // 탈출 시도
            function attemptEscape() {
                if (passcodeInput.value === FINAL_PASSWORD) {
                    // 타이머 정지
                    clearInterval(timerInterval);
                    const finalTime = timerDisplay.textContent;

                    // 화면 전환
                    mainGame.classList.add('hidden');
                    allModals.password.classList.add('hidden');
                    successScreen.classList.remove('hidden');
                    finalTimeDisplay.textContent = `소요 시간: ${finalTime}`;

                    // ★★★ 정보 저장 ★★★
                    const record = { ...userInfo, time: finalTime };
                    
                    // 1. 콘솔에 기록 (개발자 확인용)
                    console.log("=== 탈출 성공 기록 ===", record);

                    // 2. 로컬 저장소에 기록 (데이터 보존용)
                    // 키를 '기록_날짜'로 하여 중복 방지
                    localStorage.setItem(`escape_record_${Date.now()}`, JSON.stringify(record));
                    alert("탈출 성공! 기록이 저장되었습니다.");
                } else {
                    passcodeInput.style.animation = 'shake 0.5s';
                    setTimeout(() => { passcodeInput.style.animation = '' }, 500);
                    passcodeInput.value = ''; passcodeInput.focus();
                }
            }

            // --- 4. 이벤트 리스너 설정 ---
            userInfoForm.addEventListener('submit', startGame);
            allObjects.monitor.addEventListener('click', () => {
                showInfoModal('문제 1: 올바른 준비운동', '물놀이 전, 체온 유지를 위해 충분한 준비운동은 필수입니다. 권장되는 최소 준비운동 시간은 몇 분일까요? <br><br><b>정답에 2를 곱하면 비밀번호의 첫 두 자리가 됩니다.</b> (힌트: 10분)');
                messageBox.textContent = "모니터 문제를 확인했다. (정답: 10분 x 2 = 20)";
            });
            allObjects.clock.addEventListener('click', () => {
                showInfoModal('문제 2: 안전한 수온', '차가운 물에 갑자기 들어가면 심장마비의 위험이 있습니다. 안전한 수영을 위한 적정 수온은 약 몇 도(°C)일까요? <br><br><b>정답이 비밀번호의 마지막 두 자리가 됩니다.</b> (힌트: 25~26도)');
                messageBox.textContent = "시계 문제를 확인했다. (정답: 25도)";
            });
            allObjects.pencilcase.addEventListener('click', () => {
                showInfoModal('힌트 발견!', '필통 안 쪽지: <br><br><b>"모니터 문제의 답과 시계 문제의 답을 순서대로 합치면..."</b>');
            });
            allObjects.projector.addEventListener('click', startMemoryGame);
            allObjects.door.addEventListener('click', () => {
                allModals.password.classList.remove('hidden'); passcodeInput.focus();
            });
            document.querySelectorAll('.close-btn').forEach(btn => {
                btn.addEventListener('click', () => btn.closest('.modal-container').classList.add('hidden'));
            });
            escapeBtn.addEventListener('click', attemptEscape);
            passcodeInput.addEventListener('keyup', (e) => {
                if (e.key === 'Enter') attemptEscape();
            });
        });
    </script>
</body>
</html>
