<script>
  import { onMount, onDestroy } from 'svelte';
  // If you put the function in a file, import it:
  // import { createPingPong } from './utils.js';

  // Inline the function for simplicity (or import the module above)
  function createPingPong(length, startIndex = 0) {
    let index = Math.max(0, Math.min(startIndex | 0, length - 1));
    let direction = 1;
    return {
      next() {
        if (length === 1) return 0;
        if (index === length - 1) direction = -1;
        else if (index === 0) direction = 1;
        index = index + direction;
        return index;
      },
      get() { return index; }
    };
  }

  export const gifs = ['/riverside.gif', '/gifs/landscape-river.gif'];
  let index = 0;
  const rot = createPingPong(gifs.length, index);
  let timer;

  onMount(() => {
    // show initial image (index already 0)
    timer = setInterval(() => {
      index = rot.next();
    }, 5000); // 5000ms = 5s
  });

  onDestroy(() => {
    clearInterval(timer);
  });
</script>

<!-- simple background div that switches image -->
<div class="bg" style="background-image: url({gifs[index]});"></div>

<style>
  .bg {
    position: fixed;
    inset: 0;
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    z-index: 0;
    pointer-events: none;
  }
</style>