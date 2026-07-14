<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let title = "Window";
  export let id;
  export let x = 100;
  export let y = 100;
  export let active = false;
  export let minimized = false;

  let isDragging = false;
  let isMaximised = false;
  
  // Store pre-maximised positions to restore them later
  let prevX = x;
  let prevY = y;

  function startDrag(event) {
    if (isMaximised) return;
    isDragging = true;
    dispatch('active'); // Bring to front on drag start
    window.addEventListener('pointermove', drag);
    window.addEventListener('pointerup', stopDrag);
  }

  function drag(event) {
    if (!isDragging) return;
    x += event.movementX;
    y += event.movementY;
  }

  function stopDrag() {
    isDragging = false;
    window.removeEventListener('pointermove', drag);
    window.removeEventListener('pointerup', stopDrag);
  }

  function toggleMaximise() {
    if (isMaximised) {
      x = prevX;
      y = prevY;
    } else {
      prevX = x;
      prevY = y;
      x = 0;
      y = 0;
    }
    isMaximised = !isMaximised;
  }

  function closeWindow() {
    dispatch('close', { id });
  }

  function minimizeWindow() {
    dispatch('minimize', { id });
  }
</script>

<div 
  class="window" 
  class:active 
  class:maximised={isMaximised}
  style="
    transform: translate3d({x}px, {y}px, 0); 
    display: {minimized ? 'none' : 'flex'};
  "
  on:pointerdown={() => dispatch('active')}
>
  <div class="window-header" on:pointerdown={startDrag}>
    <span class="title">{title}</span>
    <div class="window-controls">
      <button class="control-btn min" on:click|stopPropagation={minimizeWindow}>–</button>
      <button class="control-btn max" on:click|stopPropagation={toggleMaximise}>square;</button>
      <button class="control-btn close" on:click|stopPropagation={closeWindow}>×</button>
    </div>
  </div>
  
  <div class="window-content">
    {#if isDragging}
      <div class="drag-overlay"></div>
    {/if}
    <slot />
  </div>
</div>

<style>
  .window {
    position: absolute;
    width: 650px;
    height: 450px;
    background: #2d2d30;
    border-radius: 6px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.6);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    border: 1px solid #444;
    z-index: 1;
    transition: border-color 0.1s;
  }
  .window.active {
    z-index: 100;
    border-color: #007acc;
    box-shadow: 0 10px 35px rgba(0, 122, 204, 0.3);
  }
  .window.maximised {
    width: 100vw !important;
    height: calc(100vh - 48px) !important; /* Subtract space for Taskbar */
    border-radius: 0;
    border: none;
  }
  .window-header {
    background: #252526;
    color: #ccc;
    padding: 8px 12px;
    cursor: move;
    user-select: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #1e1e1e;
  }
  .window-content {
    flex: 1;
    position: relative;
    background: #1e1e1e;
  }
  .drag-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 9999;
    background: transparent;
  }
  .window-controls {
    display: flex;
    gap: 6px;
  }
  .control-btn {
    background: #3c3c3c;
    border: none;
    color: #fff;
    width: 22px;
    height: 22px;
    border-radius: 4px;
    font-size: 12px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .control-btn:hover {
    background: #505050;
  }
  .control-btn.close:hover {
    background: #e81123;
  }
</style>
