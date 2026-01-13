<script>
  import { onMount, onDestroy } from 'svelte';
  import {
    parseText as parseTextUtil,
    getActualORPIndex as getActualORPIndexUtil,
    getWordDelay as getWordDelayUtil,
    formatTimeRemaining,
    shouldPauseAtWord
  } from './lib/rsvp-utils.js';

  // State
  let text = `Rapid serial visual presentation (RSVP) is a scientific method for studying the timing of vision. In RSVP, a sequence of stimuli is shown to an observer at one location in their visual field. The observer is instructed to report one of these stimuli - the target - which has a feature that differentiates it from the rest of the stream. This technique has also been adapted for speed reading applications, where words are displayed one at a time at a fixed point, eliminating the need for eye movements and potentially increasing reading speed.`;
  let words = [];
  let currentWordIndex = 0;
  let isPlaying = false;
  let isPaused = false;
  let showSettings = false;
  let showTextInput = false;
  let progress = 0;

  // Settings
  let wordsPerMinute = 300;
  let fadeEnabled = true;
  let fadeDuration = 150; // ms
  let pauseAfterWords = 0; // 0 = disabled
  let pauseDuration = 500; // ms
  let pauseOnPunctuation = true;
  let punctuationPauseMultiplier = 2;

  // Animation
  let wordOpacity = 1;
  let intervalId = null;
  let fadeTimeoutId = null;

  // Parse text into words
  function parseText() {
    words = parseTextUtil(text);
    currentWordIndex = 0;
    progress = 0;
  }

  // Get actual ORP index for current word
  function getActualORPIndex(word) {
    return getActualORPIndexUtil(word);
  }

  // Calculate delay for current word
  function getWordDelay(word) {
    return getWordDelayUtil(word, wordsPerMinute, pauseOnPunctuation, punctuationPauseMultiplier);
  }

  // Show next word with fade effect
  function showNextWord() {
    if (currentWordIndex >= words.length) {
      stop();
      return;
    }

    // Check for pause after x words
    if (shouldPauseAtWord(currentWordIndex, pauseAfterWords)) {
      isPaused = true;
      setTimeout(() => {
        if (isPlaying) {
          isPaused = false;
          scheduleNextWord();
        }
      }, pauseDuration);
      return;
    }

    if (fadeEnabled) {
      wordOpacity = 0;
      fadeTimeoutId = setTimeout(() => {
        wordOpacity = 1;
      }, 10);
    }

    progress = ((currentWordIndex + 1) / words.length) * 100;
    currentWordIndex++;
    scheduleNextWord();
  }

  function scheduleNextWord() {
    if (!isPlaying || currentWordIndex >= words.length) return;

    const currentWord = words[currentWordIndex - 1] || '';
    const delay = getWordDelay(currentWord);

    intervalId = setTimeout(showNextWord, delay);
  }

  // Control functions
  function start() {
    if (words.length === 0) parseText();
    if (words.length === 0) return;

    isPlaying = true;
    isPaused = false;
    showNextWord();
  }

  function pause() {
    isPlaying = false;
    isPaused = true;
    if (intervalId) {
      clearTimeout(intervalId);
      intervalId = null;
    }
  }

  function resume() {
    if (currentWordIndex < words.length) {
      isPlaying = true;
      isPaused = false;
      scheduleNextWord();
    }
  }

  function stop() {
    isPlaying = false;
    isPaused = false;
    currentWordIndex = 0;
    progress = 0;
    wordOpacity = 1;
    if (intervalId) {
      clearTimeout(intervalId);
      intervalId = null;
    }
  }

  function restart() {
    stop();
    start();
  }

  // Handle text change
  function handleTextChange() {
    stop();
    parseText();
  }

  // Format time remaining
  function getTimeRemaining() {
    const remainingWords = words.length - currentWordIndex;
    return formatTimeRemaining(remainingWords, wordsPerMinute);
  }

  // Keyboard shortcuts
  function handleKeydown(e) {
    if (e.target.tagName === 'TEXTAREA' || e.target.tagName === 'INPUT') return;

    switch(e.code) {
      case 'Space':
        e.preventDefault();
        if (isPlaying) pause();
        else if (isPaused) resume();
        else start();
        break;
      case 'Escape':
        stop();
        break;
      case 'ArrowUp':
        e.preventDefault();
        wordsPerMinute = Math.min(1000, wordsPerMinute + 25);
        break;
      case 'ArrowDown':
        e.preventDefault();
        wordsPerMinute = Math.max(50, wordsPerMinute - 25);
        break;
      case 'ArrowLeft':
        e.preventDefault();
        if (currentWordIndex > 1) {
          currentWordIndex = Math.max(0, currentWordIndex - 2);
          progress = (currentWordIndex / words.length) * 100;
        }
        break;
      case 'ArrowRight':
        e.preventDefault();
        if (currentWordIndex < words.length) {
          progress = ((currentWordIndex + 1) / words.length) * 100;
          currentWordIndex++;
        }
        break;
    }
  }

  onMount(() => {
    parseText();
    window.addEventListener('keydown', handleKeydown);
  });

  onDestroy(() => {
    if (intervalId) clearTimeout(intervalId);
    if (fadeTimeoutId) clearTimeout(fadeTimeoutId);
    window.removeEventListener('keydown', handleKeydown);
  });

  $: currentWord = words[currentWordIndex - 1] || (words.length > 0 ? words[0] : '');
  $: orpIndex = getActualORPIndex(currentWord);
