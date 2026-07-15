<script>
  import { createEventDispatcher, onMount } from 'svelte';
  const dispatch = createEventDispatcher();

  export let id; // Kept and used inside active state dispatching
  export let title;
  export let icon;
  export let accentColor;
  export let bgGradient;
  export let active = false;
  export let minimized = false;
  export let width = 600;
  export let height = 400;
  
  let x = 60 + (Math.random() * 100);
  let y = 60 + (Math.random() * 100);
  let isMaximized = false;
  let prevStyle = null;

  let isDragging = false;
  let isResizing = false;
  let startX, startY, startWidth, startHeight, startLeft, startTop;

  function handleHeaderMouseDown(e) {
    if (e.target.closest('button')) return;
    isDragging = true;
    dispatch('active', { id });
    startX = e.clientX;
    startY = e.clientY;
    startLeft = x;
    startTop = y;
    e.preventDefault();
  }

  function handleResizeMouseDown(e) {
    isResizing = true;
    dispatch('active', { id });
    startX = e.clientX;
    startY = e.clientY;
    startWidth = width;
    startHeight = height;
    e.preventDefault();
  }

  onMount(() => {
    const handleMouseMove = (e) => {
      if (isDragging) {
        x = Math.max(0, startLeft + (e.clientX - startX));
        y = Math.max(0, startTop + (e.clientY - startY));
      }
      if (isResizing) {
        width = Math.max(280, startWidth + (e.clientX - startX));
        height = Math.max(200, startHeight + (e.clientY - startY));
      }
    };

    const handleMouseUp = () => {
      isDragging = false;
      isResizing = false;
    };

    window.addEventListener('mousemove', handleMouseMove);
    window.addEventListener('mouseup', handleMouseUp);

    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
      window.removeEventListener('mouseup', handleMouseUp);
    };
  });

  function toggleMaximize() {
    if (isMaximized) {
      x = prevStyle.x;
      y = prevStyle.y;
      width = prevStyle.width;
      height = prevStyle.height;
      isMaximized = false;
    } else {
      prevStyle = { x, y, width, height };
      x = 0;
      y = 0;
      width = window.innerWidth;
      height = window.innerHeight - 60;
      isMaximized = true;
    }
  }
</script>

<div 
  class="absolute glass bg-slate-900/90 border border-slate-700/50 flex flex-col overflow-hidden pointer-events-auto select-none transition-all duration-150 shadow-2xl"
  class:rounded-2xl={!isMaximized}
  class:rounded-none={isMaximized}
  class:hidden={minimized}
  class:border-slate-600={active}
  style="width: {width}px; height: {height}px; left: {x}px; top: {y}px; z-index: {active ? 1000 : 100};"
  on:mousedown={() => dispatch('active', { id })}
>
  <div 
    class="h-11 px-4 flex items-center justify-between cursor-move bg-gradient-to-r {bgGradient} border-b border-slate-800/60 shrink-0"
    on:mousedown={handleHeaderMouseDown}
  >
    <div class="flex items-center space-x-2.5 pointer-events-none">
      <span class={accentColor}>
        <i data-lucide={icon} class="w-4 h-4"></i>
      </span>
      <span class="text-xs font-semibold tracking-wide text-slate-200 select-none">{title}</span>
    </div>

    <div class="flex items-center space-x-1.5">
      <button on:click={() => dispatch('minimize')} class="w-5 h-5 rounded-full bg-slate-800/80 hover:bg-amber-500/20 text-slate-400 hover:text-amber-400 flex items-center justify-center transition-colors">
        –
      </button>
      <button on:click={toggleMaximize} class="w-5 h-5 rounded-full bg-slate-800/80 hover:bg-emerald-500/20 text-slate-400 hover:text-emerald-400 flex items-center justify-center transition-colors">
        &#9633;
      </button>
      <button on:click={() => dispatch('close')} class="w-5 h-5 rounded-full bg-slate-800/80 hover:bg-rose-500/20 text-slate-400 hover:text-rose-400 flex items-center justify-center transition-colors">
        ×
      </button>
    </div>
  </div>

  <div class="flex-1 overflow-auto relative bg-slate-950/40 text-sm select-text flex flex-col">
    <slot />
  </div>

  {#if !isMaximized}
    <div 
      class="absolute bottom-0 right-0 w-4 h-4 cursor-se-resize z-[1000] flex items-end justify-end p-0.5 pointer-events-auto"
      on:mousedown={handleResizeMouseDown}
    >
      <svg class="text-slate-600/50" width="8" height="8" viewBox="0 0 8 8" fill="currentColor">
        <path d="M6 0h2v2H6zm0 4h2v2H6zm-4 2h2v2H2zm4 0h2v2H6z"/>
      </svg>
    </div>
  {/if}
</div>
