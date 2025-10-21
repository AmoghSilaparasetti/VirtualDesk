<script>
  import { onMount } from 'svelte';

  // Canvas state
  let canvas;
  let ctx;
  let drawing = false;
  let color = '#ffcc00';
  let lineWidth = 4;

  // Load/save canvas
  function loadCanvas() {
    const data = localStorage.getItem('virtualDeskCanvas');
    if (data) {
      const img = new Image();
      img.src = data;
      img.onload = () => ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
    }
  }

  function saveCanvas() {
    localStorage.setItem('virtualDeskCanvas', canvas.toDataURL());
  }

  function startDraw(e) {
    drawing = true;
    draw(e);
  }

  function endDraw() {
    drawing = false;
    ctx.beginPath();
    saveCanvas();
  }

  function draw(e) {
    if (!drawing) return;
    const rect = canvas.getBoundingClientRect();
    ctx.lineWidth = lineWidth;
    ctx.lineCap = 'round';
    ctx.strokeStyle = color;
    ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(e.clientX - rect.left, e.clientY - rect.top);
  }

  function clearCanvas() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    saveCanvas();
  }

  // Sticky notes
  let notes = [];
  let nextId = 1;

  function loadNotes() {
    const stored = localStorage.getItem('virtualDeskNotes');
    if (stored) {
      notes = JSON.parse(stored);
      nextId = notes.reduce((max, n) => Math.max(max, n.id), 0) + 1;
    }
  }

  function saveNotes() {
    localStorage.setItem('virtualDeskNotes', JSON.stringify(notes));
  }

  function addNote() {
    const colors = ['#fff8a0', '#a0f0ff', '#ffd0f0', '#c0ffc0'];
    const note = {
      id: nextId++,
      text: '',
      x: 100,
      y: 100,
      height: 50,
      color: colors[Math.floor(Math.random() * colors.length)]
    };
    notes = [...notes, note];
    saveNotes();
  }

  function deleteNote(id) {
    notes = notes.filter((n) => n.id !== id);
    saveNotes();
  }

  // Drag notes
  function startDrag(e, note) {
    const offsetX = e.clientX - note.x;
    const offsetY = e.clientY - note.y;

    function onMove(ev) {
      note.x = ev.clientX - offsetX;
      note.y = ev.clientY - offsetY;
    }

    function onUp() {
      saveNotes();
      window.removeEventListener('mousemove', onMove);
      window.removeEventListener('mouseup', onUp);
    }

    window.addEventListener('mousemove', onMove);
    window.addEventListener('mouseup', onUp);
  }

  // Auto-resize textarea
  function autoResize(e, note) {
    e.target.style.height = 'auto';
    e.target.style.height = e.target.scrollHeight + 'px';
    note.height = e.target.scrollHeight;
    saveNotes();
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    canvas.width = canvas.clientWidth;
    canvas.height = canvas.clientHeight;

    loadCanvas();
    loadNotes();
  });

</script>

<!-- Toolbar -->
<div class="toolbar">
  <label>🎨 Color: <input type="color" bind:value={color} /></label>
  <label>✏️ Size: <input type="range" min="1" max="15" bind:value={lineWidth} /></label>
  <button on:click={clearCanvas}>Clear</button>
  <button on:click={addNote}>Add Note</button>
</div>

<!-- Canvas -->
<canvas
  bind:this={canvas}
  on:mousedown={startDraw}
  on:mouseup={endDraw}
  on:mousemove={draw}
  on:mouseleave={endDraw}
></canvas>

<!-- Sticky Notes -->
{#each notes as note (note.id)}
  <div
    class="note"
    style="top:{note.y}px; left:{note.x}px; background:{note.color}; height:{note.height}px;"
    on:mousedown={(e) => startDrag(e, note)}
  >
    <textarea
      bind:value={note.text}
      placeholder="Write something..."
      on:input={(e) => autoResize(e, note)}
      style="height:{note.height}px;"
    ></textarea>
    <button class="delete" on:click={() => deleteNote(note.id)}>✖</button>
  </div>
{/each}

<style>
  body {
    margin: 0;
    font-family: 'Inter', sans-serif;
  }

  canvas {
    width: 100vw;
    height: 100vh;
    cursor: crosshair;
    display: block;
  }

  .toolbar {
    position: fixed;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10;
    display: flex;
    gap: 1rem;
    align-items: center;
    background: rgba(0,0,0,0.4);
    backdrop-filter: blur(8px);
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 12px;
  }

  .toolbar button {
    background: rgba(255, 255, 255, 0.15);
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    padding: 0.3rem 0.7rem;
    color: white;
    cursor: pointer;
    transition: background 0.2s, transform 0.1s;
  }

  .toolbar button:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: scale(1.05);
  }

  .note {
    position: absolute;
    padding: 0.5rem 0.75rem;
    border-radius: 12px;
    width: 200px;
    min-height: 50px;
    box-shadow: 0 6px 18px rgba(0,0,0,0.35);
    cursor: grab;
    z-index: 5;
    display: flex;
    flex-direction: column;
    transition: transform 0.1s;
    overflow: hidden;
  }

  .note:active {
    transform: scale(1.03);
  }

  .note textarea {
    width: 100%;
    border: none;
    resize: none;
    background: transparent;
    font-size: 0.95rem;
    color: #333;
    outline: none;
    overflow: hidden;
    font-weight: bold;
  }

  .delete {
    position: absolute;
    top: 4px;
    right: 6px;
    background: none;
    border: none;
    color: #333;
    cursor: pointer;
  }
</style>