</script>

<main>
  <div class="container">
    <!-- Header -->
    <header>
      <h1>RSVP Reader</h1>
      <div class="header-buttons">
        <button class="icon-btn" on:click={() => showTextInput = !showTextInput} title="Set Text">
          📝
        </button>
        <button class="icon-btn" on:click={() => showSettings = !showSettings} title="Settings">
          ⚙️
        </button>
      </div>
    </header>

    <!-- Text Input Panel -->
    {#if showTextInput}
      <div class="panel text-input-panel">
        <h3>Enter Text</h3>
        <textarea
          bind:value={text}
          placeholder="Paste or type your text here..."
          rows="6"
        ></textarea>
        <div class="panel-buttons">
          <button on:click={handleTextChange}>Apply</button>
          <button on:click={() => showTextInput = false}>Close</button>
        </div>
      </div>
    {/if}

    <!-- Settings Panel -->
    {#if showSettings}
      <div class="panel settings-panel">
        <h3>Settings</h3>

        <div class="setting">
          <label>
            <span>Words Per Minute: {wordsPerMinute}</span>
            <input type="range" min="50" max="1000" step="25" bind:value={wordsPerMinute}>
          </label>
        </div>

        <div class="setting">
          <label>
            <input type="checkbox" bind:checked={fadeEnabled}>
            <span>Enable Fade Effect</span>
          </label>
        </div>

        {#if fadeEnabled}
          <div class="setting sub-setting">
            <label>
              <span>Fade Duration: {fadeDuration}ms</span>
              <input type="range" min="50" max="300" step="25" bind:value={fadeDuration}>
            </label>
          </div>
        {/if}

        <div class="setting">
          <label>
            <input type="checkbox" bind:checked={pauseOnPunctuation}>
            <span>Pause on Punctuation</span>
          </label>
        </div>

        {#if pauseOnPunctuation}
          <div class="setting sub-setting">
            <label>
              <span>Punctuation Pause Multiplier: {punctuationPauseMultiplier}x</span>
              <input type="range" min="1" max="4" step="0.5" bind:value={punctuationPauseMultiplier}>
            </label>
          </div>
        {/if}

        <div class="setting">
          <label>
            <span>Pause Every N Words: {pauseAfterWords === 0 ? 'Disabled' : pauseAfterWords}</span>
            <input type="range" min="0" max="50" step="5" bind:value={pauseAfterWords}>
          </label>
        </div>

        {#if pauseAfterWords > 0}
          <div class="setting sub-setting">
            <label>
              <span>Pause Duration: {pauseDuration}ms</span>
              <input type="range" min="100" max="2000" step="100" bind:value={pauseDuration}>
            </label>
          </div>
        {/if}

        <button class="close-btn" on:click={() => showSettings = false}>Close</button>
      </div>
    {/if}

    <!-- RSVP Display -->
    <div class="rsvp-display">
      <div class="focus-lines">
        <div class="focus-line top"></div>
        <div class="focus-line bottom"></div>
      </div>
      <div
        class="word-container"
        style="opacity: {wordOpacity}; transition: opacity {fadeEnabled ? fadeDuration : 0}ms ease-in-out;"
      >
        {#if currentWord}
          <span class="word">
            <span class="before-orp">{currentWord.slice(0, orpIndex)}</span>
            <span class="orp">{currentWord[orpIndex] || ''}</span>
            <span class="after-orp">{currentWord.slice(orpIndex + 1)}</span>
          </span>
        {:else}
          <span class="placeholder">Press Play to Start</span>
        {/if}
      </div>
    </div>

    <!-- Progress Bar -->
    <div class="progress-container">
      <div class="progress-bar" style="width: {progress}%"></div>
    </div>

    <!-- Stats -->
    <div class="stats">
      <span>{currentWordIndex} / {words.length} words</span>
      <span>{wordsPerMinute} WPM</span>
      <span>~{getTimeRemaining()} remaining</span>
    </div>

    <!-- Controls -->
    <div class="controls">
      {#if !isPlaying && !isPaused}
        <button class="control-btn play" on:click={start} disabled={words.length === 0}>
          ▶ Play
        </button>
      {:else if isPlaying}
        <button class="control-btn pause" on:click={pause}>
          ⏸ Pause
        </button>
      {:else}
        <button class="control-btn play" on:click={resume}>
          ▶ Resume
        </button>
      {/if}

      <button class="control-btn stop" on:click={stop} disabled={!isPlaying && !isPaused}>
        ⏹ Stop
      </button>

      <button class="control-btn restart" on:click={restart} disabled={words.length === 0}>
        ↺ Restart
      </button>
    </div>

    <!-- Keyboard Shortcuts Info -->
    <div class="shortcuts">
      <p>Keyboard: <kbd>Space</kbd> Play/Pause | <kbd>Esc</kbd> Stop | <kbd>↑↓</kbd> Speed | <kbd>←→</kbd> Skip</p>
    </div>
  </div>
</main>

<style>
  :global(body) {
    background-color: #000 !important;
    margin: 0;
    padding: 0;
  }

  main {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #000;
    color: #fff;
    font-family: 'Segoe UI', system-ui, sans-serif;
  }

  .container {
    width: 100%;
    max-width: 900px;
    padding: 2rem;
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
  }

  h1 {
    font-size: 1.5rem;
    font-weight: 300;
    color: #888;
    margin: 0;
  }

  .header-buttons {
    display: flex;
    gap: 0.5rem;
  }

  .icon-btn {
    background: transparent;
    border: 1px solid #333;
    color: #888;
    font-size: 1.2rem;
    padding: 0.5rem 0.75rem;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .icon-btn:hover {
    border-color: #666;
    color: #fff;
  }

  /* Panels */
  .panel {
    background: #111;
    border: 1px solid #333;
    border-radius: 12px;
    padding: 1.5rem;
    margin-bottom: 2rem;
  }

  .panel h3 {
    margin: 0 0 1rem;
    font-weight: 400;
    color: #aaa;
  }

  .text-input-panel textarea {
    width: 100%;
    background: #1a1a1a;
    border: 1px solid #333;
    border-radius: 8px;
    color: #fff;
    padding: 1rem;
    font-size: 1rem;
    font-family: inherit;
    resize: vertical;
    box-sizing: border-box;
  }

  .text-input-panel textarea:focus {
    outline: none;
    border-color: #666;
  }

  .panel-buttons {
    display: flex;
    gap: 0.5rem;
    margin-top: 1rem;
  }

  .panel-buttons button {
    flex: 1;
    padding: 0.75rem;
    background: #222;
    border: 1px solid #444;
    color: #fff;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .panel-buttons button:first-child {
    background: #ff4444;
    border-color: #ff4444;
  }

  .panel-buttons button:hover {
    opacity: 0.8;
  }

  /* Settings */
  .setting {
    margin-bottom: 1rem;
  }

  .setting label {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    color: #ccc;
  }

  .setting input[type="range"] {
    flex: 1;
    height: 6px;
    background: #333;
    border-radius: 3px;
    appearance: none;
    cursor: pointer;
  }

  .setting input[type="range"]::-webkit-slider-thumb {
    appearance: none;
    width: 16px;
    height: 16px;
    background: #ff4444;
    border-radius: 50%;
    cursor: pointer;
  }

  .setting input[type="checkbox"] {
    width: 18px;
    height: 18px;
    accent-color: #ff4444;
  }

  .sub-setting {
    padding-left: 1.5rem;
    border-left: 2px solid #333;
    margin-left: 0.5rem;
  }

  .close-btn {
    width: 100%;
    padding: 0.75rem;
    background: #333;
    border: none;
    color: #fff;
    border-radius: 8px;
    cursor: pointer;
    margin-top: 0.5rem;
  }

  .close-btn:hover {
    background: #444;
  }

  /* RSVP Display */
  .rsvp-display {
    position: relative;
    height: 200px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 2rem 0;
  }

  .focus-lines {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    width: 4px;
    height: 100%;
    pointer-events: none;
  }

  .focus-line {
    position: absolute;
    left: 0;
    width: 100%;
    height: 30px;
    background: linear-gradient(to bottom, transparent, #ff4444);
  }

  .focus-line.top {
    top: 0;
    background: linear-gradient(to bottom, #ff4444, transparent);
  }

  .focus-line.bottom {
    bottom: 0;
    background: linear-gradient(to top, #ff4444, transparent);
  }

  .word-container {
    font-size: 4rem;
    font-weight: 600;
    font-family: 'Georgia', serif;
    letter-spacing: 2px;
    white-space: nowrap;
    position: relative;
  }

  .word {
    display: inline-flex;
  }

  .before-orp {
    color: #fff;
    text-align: right;
    min-width: 150px;
    display: inline-block;
  }

  .orp {
    color: #ff4444;
    font-weight: 700;
    text-shadow: 0 0 20px rgba(255, 68, 68, 0.5);
  }

  .after-orp {
    color: #fff;
    text-align: left;
    min-width: 150px;
    display: inline-block;
  }

  .placeholder {
    color: #444;
    font-size: 1.5rem;
    font-weight: 300;
    font-family: 'Segoe UI', system-ui, sans-serif;
  }

  /* Progress */
  .progress-container {
    height: 4px;
    background: #222;
    border-radius: 2px;
    overflow: hidden;
    margin-bottom: 1rem;
  }

  .progress-bar {
    height: 100%;
    background: linear-gradient(90deg, #ff4444, #ff6666);
    transition: width 0.1s linear;
  }

  /* Stats */
  .stats {
    display: flex;
    justify-content: space-between;
    color: #666;
    font-size: 0.9rem;
    margin-bottom: 2rem;
  }

  /* Controls */
  .controls {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .control-btn {
    padding: 1rem 2rem;
    font-size: 1rem;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
    font-weight: 500;
  }

  .control-btn:disabled {
    opacity: 0.3;
    cursor: not-allowed;
  }

  .control-btn.play {
    background: #ff4444;
    color: #fff;
  }

  .control-btn.play:hover:not(:disabled) {
    background: #ff6666;
  }

  .control-btn.pause {
    background: #ffaa00;
    color: #000;
  }

  .control-btn.pause:hover {
    background: #ffcc44;
  }

  .control-btn.stop {
    background: #333;
    color: #fff;
  }

  .control-btn.stop:hover:not(:disabled) {
    background: #444;
  }

  .control-btn.restart {
    background: #333;
    color: #fff;
  }

  .control-btn.restart:hover:not(:disabled) {
    background: #444;
  }

  /* Shortcuts */
  .shortcuts {
    text-align: center;
    color: #444;
    font-size: 0.85rem;
  }

  .shortcuts p {
    margin: 0;
  }

  kbd {
    background: #222;
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    font-family: monospace;
    color: #888;
  }
</style>
