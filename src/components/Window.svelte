<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let title = "Window";
  export let id;
  export let x = 100;
  export let y = 100;
  export let width = 650;  // Added width control
  export let height = 450; // Added height control
  export let active = false;
  export let minimized = false;
  export let icon = "";    // Custom window icon

  let isDragging = false;
  let isResizing = false;
  let resizeType = ''; // 'r' (right), 'b' (bottom), 'br' (bottom-right)
  let isMaximised = false;
  
  let prevX = x;
  let prevY = y;
  let prevWidth = width;
  let prevHeight = height;

  // Window Dragging Logic
  function startDrag(event) {
    if (isMaximised) return;
    isDragging = true;
    dispatch('active');
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

  // Window Resizing Logic
  function startResize(event, type) {
    if (isMaximised) return;
    event.preventDefault();
    event.stopPropagation();
    isResizing = true;
    resizeType = type;
    dispatch('active');
    window.addEventListener('pointermove', resize);
    window.addEventListener('pointerup', stopResize);
  }

  function resize(event) {
    if (!isResizing) return;
    
    const minWidth = 300;
    const minHeight = 200;

    if (resizeType === 'r' || resizeType === 'br') {
      width = Math.max(minWidth, width + event.movementX);
    }
    if (resizeType === 'b' || resizeType === 'br') {
      height = Math.max(minHeight, height + event.movementY);
    }
  }

  function stopResize() {
    isResizing = false;
    window.removeEventListener('pointermove', resize);
    window.removeEventListener('pointerup', stopResize);
  }

  function toggleMaximise() {
    if (isMaximised) {
      x = prevX;
      y = prevY;
      width = prevWidth;
      height = prevHeight;
    } else {
      prevX = x;
      prevY = y;
      prevWidth = width;
      prevHeight = height;
      x = 0;
      y = 0;
      width = window.innerWidth;
      height = window.innerHeight - 48; // Account for taskbar
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
    width: {isMaximised ? '100vw' : width + 'px'};
    height: {isMaximised ? 'calc(100vh - 48px)' : height + 'px'};
    display: {minimized ? 'none' : 'flex'};
  "
  on:pointerdown={() => dispatch('active')}
>
  <div class="window-header" on:pointerdown={startDrag}>
    <div class="window-title-group">
      {#if icon}
        <img src={icon} class="window-icon" alt="" />
      {/if}
      <span class="title">{title}</span>
    </div>
    <div class="window-controls">
      <button class="control-btn min" on:click|stopPropagation={minimizeWindow}>–</button>
      <button class="control-btn max" on:click|stopPropagation={toggleMaximise}>&#9633;</button>
      <button class="control-btn close" on:click|stopPropagation={closeWindow}>×</button>
    </div>
  </div>
  
  <div class="window-content">
    {#if isDragging || isResizing}
      <div class="drag-overlay"></div>
    {/if}
    <slot />
  </div>

  {#if !isMaximised}
    <div class="resize-handle r" on:pointerdown={(e) => startResize(e, 'r')}></div>
    <div class="resize-handle b" on:pointerdown={(e) => startResize(e, 'b')}></div>
    <div class="resize-handle br" on:pointerdown={(e) => startResize(e, 'br')}></div>
  {/if}
</div>

<style>
  .window {
    position: absolute;
    background: #2d2d30;
    border-radius: 6px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.6);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    border: 1px solid #444;
    z-index: 1;
  }
  .window.active {
    z-index: 100;
    border-color: #007acc;
    box-shadow: 0 10px 35px rgba(0, 122, 204, 0.3);
  }
  .window.maximised {
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
  .window-title-group {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .window-icon {
    width: 16px;
    height: 16px;
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

  /* Resize Grab Zones */
  .resize-handle {
    position: absolute;
    background: transparent;
    z-index: 1000;
  }
  .resize-handle.r {
    top: 0;
    right: 0;
    width: 6px;
    height: 100%;
    cursor: e-resize;
  }
  .resize-handle.b {
    bottom: 0;
    left: 0;
    width: 100%;
    height: 6px;
    cursor: s-resize;
  }
  .resize-handle.br {
    bottom: 0;
    right: 0;
    width: 12px;
    height: 12px;
    cursor: se-resize;
  }
</style>
