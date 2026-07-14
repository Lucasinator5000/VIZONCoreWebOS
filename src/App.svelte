<script>
  import Window from './components/Window.svelte';
  import Editor from './components/Editor.svelte';

  // Windows configuration with dimensions and custom SVG icons
  let windows = [
    { 
      id: 'terminal', 
      title: 'Terminal Core', 
      x: 60, 
      y: 60, 
      width: 700, 
      height: 450, 
      type: 'webvm', 
      minimized: false, 
      closed: false,
      icon: 'https://lucasinator5000.github.io/VIZONwebvm/favicon.ico' // WebVM favicon
    },
    { 
      id: 'editor', 
      title: 'Code Workspace', 
      x: 250, 
      y: 150, 
      width: 650, 
      height: 400, 
      type: 'editor', 
      minimized: false, 
      closed: false,
      icon: 'https://cdn-icons-png.flaticon.com/512/1005/1005141.png' // Visual Code Icon style
    }
  ];

  let activeWindowId = 'terminal';

  function bringToFront(id) {
    activeWindowId = id;
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
  
  <div class="desktop-icons">
    {#each windows as app}
      <button class="desktop-shortcut" on:dblclick={() => openApp(app.id)}>
        <img src={app.icon} alt="" class="shortcut-icon" />
        <span class="shortcut-label">{app.title}</span>
      </button>
    {/each}
  </div>

  <div class="window-layer">
    {#each windows.filter(w => !w.closed) as win (win.id)}
      <Window 
        id={win.id}
        title={win.title} 
        bind:x={win.x} 
        bind:y={win.y}
        bind:width={win.width}
        bind:height={win.height}
        minimized={win.minimized}
        icon={win.icon}
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
          title={win.title}
        >
          <img src={win.icon} alt="" class="taskbar-icon" />
          <span class="taskbar-text">{win.title}</span>
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
    user-select: none;
  }
  
  /* Desktop Icon Shortcuts grid */
  .desktop-icons {
    position: absolute;
    top: 20px;
    left: 20px;
    display: flex;
    flex-direction: column;
    gap: 20px;
    z-index: 5;
  }
  .desktop-shortcut {
    background: transparent;
    border: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 80px;
    cursor: pointer;
    color: #fff;
    gap: 8px;
    padding: 8px;
    border-radius: 6px;
  }
  .desktop-shortcut:hover {
    background: rgba(255, 255, 255, 0.1);
  }
  .shortcut-icon {
    width: 36px;
    height: 36px;
  }
  .shortcut-label {
    font-size: 11px;
    text-shadow: 0 2px 4px rgba(0,0,0,0.8);
    text-align: center;
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
    background: rgba(15, 15, 20, 0.85);
    backdrop-filter: blur(12px);
    border-top: 1px solid rgba(255, 255, 255, 0.08);
    display: flex;
    align-items: center;
    padding: 0 10px;
    gap: 12px;
    z-index: 99999;
  }
  .start-btn {
    background: #007acc;
    color: #fff;
    font-weight: bold;
    padding: 6px 14px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 13px;
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
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.05);
    color: #aaa;
    padding: 4px 12px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 12px;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: all 0.2s;
  }
  .taskbar-item:hover {
    background: rgba(255, 255, 255, 0.08);
    color: #fff;
  }
  .taskbar-item.active {
    background: rgba(0, 122, 204, 0.2);
    color: #fff;
    border-color: #007acc;
  }
  .taskbar-item.closed {
    opacity: 0.3;
  }
  .taskbar-icon {
    width: 16px;
    height: 16px;
  }
  .system-time {
    color: #666;
    font-size: 12px;
    padding-right: 10px;
  }
</style>
