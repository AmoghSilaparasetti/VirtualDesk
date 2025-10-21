<script>
// @ts-nocheck

  import { onMount, onDestroy } from 'svelte';

  export let tracks = [];

  let audio;
  let currentIndex = 0;
  let isPlaying = false;
  let isMuted = false;
  let volume = 0.8;
  let currentTime = 0;
  let duration = 0;
  let loop = false;
  let shuffle = false;

  // load and optionally play a track by index
  function loadTrack(index, autoplay = true) {
    if (!tracks || tracks.length === 0) return;
    currentIndex = ((index % tracks.length) + tracks.length) % tracks.length;
    audio.src = tracks[currentIndex].src;
    audio.load();
    if (autoplay) {
      audio.play().then(() => { isPlaying = true; }).catch(() => { isPlaying = false; });
    }
  }

  function togglePlay() {
    if (!audio) return;
    if (isPlaying) {
      audio.pause();
      isPlaying = false;
    } else {
      audio.play().then(() => isPlaying = true).catch(() => isPlaying = false);
    }
  }

  function playIndex(i) {
    if (i === currentIndex) {
      togglePlay();
      return;
    }
    loadTrack(i, true);
  }

  function next() {
    if (!tracks || tracks.length === 0) return;
    if (shuffle) {
      playIndex(Math.floor(Math.random() * tracks.length));
      return;
    }
    const nextIndex = (currentIndex + 1) % tracks.length;
    loadTrack(nextIndex, true);
  }

  function prev() {
    if (!tracks || tracks.length === 0) return;
    const prevIndex = (currentIndex - 1 + tracks.length) % tracks.length;
    loadTrack(prevIndex, true);
  }

  function onTimeUpdate() {
    currentTime = audio.currentTime || 0;
  }

  function onLoadedMetadata() {
    duration = audio.duration || 0;
  }

  function seekTo(percent) {
    if (!audio || !duration) return;
    audio.currentTime = percent * duration;
  }

  function toggleMute() {
    isMuted = !isMuted;
    if (audio) audio.muted = isMuted;
  }

  function setVolume(v) {
    volume = v;
    if (audio) audio.volume = v;
  }

  // keyboard shortcuts (space = play/pause, n = next, p = prev)
  function onKey(e) {
    if (e.code === 'Space') { e.preventDefault(); togglePlay(); }
    if (e.key === 'n') next();
    if (e.key === 'p') prev();
  }

  function onEnded() {
    isPlaying = false;
    if (!loop) next();
  }

  onMount(() => {
    audio = new Audio();
    audio.preload = 'auto';
    audio.volume = volume;
    audio.muted = isMuted;
    audio.addEventListener('timeupdate', onTimeUpdate);
    audio.addEventListener('loadedmetadata', onLoadedMetadata);
    audio.addEventListener('ended', onEnded);
    // load first track but do not autoplay
    loadTrack(0, false);
    window.addEventListener('keydown', onKey);
  });

  onDestroy(() => {
    if (!audio) return;
    audio.pause();
    audio.src = '';
    audio.removeEventListener('timeupdate', onTimeUpdate);
    audio.removeEventListener('loadedmetadata', onLoadedMetadata);
    audio.removeEventListener('ended', onEnded);
    window.removeEventListener('keydown', onKey);
  });

  // helpers for UI formatting
  function formatTime(sec = 0) {
    const s = Math.floor(sec % 60).toString().padStart(2, '0');
    const m = Math.floor(sec / 60).toString();
    return `${m}:${s}`;
  }

  $: progress = duration ? currentTime / duration : 0;
</script>

