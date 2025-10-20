<script>
  import Menu from './lib/Menu.svelte';
  import MusicPlayer from './lib/MusicPlayer.svelte';
  import NutritionAnalyzer from './lib/NutritionAnalyzer.svelte';

  let active = 'canvas';
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
    {/if}
  </section>

  <MusicPlayer src="/music.mp3" />
</main>