<script lang="ts">
  import { onDestroy } from 'svelte';
  import { fade } from 'svelte/transition';

  let hours = 0;
  let minutes = 0;
  let seconds = 0;
  let totalSeconds = 0;
  let intervalId: number;
  let isRunning = false;
  let isPaused = false;
  let showControls = true;
  let almostDone = false;
  let hasCompleted = false;

  function startTimer() {
    if (!totalSeconds) return;
    
    isRunning = true;
    isPaused = false;
    showControls = false;
  
    intervalId = setInterval(() => {
      if (totalSeconds <= 0) {
        clearInterval(intervalId);
        isRunning = false;
        almostDone = false;
        hasCompleted = true;
        showControls = true;
        return;
      }
      
      totalSeconds--;
      if (totalSeconds <= 10) {
        almostDone = true;
      }
      updateDisplay();
    }, 1000);
  }

  function pauseTimer() {
    clearInterval(intervalId);
    isPaused = true;
    isRunning = false;
    showControls = true;
  }

  function resetTimer() {
    clearInterval(intervalId);
    totalSeconds = 0;
    hours = 0;
    minutes = 0;
    seconds = 0;
    isRunning = false;
    isPaused = false;
    almostDone = false;
    hasCompleted = false;
    showControls = true;
  }

  function updateDisplay() {
    hours = Math.floor(totalSeconds / 3600);
    minutes = Math.floor((totalSeconds % 3600) / 60);
    seconds = totalSeconds % 60;
  }

  function setTime() {
    totalSeconds = hours * 3600 + minutes * 60 + seconds;
    hasCompleted = false;
  }

  function toggleControls() {
    if (isRunning) {
      showControls = !showControls;
    }
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === ' ' || event.key === 'Enter') {
      if (!isRunning && !isPaused) {
        startTimer();
      } else if (isPaused) {
        startTimer();
      } else {
        pauseTimer();
      }
    } else if (event.key === 'Escape') {
      resetTimer();
    }
  }

  onDestroy(() => {
    if (intervalId) clearInterval(intervalId);
  });
</script>

<svelte:window on:keydown={handleKeydown}/>

<div class="timer-container" on:click={toggleControls}>
  {#if hasCompleted && totalSeconds === 0 && !showControls}
    <div class="time-up" transition:fade>Time's Up!</div>
  {:else}
    <div class="display" class:almost-done={almostDone}>
      <div class="time">
        {#if hours > 0}
          <span>{hours.toString().padStart(2, '0')}:</span>
        {/if}
        <span>{minutes.toString().padStart(2, '0')}:</span>
        <span>{seconds.toString().padStart(2, '0')}</span>
      </div>
    </div>
  {/if}

  {#if showControls}
    <div class="controls" transition:fade>
      <div class="time-input">
        <div class="input-group">
          <input
            type="number"
            min="0"
            max="23"
            bind:value={hours}
            on:change={setTime}
            placeholder="HH"
          />
          <span>h</span>
        </div>
        <div class="input-group">
          <input
            type="number"
            min="0"
            max="59"
            bind:value={minutes}
            on:change={setTime}
            placeholder="MM"
          />
          <span>m</span>
        </div>
        <div class="input-group">
          <input
            type="number"
            min="0"
            max="59"
            bind:value={seconds}
            on:change={setTime}
            placeholder="SS"
          />
          <span>s</span>
        </div>
      </div>

      <div class="buttons">
        {#if !isRunning}
          <button class="start" on:click={startTimer} disabled={!totalSeconds}>Start</button>
        {:else}
          <button class="pause" on:click={pauseTimer}>Pause</button>
        {/if}
        <button class="reset" on:click={resetTimer}>Reset</button>
      </div>
    </div>
  {/if}
</div>

<style>
  .timer-container {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: #000;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .display {
    padding: 2rem;
    transition: color 0.3s ease;
  }

  .time {
    font-family: 'Courier New', monospace;
    font-size: min(20vw, 20vh);
    font-weight: bold;
    color: #00ff00;
    text-align: center;
    text-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
    line-height: 1;
  }

  .almost-done .time {
    color: #ff3333;
    text-shadow: 0 0 20px rgba(255, 0, 0, 0.8);
    animation: pulse 1s ease-in-out infinite;
  }

  .time-up {
    font-family: 'Courier New', monospace;
    font-size: 15vw;
    font-weight: bold;
    color: #ff3333;
    text-shadow: 0 0 20px rgba(255, 0, 0, 0.8);
    text-align: center;
  }

  .controls {
    position: absolute;
    bottom: 2rem;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.8);
    padding: 2rem;
    border-radius: 1rem;
    backdrop-filter: blur(10px);
  }

  .time-input {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .input-group {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  input {
    background: #333;
    border: none;
    border-radius: 0.25rem;
    color: #fff;
    font-size: 1.5rem;
    padding: 0.5rem;
    width: 4rem;
    text-align: center;
  }

  input::-webkit-inner-spin-button,
  input::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  .buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
  }

  button {
    font-size: 1.2rem;
    padding: 0.8rem 2rem;
    border-radius: 0.5rem;
    border: none;
    cursor: pointer;
    transition: all 0.2s;
    min-width: 120px;
  }

  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .start {
    background: #00aa00;
    color: white;
  }

  .start:hover:not(:disabled) {
    background: #008800;
  }

  .pause {
    background: #aa8800;
    color: white;
  }

  .pause:hover {
    background: #886600;
  }

  .reset {
    background: #aa0000;
    color: white;
  }

  .reset:hover {
    background: #880000;
  }

  @keyframes pulse {
    0% {
      opacity: 1;
      transform: scale(1);
    }
    50% {
      opacity: 0.8;
      transform: scale(1.05);
    }
    100% {
      opacity: 1;
      transform: scale(1);
    }
  }

  @media (max-width: 600px) {
    .time-input {
      flex-direction: column;
      align-items: center;
    }

    .buttons {
      flex-direction: column;
    }

    button {
      width: 100%;
    }
  }
</style>