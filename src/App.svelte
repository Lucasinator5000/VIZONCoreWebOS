<script>
  import Window from './components/Window.svelte';
  import Editor from './components/Editor.svelte';

  // Windows list
  let windows = [
    { id: 'terminal', title: 'VIZON Linux Terminal', x: 60, y: 60, type: 'webvm', minimized: false, closed: false },
    { id: 'editor', title: 'VIZON Code Workspace', x: 250, y: 150, type: 'editor', minimized: false, closed: false }
  ];

  let activeWindowId = 'terminal';

  function bringToFront(id) {
    activeWindowId = id;
    // Auto-unminimize when focused
    const win = windows.find(w => w.id === id);
    if (win) win.minimized = false;
    windows = [...windows];
  }

  function handleClose(event) {
    const id = event.detail.id;
    windows = windows.map(w => w.id === id ? { ...w, closed: true } : w);
  }

  function handleMinimize(event) {
    const id = event.detail.id;
    windows = windows.map(w => w.id === id ? { ...w, minimized: true } : w);
  }

  function openApp(id) {
    windows = windows.map(w => {
      if (w.id === id) {
        return { ...w, closed: false, minimized: false };
      }
      return w;
    });
    bringToFront(id);
  }
</script>

<main class="desktop">
  <div class="window-layer">
    {#each windows.filter(w => !w.closed) as win (win.id)}
      <Window 
        id={win.id}
        title={win.title} 
        bind:x={win.x} 
        bind:y={win.y}
        minimized={win.minimized}
        active={activeWindowId === win.id}
        on:active={() => bringToFront(win.id)}
        on:close={handleClose}
        on:minimize={handleMinimize}
      >
        {#if win.type === 'webvm'}
          <iframe 
            src="https://lucasinator5000.github.io/VIZONwebvm/" 
            title="WebVM Engine"
            class="core-frame"
          ></iframe>
        {:else if win.type === 'editor'}
          <Editor />
        {/if}
      </Window>
    {/each}
  </div>

  <div class="taskbar">
    <div class="start-btn">VIZON</div>
    <div class="taskbar-apps">
      {#each windows as win}
        <button 
          class="taskbar-item" 
          class:active={activeWindowId === win.id && !win.minimized && !win.closed}
          class:closed={win.closed}
          on:click={() => openApp(win.id)}
        >
          {win.title.split(' ')[1] || win.title}
        </button>
      {/each}
    </div>
    <div class="system-time">
      {new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}
    </div>
  </div>
</main>

<style>
  .desktop {
    width: 100vw;
    height: 100vh;
    background: radial-gradient(circle, #1a1b26 0%, #101014 100%);
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }
  .window-layer {
    flex: 1;
    position: relative;
    width: 100%;
    height: 100%;
  }
  .core-frame {
    width: 100%;
    height: 100%;
    border: none;
    background: #000;
  }
  
  /* Taskbar CSS Design */
  .taskbar {
    height: 48px;
    background: rgba(30, 30, 30, 0.85);
    backdrop-filter: blur(12px);
    border-top: 1px solid rgba(255, 255, 255, 0.1);
    display: flex;
    align-items: center;
    padding: 0 10px;
    gap: 12px;
    z-index: 99999; /* Ensure taskbar sits above all windows */
  }
  .start-btn {
    background: #007acc;
    color: #fff;
    font-weight: bold;
    padding: 6px 14px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
  }
  .start-btn:hover {
    background: #0098ff;
  }
  .taskbar-apps {
    flex: 1;
    display: flex;
    gap: 8px;
  }
  .taskbar-item {
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.1);
    color: #ccc;
    padding: 6px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 13px;
    transition: all 0.2s;
  }
  .taskbar-item:hover {
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
  }
  .taskbar-item.active {
    background: #007acc;
    color: #fff;
    border-color: #007acc;
  }
  .taskbar-item.closed {
    opacity: 0.4;
    border-style: dashed;
  }
  .system-time {
    color: #888;
    font-size: 13px;
    padding-right: 10px;
  }
</style>
