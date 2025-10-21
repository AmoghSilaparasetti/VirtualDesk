<script>
  // Wordle-like game (5 letters, 6 tries)
  // Works with keyboard and on-screen keys.

  import { onMount, onDestroy } from 'svelte';

  const WORD_LENGTH = 5;
  const MAX_GUESSES = 6;

  // A tiny built-in dictionary to keep things simple.
  // You can expand this list or import from a JSON file later.
  const WORDS = [
    'APPLE','BERRY','MANGO','LEMON','GRAPE','PEACH','CHILI','BREAD','CHAIR','TABLE',
    'WATER','LIGHT','BLEND','CLOUD','MOUSE','ROBOT','PLANT','BRICK','SMILE','FLOOR',
    'CRANE','TRACE','SLATE','SLICE','PRIDE','GLARE','FRAME','DRIVE','SHINE','THORN'
  ];

  // Choose a word deterministically for the day so it "feels" daily.
  // You can switch to a random word by using Math.random() instead.
  function pickDailyWord() {
    const d = new Date();
    const seed = d.getFullYear()*10000 + (d.getMonth()+1)*100 + d.getDate();
    const idx = seed % WORDS.length;
    return WORDS[idx];
  }

  let solution = pickDailyWord(); // always uppercase in this component

  // Board state
  let rows = Array.from({ length: MAX_GUESSES }, () => Array(WORD_LENGTH).fill(''));
  let colors = Array.from({ length: MAX_GUESSES }, () => Array(WORD_LENGTH).fill('')); // '', 'correct', 'present', 'absent'
  let currentRow = 0;
  let currentCol = 0;
  let message = '';
  let gameOver = false;

  // Keyboard coloring map
  let keyState = {}; // letter -> 'correct' | 'present' | 'absent'

  const alphaRows = [
    ['Q','W','E','R','T','Y','U','I','O','P'],
    ['A','S','D','F','G','H','J','K','L'],
    ['ENTER','Z','X','C','V','B','N','M','⌫']
  ];

  function reset() {
    solution = pickDailyWord();
    rows = Array.from({ length: MAX_GUESSES }, () => Array(WORD_LENGTH).fill(''));
    colors = Array.from({ length: MAX_GUESSES }, () => Array(WORD_LENGTH).fill(''));
    currentRow = 0;
    currentCol = 0;
    message = '';
    gameOver = false;
    keyState = {};
  }

  function handleKey(char) {
    if (gameOver) return;

    if (char === 'ENTER') {
      submitRow();
      return;
    }
    if (char === '⌫') {
      if (currentCol > 0) {
        currentCol -= 1;
        rows[currentRow][currentCol] = '';
      }
      return;
    }

    if (/^[A-Z]$/.test(char)) {
      if (currentCol < WORD_LENGTH) {
        rows[currentRow][currentCol] = char;
        currentCol += 1;
      }
    }
  }

  function onKeydown(e) {
    const k = e.key;
    if (k === 'Enter') handleKey('ENTER');
    else if (k === 'Backspace') handleKey('⌫');
    else if (/^[a-zA-Z]$/.test(k)) handleKey(k.toUpperCase());
  }

  function submitRow() {
    if (currentCol < WORD_LENGTH) {
      message = 'Not enough letters';
      return;
    }

    const guess = rows[currentRow].join('');

    // Optional validity check: allow any 5-letter guess from our WORDS list.
    // If you want to be strict, require WORDS.includes(guess).

    evaluate(guess);

    if (guess === solution) {
      message = 'You got it!';
      gameOver = true;
      return;
    }

    currentRow += 1;
    currentCol = 0;

    if (currentRow >= MAX_GUESSES) {
      message = `The word was ${solution}`;
      gameOver = true;
    }
  }

  function evaluate(guess) {
    // Two-pass algorithm to handle duplicate letters correctly.
    const sol = solution.split('');
    const g = guess.split('');

    const result = Array(WORD_LENGTH).fill('absent');
    const remaining = {};

    // Count remaining letters in solution
    for (let i = 0; i < WORD_LENGTH; i++) {
      const s = sol[i];
      if (!remaining[s]) remaining[s] = 0;
      remaining[s] += 1;
    }

    // First pass: mark exact matches
    for (let i = 0; i < WORD_LENGTH; i++) {
      if (g[i] === sol[i]) {
        result[i] = 'correct';
        remaining[g[i]] -= 1;
      }
    }

    // Second pass: mark presents
    for (let i = 0; i < WORD_LENGTH; i++) {
      if (result[i] === 'correct') continue;
      const ch = g[i];
      if (remaining[ch] > 0) {
        result[i] = 'present';
        remaining[ch] -= 1;
      }
    }

    // Apply to colors and keyboard
    for (let i = 0; i < WORD_LENGTH; i++) {
      colors[currentRow][i] = result[i];
      const ch = g[i];
      const prev = keyState[ch];
      // Upgrade precedence: correct > present > absent
      if (prev === 'correct') continue;
      if (result[i] === 'correct') keyState[ch] = 'correct';
      else if (result[i] === 'present' && prev !== 'correct') keyState[ch] = 'present';
      else if (!prev) keyState[ch] = 'absent';
    }
  }

  onMount(() => {
    window.addEventListener('keydown', onKeydown);
  });

  onDestroy(() => {
    window.removeEventListener('keydown', onKeydown);
  });
