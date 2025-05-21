<script>
  import { onMount, createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  const ROWS = 15;
  const COLS = 15;

  const walls = [
    [3,3],[3,4],[3,5],[3,6],[3,7],[3,8],
    [6,2],[7,2],[8,2],
    [6,12],[7,12],[8,12],
    [10,5],[10,6],[10,7],
    [5,10],[6,10],[7,10]
  ];

  let map;
  let pacman;
  let ghosts;
  let score;
  let gameOver;
  let direction;
  let mouthTimer;

  function initGame() {
    map = Array(ROWS).fill(null).map(() =>
      Array(COLS).fill(2)
    );

    for(let r = 0; r < ROWS; r++) {
      for(let c = 0; c < COLS; c++) {
        if (r === 0 || r === ROWS -1 || c === 0 || c === COLS -1) {
          map[r][c] = 1;
        }
      }
    }

    walls.forEach(([r,c]) => map[r][c] = 1);

    pacman = { r: 1, c: 1, mouthOpen: true };
    ghosts = [
      { r: ROWS - 2, c: COLS - 2, dir: 'left' },
      { r: ROWS - 2, c: 1, dir: 'right' }
    ];
    score = 0;
    gameOver = false;
    direction = 'right';
  }

  function canMove(r, c) {
    return r >= 0 && r < ROWS && c >= 0 && c < COLS && map[r][c] !== 1;
  }

  function movePacman() {
    if(!direction) return;
    let newR = pacman.r;
    let newC = pacman.c;

    if(direction === 'up') newR--;
    else if(direction === 'down') newR++;
    else if(direction === 'left') newC--;
    else if(direction === 'right') newC++;

    if(canMove(newR, newC)) {
      pacman.r = newR;
      pacman.c = newC;

      if(map[newR][newC] === 2) {
        score++;
        map[newR][newC] = 0;
      }
    }
  }

  function moveGhosts() {
    ghosts = ghosts.map(g => {
      const moves = ['up', 'down', 'left', 'right'];
      let possibleMoves = moves.filter(move => {
        let r = g.r, c = g.c;
        if(move === 'up') r--;
        else if(move === 'down') r++;
        else if(move === 'left') c--;
        else if(move === 'right') c++;
        return canMove(r,c);
      });

      if(possibleMoves.length === 0) return g;

      if(possibleMoves.includes(g.dir)) {
        if(Math.random() < 0.7) {
          let r = g.r, c = g.c;
          if(g.dir === 'up') r--;
          else if(g.dir === 'down') r++;
          else if(g.dir === 'left') c--;
          else if(g.dir === 'right') c++;
          return {...g, r, c};
        }
      }

      let newDir = possibleMoves[Math.floor(Math.random() * possibleMoves.length)];
      let r = g.r, c = g.c;
      if(newDir === 'up') r--;
      else if(newDir === 'down') r++;
      else if(newDir === 'left') c--;
      else if(newDir === 'right') c++;
      return {...g, r, c, dir: newDir};
    });
  }

  function checkCollision() {
    for(const g of ghosts) {
      if(g.r === pacman.r && g.c === pacman.c) {
        gameOver = true;
      }
    }
  }

  function checkWin() {
    if (map.flat().every(cell => cell !== 2)) {
      gameOver = true;
    }
  }

  function restartGame() {
    initGame();
  }

  function goBack() {
    dispatch('back');
  }

  onMount(() => {
    initGame();

    function handleKey(e) {
      if(gameOver) return;
      if(e.key === 'ArrowUp') direction = 'up';
      else if(e.key === 'ArrowDown') direction = 'down';
      else if(e.key === 'ArrowLeft') direction = 'left';
      else if(e.key === 'ArrowRight') direction = 'right';
    }
    window.addEventListener('keydown', handleKey);

    const gameInterval = setInterval(() => {
      if(!gameOver) {
        movePacman();
        moveGhosts();
        checkCollision();
        checkWin();
      }
    }, 150);

    mouthTimer = setInterval(() => {
      pacman.mouthOpen = !pacman.mouthOpen;
    }, 300);

    return () => {
      window.removeEventListener('keydown', handleKey);
      clearInterval(gameInterval);
      clearInterval(mouthTimer);
    };
  });
</script>

<button on:click={goBack} class="mb-4 bg-gray-300 px-3 py-1 rounded hover:bg-gray-400">
  ⬅ zurück
</button>

<style>
  .grid {
    display: grid;
    grid-template-columns: repeat(15, 30px);
    grid-template-rows: repeat(15, 30px);
    gap: 1px;
    margin-bottom: 20px;
  }
  .cell {
    width: 30px;
    height: 30px;
    background: black;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .wall {
    background: #222;
  }
  .dot {
    background: black;
    color: yellow;
    font-size: 12px;
  }
  .ghost {
    color: red;
    font-size: 24px;
  }
  .pacman {
    font-size: 28px;
    user-select: none;
    transition: transform 0.2s;
  }
  .pacman.open::before {
    content: '😮';
    display: block;
  }
  .pacman.closed::before {
    content: '🟡';
    display: block;
  }
  .pacman.up { transform: rotate(-90deg); }
  .pacman.down { transform: rotate(90deg); }
  .pacman.left { transform: rotate(180deg); }
  .pacman.right { transform: rotate(0deg); }
  button {
    font-size: 18px;
    padding: 10px 20px;
    margin-top: 10px;
  }
</style>

<h2>Punkte: {score}</h2>

<div class="grid">
  {#each map as row, r}
    {#each row as cell, c}
      <div class="cell {cell === 1 ? 'wall' : cell === 2 ? 'dot' : ''}">
        {#if pacman.r === r && pacman.c === c}
          <span class="pacman {pacman.mouthOpen ? 'open' : 'closed'} {direction}" />
        {:else if ghosts.find(g => g.r === r && g.c === c)}
          <span class="ghost">👻</span>
        {:else if cell === 2}
          <span style="color: yellow">.</span>
        {/if}
      </div>
    {/each}
  {/each}
</div>

{#if gameOver}
  <div style="font-size: 24px;">
    {map.flat().every(cell => cell !== 2) ? '🏆 Du hast gewonnen!' : '💀 Game Over'}
  </div>
  <button on:click={restartGame}>🔁 Neu starten</button>
{/if}
