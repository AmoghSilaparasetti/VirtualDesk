<script>
  import BackgroundCycler from './lib/BackgroundCycler.svelte';
  import MusicPlayer from './lib/MusicPlayer.svelte';

  export function createPingPong(length, startIndex = 0) {
    if (!Number.isInteger(length) || length <= 0) {
      throw new Error('length must be a positive integer');
    }
    let index = Math.max(0, Math.min(startIndex | 0, length - 1));
    let direction = 1; // 1 forward, -1 backward

    return {
      next() {
        if (length === 1) return 0;
        // reverse at ends
        if (index === length - 1) direction = -1;
        else if (index === 0) direction = 1;
        index = index + direction;
        return index;
      },
      // optional helpers
      get() { return index; },
      reset(i = 0) { index = Math.max(0, Math.min(i | 0, length - 1)); direction = 1; return index; }
    };
  }
</script>
<main>
  <div class="screen">
    <img src="riverside.gif" alt="riverside" class="screen-image"/>
  </div>
  <BackgroundCycler gifs={['/riverside.gif','/gifs/landscape-river.gif']}/>
  <MusicPlayer src="/music.mp3" />
</main>