</script>

<div class="wordle">
  <div class="top">
    <h2>Wordle</h2>
    <div class="spacer"></div>
    <button class="reset" on:click={reset} aria-label="Reset">Reset</button>
  </div>

  <div class="board" role="grid" aria-label="Wordle board">
    {#each rows as row, r}
      <div class="row" role="row">
        {#each row as cell, c}
          <div class="tile {colors[r][c]}" role="gridcell" aria-live="polite">{cell}</div>
        {/each}
      </div>
    {/each}
  </div>

  {#if message}
    <div class="message" aria-live="polite">{message}</div>
  {/if}

  <div class="keyboard" aria-label="On-screen keyboard">
    {#each alphaRows as r}
      <div class="krow">
        {#each r as label}
          {#if label === 'ENTER'}
            <button class="key enter" on:click={() => handleKey('ENTER')}>ENTER</button>
          {:else if label === '⌫'}
            <button class="key backspace" on:click={() => handleKey('⌫')}>⌫</button>
          {:else}
            <button class="key {keyState[label] || ''}" on:click={() => handleKey(label)}>{label}</button>
          {/if}
        {/each}
      </div>
    {/each}
  </div>
</div>

<style>
  :global(:root) {
    --board-bg: #121213;
    --tile-border: #3a3a3c;
    --tile-bg: #121213;
    --text: #e8e8e8;
    --correct: #538d4e;
    --present: #b59f3b;
    --absent: #3a3a3c;
  }

  .wordle { 
    max-width: 440px; 
    margin: 0 auto; 
    color: var(--text);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif;
  }
  .top { display: flex; align-items: center; gap: 12px; }
  .top h2 { margin: 12px 0; font-weight: 700; letter-spacing: 2px; }
  .top .spacer { flex: 1; }
  .reset { background: transparent; border: 1px solid var(--tile-border); color: var(--text); padding: 6px 10px; border-radius: 6px; cursor: pointer; }

  .board { display: grid; grid-template-rows: repeat(6, 1fr); gap: 6px; background: var(--board-bg); padding: 10px; border-radius: 8px; border: 1px solid var(--tile-border); }
  .row { display: grid; grid-template-columns: repeat(5, 1fr); gap: 6px; }
  .tile { width: 64px; height: 64px; display: grid; place-items: center; border: 2px solid var(--tile-border); text-transform: uppercase; font-weight: 700; font-size: 24px; background: var(--tile-bg); border-radius: 6px; }
  .tile.correct { background: var(--correct); border-color: var(--correct); }
  .tile.present { background: var(--present); border-color: var(--present); }
  .tile.absent { background: var(--absent); border-color: var(--absent); }

  .message { text-align: center; padding: 10px 0; min-height: 24px; }

  .keyboard { margin-top: 8px; user-select: none; }
  .krow { display: flex; justify-content: center; gap: 6px; margin-bottom: 6px; }
  .key { padding: 12px 10px; min-width: 36px; border: none; border-radius: 6px; background: #818384; color: #fff; font-weight: 600; cursor: pointer; }
  .key.correct { background: var(--correct); }
  .key.present { background: var(--present); }
  .key.absent { background: var(--absent); }
  .key.enter { min-width: 64px; }
  .key.backspace { min-width: 44px; }
</style>
