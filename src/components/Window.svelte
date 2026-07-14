<script>
  export let title = "Window";
  export let x = 100;
  export let y = 100;
  export let active = false;

  let isDragging = false;

  function startDrag(event) {
    isDragging = true;
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
</script>

<div 
  class="window" 
  class:active 
  style="transform: translate3d({x}px, {y}px, 0);"
  on:pointerdown
>
  <div class="window-header" on:pointerdown={startDrag}>
    <span class="title">{title}</span>
    <button class="close-btn">×</button>
  </div>
  <div class="window-content">
    <slot />
  </div>
</div>

<style>
  .window {
    position: absolute;
    width: 600px;
    height: 400px;
    background: #2d2d30;
    border-radius: 6px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.5);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    border: 1px solid #444;
    z-index: 1;
  }
  .window.active {
    z-index: 100;
    border-color: #007acc;
  }
  .window-header {
    background: #3c3c3c;
    color: #fff;
    padding: 8px 12px;
    cursor: move;
    user-select: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .window-content {
    flex: 1;
    position: relative;
    background: #1e1e1e;
  }
  .close-btn {
    background: none;
    border: none;
    color: #aaa;
    font-size: 18px;
    cursor: pointer;
  }
</style>