<div class="music-library">
  <aside class="library-list" aria-label="Music library">
    <h3>Library</h3>
    <ul>
      {#each tracks as t, i}
        <li class:active={i === currentIndex}>
          <button on:click={() => playIndex(i)} aria-pressed={i === currentIndex && isPlaying}>
            <div class="meta">
              <span class="title">{t.title}</span>
              <span class="artist">{t.artist}</span>
            </div>
            <div class="controls-small">
              {#if i === currentIndex}
                <small>{isPlaying ? 'Playing' : 'Paused'}</small>
              {/if}
            </div>
          </button>
        </li>
      {/each}
    </ul>
  </aside>

  <section class="player" aria-label="Player">
    <div class="now">
      <img class="cover" src={tracks[currentIndex]?.cover} alt="cover"/>
      <div class="info">
        <div class="title">{tracks[currentIndex]?.title}</div>
        <div class="artist">{tracks[currentIndex]?.artist}</div>
      </div>
    </div>

    <div class="progress-row">
      <span class="time">{formatTime(currentTime)}</span>
      <input class="progress" type="range" min="0" max="1" step="0.001" value={progress}
        on:input={(e) => seekTo(+e.target.value)} />
      <span class="time">{formatTime(duration)}</span>
    </div>

    <div class="controls">
      <button on:click={prev} title="Previous">⏮</button>
      <button on:click={togglePlay} title="Play/Pause">{isPlaying ? '⏸' : '▶'}</button>
      <button on:click={next} title="Next">⏭</button>

      <div class="extras">
  <button on:click={toggleMute} title="Mute/Unmute">{isMuted ? '🔇' : '🔊'}</button>
  <input type="range" min="0" max="1" step="0.01" bind:value={volume} on:input={(e)=>setVolume(+e.target.value)} />
      </div>
    </div>
  </section>
</div>

<style>
  .music-library { display:flex; gap:1rem; align-items:flex-start; color:#fff; font-family:system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; }
  .library-list { width:260px; background:rgba(0,0,0,0.35); padding:0.75rem; border-radius:8px; }
  .library-list h3 { margin:0 0 0.5rem 0; font-size:0.95rem; }
  .library-list ul { list-style:none; margin:0; padding:0; display:flex; flex-direction:column; gap:0.35rem; }
  .library-list li button { width:100%; display:flex; justify-content:space-between; padding:0.45rem; border-radius:6px; background:transparent; border:none; color:inherit; cursor:pointer; text-align:left; }
  .library-list li.active button { background: rgba(255,255,255,0.06); }
  .meta { display:flex; flex-direction:column; }
  .meta .title { font-weight:600; font-size:0.95rem; }
  .meta .artist { font-size:0.8rem; opacity:0.8; }

  .player { flex:1; background: rgba(0,0,0,0.32); padding:0.75rem; border-radius:8px; display:flex; flex-direction:column; gap:0.6rem; min-width:320px; }
  .now { display:flex; gap:0.7rem; align-items:center; }
  .cover { width:72px; height:72px; object-fit:cover; border-radius:6px; background:linear-gradient(180deg,#222,#111); }
  .info .title { font-size:1.05rem; font-weight:700; }
  .info .artist { font-size:0.85rem; opacity:0.85; margin-top:0.15rem; }

  .progress-row { display:flex; align-items:center; gap:0.5rem; }
  .progress { -webkit-appearance: none; appearance:none; width:100%; background:transparent; }
  .progress::-webkit-slider-runnable-track { height:6px; background:rgba(255,255,255,0.12); border-radius:6px; }
  .progress::-webkit-slider-thumb { -webkit-appearance:none; width:12px; height:12px; background:#fff; border-radius:50%; margin-top:-3px; }
  .time { font-size:0.8rem; width:40px; text-align:center; opacity:0.9; }

  .controls { display:flex; align-items:center; gap:0.6rem; }
  .controls button { padding:0.4rem 0.6rem; border-radius:6px; border:1px solid rgba(255,255,255,0.08); background:rgba(255,255,255,0.03); color:#fff; cursor:pointer; }
  .extras { margin-left:auto; display:flex; align-items:center; gap:0.5rem; }
  .extras input[type="range"] { width:110px; }
  .active { box-shadow: inset 0 -2px 0 0 rgba(255,255,255,0.06); background:rgba(255,255,255,0.06); }
</style>