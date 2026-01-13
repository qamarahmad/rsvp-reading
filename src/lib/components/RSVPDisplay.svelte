<script>
  import { getActualORPIndex } from '../rsvp-utils.js';

  export let word = '';
  export let opacity = 1;
  export let fadeDuration = 150;
  export let fadeEnabled = true;

  $: orpIndex = getActualORPIndex(word);
  $: beforeOrp = word ? word.slice(0, orpIndex) : '';
  $: orpLetter = word ? (word[orpIndex] || '') : '';
  $: afterOrp = word ? word.slice(orpIndex + 1) : '';
</script>

<div class="rsvp-display">
  <div class="focus-marker">
    <div class="marker-line top"></div>
    <div class="marker-line bottom"></div>
  </div>

  <div
    class="word-container"
    style="opacity: {opacity}; transition: opacity {fadeEnabled ? fadeDuration : 0}ms ease-in-out;"
  >
    {#if word}
      <!-- ORP letter is absolutely positioned at center -->
      <span class="orp">{orpLetter}</span>
      <!-- Before text positioned to the left of center -->
      <span class="before-orp">{beforeOrp}</span>
      <!-- After text positioned to the right of center -->
      <span class="after-orp">{afterOrp}</span>
    {:else}
      <span class="placeholder">Ready</span>
    {/if}
  </div>
</div>

<style>
  .rsvp-display {
    position: relative;
    width: 100%;
    height: 100%;
    min-height: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex: 1;
    overflow: hidden;
  }

  .focus-marker {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    height: 100%;
    width: 3px;
    pointer-events: none;
    z-index: 10;
  }

  .marker-line {
    position: absolute;
    left: 0;
    width: 100%;
    height: 50px;
  }

  .marker-line.top {
    top: 0;
    background: linear-gradient(to bottom, #ff4444, transparent);
  }

  .marker-line.bottom {
    bottom: 0;
    background: linear-gradient(to top, #ff4444, transparent);
  }

  .word-container {
    position: relative;
    font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', 'Source Code Pro', 'Menlo', 'Consolas', monospace;
    font-size: clamp(3rem, 8vw, 6rem);
    font-weight: 500;
    line-height: 1;
    white-space: nowrap;
    text-rendering: geometricPrecision;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    /* Container needs width for absolute children to position against */
    width: 100%;
    height: 1.2em;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .orp {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    color: #ff4444;
    font-weight: 700;
    text-shadow: 0 0 30px rgba(255, 68, 68, 0.6);
    z-index: 2;
  }

  .before-orp {
    position: absolute;
    right: calc(50% + 0.5ch);
    color: #fff;
    text-align: right;
  }

  .after-orp {
    position: absolute;
    left: calc(50% + 0.5ch);
    color: #fff;
    text-align: left;
  }

  .placeholder {
    color: #333;
    font-size: 2rem;
    font-weight: 300;
    font-family: system-ui, sans-serif;
    line-height: 1;
  }

  @media (max-width: 600px) {
    .rsvp-display {
      min-height: 200px;
    }

    .marker-line {
      height: 30px;
    }
  }
</style>
