<script>
  import { onMount } from 'svelte';

  let score = 0;
  let highscore = localStorage.getItem("highscore") || 0;
  let moleIndex = -1;
  let gameRunning = false;
  let timeLeft = 30;
  let speed = 1000;

  let timer;  // für den Timer-Interval
  let moleMover; // für den Mole-Bewegungs-Timer

  function startGame(mode) {
    score = 0;
    timeLeft = 30;
    gameRunning = true;

    if (mode === "easy") speed = 1200;
    if (mode === "medium") speed = 800;
    if (mode === "hard") speed = 400;

    moveMole();
    startTimer();
  }

  function moveMole() {
    if (!gameRunning) return;

    moleIndex = Math.floor(Math.random() * 9);
    clearTimeout(moleMover);
    moleMover = setTimeout(moveMole, speed);
  }

  function bonk(index) {
    if (!gameRunning) return;
    if (index === moleIndex) {
      score++;
      moleIndex = -1;
    }
  }

  function startTimer() {
    clearInterval(timer);
    timer = setInterval(() => {
      timeLeft--;
      if (timeLeft <= 0) {
        clearInterval(timer);
        clearTimeout(moleMover);
        gameRunning = false;
        moleIndex = -1;

        if (score > highscore) {
          highscore = score;
          localStorage.setItem("highscore", highscore);
        }
      }
    }, 1000);
  }

  onMount(() => {
    return () => {
      clearInterval(timer);
      clearTimeout(moleMover);
    }
  });
</script>

<h1>Quak-Mohle 🐸</h1>
<h2>Punkte: {score}</h2>
<h2>Rekord: {highscore}</h2>
<div class="timer">🕒 Zeit: {timeLeft} Sek</div>

{#if !gameRunning}
  <p>Wähle den Modus:</p>
  <button on:click={() => startGame("easy")}>Langsam 🐢</button>
  <button on:click={() => startGame("medium")}>Mittel 🐸</button>
  <button on:click={() => startGame("hard")}>Schnell 🐇</button>
{/if}

<div class="grid">
  {#each Array(9) as _, index}
    <div class="hole {index === moleIndex ? 'mole' : ''}" on:click={() => bonk(index)}>
      {index === moleIndex ? '🐸' : ''}
    </div>
  {/each}
</div>

<style>
  .grid {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    gap: 10px;
    justify-content: center;
    margin-top: 20px;
  }
  .hole {
    width: 100px;
    height: 100px;
    background-color: #ccc;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    cursor: pointer;
    font-size: 24px;
    user-select: none;
  }
  .mole {
    background-color: green;
    color: white;
  }
  button {
    margin: 5px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }
  .timer {
    font-size: 18px;
    margin-top: 10px;
  }
</style>
