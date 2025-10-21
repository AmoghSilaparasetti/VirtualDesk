<script>
  import Menu from './lib/Menu.svelte';
  import MusicPlayer from './lib/MusicPlayer.svelte';
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
  function onSelect(e){ active = e.detail.id }
</script>

<main>
  <!-- menu -->
  <header style="position:relative; z-index: 3; padding:1rem;">
    <Menu on:select={onSelect} />
  </header>

  <!-- screen -->
  <div class="screen">
    <img src="landscape-river.gif" alt="landscape-river" class="screen-image"/>
  </div>

  <!-- overlay content area -->
  <section class="app-content" style="position:relative; z-index:2; padding:1rem;">
    {#if active === 'canvas'}
      <div class="panel"> <h2>Canvas</h2> <p>Placeholder for canvas drawing feature.</p> </div>
    {:else if active === 'game'}
      <div class="panel"> <h2>Simple Game</h2> <p>Placeholder for a simple chess game.</p> </div>
    {:else if active === 'foodai'}
      <NutritionAnalyzer/>
    {:else if active === 'music'}
      <div class="panel">
        <h2>Music Player</h2>
        <div style="margin-top:1rem;">
          <MusicPlayer {tracks} />
        </div>
      </div>
    {/if}
  </section>

</main>