<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let items = [
    { id: 'canvas', label: 'Canvas'},
    { id: 'game', label: 'Simple Game'},
    { id: 'foodai', label: 'Nutrition Analyzer'},
    { id: 'music', label: 'Music Player'}
  ];

  let active = items[0].id;

  function select(id){
    active = id;
    dispatch('select', { id });
  }
</script>

<nav class="menu" aria-label="Main navigation">
  <ul>
    {#each items as it}
      <li>
        <button
          class:active={it.id === active}
          on:click={() => select(it.id)}
          aria-pressed={it.id === active}
        >{it.label}</button>
      </li>
    {/each}
  </ul>
</nav>

<style>
  .menu {
    background-color: black;
    margin: -16px;
    border-radius: 5px;
  }
  .menu ul{ list-style:none; margin:-10px; padding:0; display:flex; gap:0.5rem; }
  .menu button{
    padding:0.5rem 0.75rem; border-radius:6px; border:1px solid rgba(0,0,0,0.12);
    background: rgba(255,255,255,0.06); color: #fff; cursor:pointer;
  }
  .menu button.active{ background: rgba(255,255,255,0.12); box-shadow: 0 1px 0 rgba(0,0,0,0.15) inset; }
  .menu button:focus{ outline: 2px solid rgba(255,255,255,0.18); }
</style>
