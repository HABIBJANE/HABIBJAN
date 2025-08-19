<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Mines 1Win - Бот бо кушодани 4 ҳуҷра</title>
<style>
  body {
    background-color: #0f0f1c;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 0;
    font-family: 'Segoe UI', sans-serif;
    height: 100vh;
    padding: 20px;
  }

  .top-buttons {
    margin-bottom: 20px;
    display: flex;
    gap: 15px;
  }

  button {
    background-color: #3a60f5;
    border: none;
    color: white;
    font-size: 16px;
    padding: 10px 18px;
    border-radius: 10px;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  button:hover {
    background-color: #2446d4;
  }

  .board {
    display: grid;
    grid-template-columns: repeat(5, 50px);
    grid-template-rows: repeat(5, 50px);
    gap: 8px;
    padding: 15px;
    background: linear-gradient(to bottom right, #1c1c2e, #2e2e47);
    border-radius: 15px;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
  }

  .cell {
    background: linear-gradient(to bottom, #3a60f5, #2446d4);
    border-radius: 10px;
    box-shadow: inset 0 -3px 6px rgba(0, 0, 0, 0.3),
                0 3px 6px rgba(0, 0, 0, 0.4);
    display: flex;
    justify-content: center;
    align-items: center;
    color: white;
    font-size: 18px;
    font-weight: bold;
    cursor: pointer;
    transition: transform 0.15s ease, box-shadow 0.15s ease;
    user-select: none;
  }

  .cell:hover {
    transform: scale(1.05);
    box-shadow: inset 0 -3px 6px rgba(0, 0, 0, 0.3),
                0 4px 10px rgba(0, 0, 0, 0.5);
  }

  .star {
    color: gold;
    font-size: 24px;
    animation: fadeIn 0.7s ease forwards;
  }

  .bottom-controls {
    margin-top: 25px;
    display: flex;
    align-items: center;
    gap: 15px;
  }

  .counter {
    color: white;
    font-size: 28px;
    font-weight: bold;
    width: 50px;
    text-align: center;
    user-select: none;
  }

  @keyframes fadeIn {
    from {opacity: 0; transform: scale(0.8);}
    to {opacity: 1; transform: scale(1);}
  }
</style>
</head>
<body>

  <div class="top-buttons">
    <button id="receiveSignalBtn">получит сигнал</button>
    <button id="newSignalBtn">нови сигнал</button>
  </div>

  <div class="board" id="board">
    <!-- 25 ҳуҷра -->
  </div>

  <div class="bottom-controls">
    <button id="decreaseBtn">-</button>
    <div class="counter" id="countDisplay">1</div>
    <button id="increaseBtn">+</button>
  </div>

  <script>
    const board = document.getElementById('board');
    const receiveSignalBtn = document.getElementById('receiveSignalBtn');
    const newSignalBtn = document.getElementById('newSignalBtn');
    const decreaseBtn = document.getElementById('decreaseBtn');
    const increaseBtn = document.getElementById('increaseBtn');
    const countDisplay = document.getElementById('countDisplay');

    // Эҷоди 25 ҳуҷра
    for(let i = 0; i < 25; i++) {
      const cell = document.createElement('div');
      cell.classList.add('cell');
      board.appendChild(cell);
    }

    const cells = [...document.querySelectorAll('.cell')];
    let starsSet = new Set();

    function setStar(cell, idx) {
      cell.innerHTML = '⭐️';
      cell.classList.add('star');
      cell.style.background = 'transparent';
      cell.style.boxShadow = 'none';
      cell.style.cursor = 'default';
      starsSet.add(idx);
    }

    function resetCell(cell, idx) {
      cell.innerHTML = '';
      cell.classList.remove('star');
      cell.style.background = 'linear-gradient(to bottom, #3a60f5, #2446d4)';
      cell.style.boxShadow = 'inset 0 -3px 6px rgba(0, 0, 0, 0.3), 0 3px 6px rgba(0, 0, 0, 0.4)';
      cell.style.cursor = 'pointer';
      starsSet.delete(idx);
    }

    // Садои мулоим
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function playSoftSound() {
      const oscillator = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();

      oscillator.type = 'sine';
      oscillator.frequency.setValueAtTime(440, audioCtx.currentTime);
      gainNode.gain.setValueAtTime(0, audioCtx.currentTime);

      oscillator.connect(gainNode);
      gainNode.connect(audioCtx.destination);

      gainNode.gain.linearRampToValueAtTime(0.1, audioCtx.currentTime + 0.1);
      gainNode.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 1.4);

      oscillator.start(audioCtx.currentTime);
      oscillator.stop(audioCtx.currentTime + 1.4);
    }

    // Функсияи тасодуфӣ барои интихоби 4 ҳуҷра нодубликат, ки ҳоло ⭐️ нестанд
    function getRandomCells(count) {
      let selected = new Set();
      while(selected.size < count) {
        let randIndex = Math.floor(Math.random() * cells.length);
        if(!starsSet.has(randIndex)) {
          selected.add(randIndex);
        }
      }
      return Array.from(selected);
    }

    // Рақамҳои -+ - танҳо барои нишон додан, бе таъсир ба бозӣ
    const values = [1, 3, 5, 7];
    let currentIndex = 0;

    // "получит сигнал" пахш шуд — 4 ҳуҷра оҳиста ба ⭐️ ва садои мулоим табдил мешавад
    async function receiveSignal() {
      if (receiveSignal.running) return;
      receiveSignal.running = true;

      let toOpenCount = 4;
      let randomIndices = getRandomCells(toOpenCount);

      for(let i = 0; i < randomIndices.length; i++) {
        await new Promise(resolve => setTimeout(resolve, 1400));
        let idx = randomIndices[i];
        setStar(cells[idx], idx);
        playSoftSound();
      }

      receiveSignal.running = false;
    }

    // "нови сигнал" — ҳама ҳуҷраҳоро тоза мекунад
    function newSignal() {
      starsSet.clear();
      cells.forEach(resetCell);
    }

    // Идоракунии тугмаҳои - ва + танҳо барои нишон додани рақам
    decreaseBtn.addEventListener('click', () => {
      currentIndex = (currentIndex - 1 + values.length) % values.length;
      countDisplay.textContent = values[currentIndex];
    });

    increaseBtn.addEventListener('click', () => {
      currentIndex = (currentIndex + 1) % values.length;
      countDisplay.textContent = values[currentIndex];
    });

    receiveSignalBtn.addEventListener('click', receiveSignal);
    newSignalBtn.addEventListener('click', newSignal);

  </script>

</body>
</html>
