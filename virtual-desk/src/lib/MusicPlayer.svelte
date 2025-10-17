<script>
  import { onMount } from 'svelte';
  export let src = '/music.mp3'; // file will be in public/music.mp3 - Need to input some music
  export let fallback = 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3';

  let audioEl;
  let isPlaying = false;
  let isMuted = true; // start muted to improve chance of autoplay

  // try to play local file; if fails, swap to fallback URL
  onMount(async () => {
    if (!audioEl) return;
    audioEl.loop = true;
    audioEl.muted = isMuted;

    try {
      await audioEl.play();
      isPlaying = true;
    } catch (err) {
      // try fallback URL
      audioEl.src = fallback;
      try {
        await audioEl.play();
        isPlaying = true;
      } catch (err2) {
        isPlaying = false;
      }
    }
  });

  function togglePlay() {
    if (!audioEl) return;
    if (isPlaying) {
      audioEl.pause();
      isPlaying = false;
    } else {
      audioEl.play();
      isPlaying = true;
    }
  }

  function toggleMute() {
    if (!audioEl) return;
    isMuted = !isMuted;
    audioEl.muted = isMuted;
  }
</script>

<div class="music-player" style="position:fixed; right:1rem; bottom:1rem; z-index:5;">
  <!-- svelte-ignore element_invalid_self_closing_tag -->
  <audio bind:this={audioEl} src={src} preload="auto" />
  <button on:click={togglePlay}>{isPlaying ? 'Pause' : 'Play'}</button>
  <button on:click={toggleMute}>{isMuted ? 'Unmute' : 'Mute'}</button>
</div>

<style>
  .music-player button { margin-left: 0.5rem; padding: 0.35rem 0.6rem; }
</style>