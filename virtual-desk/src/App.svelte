<script>
  import Menu from './lib/Menu.svelte';
  import MusicPlayer from './lib/MusicPlayer.svelte';
  import Canvas from './lib/Canvas.svelte';
  import NutritionAnalyzer from './lib/NutritionAnalyzer.svelte';
  import track1 from './music/track1.mp3';
  import track2 from './music/track2.mp3';
  import track3 from './music/track3.mp3';
  import cover1 from './music/cover1.jpg';
  import cover2 from './music/cover2.jpg';
  import cover3 from './music/cover3.jpg';

  let active = 'canvas';
  let tracks = [
    { id: '1', title: 'Chill Morning', artist: 'Unknown', src: track1, cover: cover1 },
    { id: '2', title: 'Christmas Time', artist: 'Unknown', src: track2, cover: cover2 },
    { id: '3', title: 'Tokyo Drift', artist: 'Unknown', src: track3, cover: cover3 }
  ];
  function onSelect(e) {
    active = e.detail.id;
  }
</script>

<main>
  <!-- 🏞️ Background -->
  <div class="background">
    <img src="landscape-river.gif" alt="landscape river" class="background-image" />
  </div>

  <!-- 🧭 Menu -->
  <header class="menu-bar">
    <Menu on:select={onSelect} />
  </header>

  <!-- 🎨 Canvas area (centered and enlarged) -->
  {#if active === 'canvas'}
    <section class="canvas-wrapper">
      <Canvas />
    </section>
  {/if}

  <!-- 🧠 Other modules -->
  {#if active === 'game'}
    <div class="panel"><h2>Simple Game</h2><p>Placeholder for a simple chess game.</p></div>
  {:else if active === 'foodai'}
    <div class="panel"><h2>Nutrition Analyzer</h2><p>Placeholder for Nutrition Analyzer controls.</p></div>
  {/if}

  <!-- 🎵 Music -->
</main>

<style>
  main {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    font-family: system-ui, sans-serif;
    color: white;
  }

  /* 🏞 Background */
  .background {
    position: absolute;
    inset: 0;
    z-index: 0;
  }

  .background-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  /* 🧭 Menu */
  .menu-bar {
    position: absolute;
    top: 1rem;
    left: 1rem;
    z-index: 3;
  }

  /* 🎨 Canvas centered */
  .canvas-wrapper {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 2;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 950px;
    height: 600px;
    background: rgba(255, 255, 255, 0.18);
    border-radius: 16px;
    border: 2px solid rgba(255, 255, 255, 0.25);
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.4);
    backdrop-filter: blur(8px) brightness(1.1);
    overflow: hidden;
  }

  /* 🧠 Info Panels */
  .panel {
    position: absolute;
    top: 7rem;
    left: 2rem;
    background: rgba(0, 0, 0, 0.55);
    padding: 1rem 1.5rem;
    border-radius: 10px;
    backdrop-filter: blur(8px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  }
</style>
