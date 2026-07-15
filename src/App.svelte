<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import Window from './components/Window.svelte';
  import EditorApp from './apps/EditorApp.svelte';

  // Desktop configuration state
  let showBoot = true;
  let bootProgress = 0;
  let bootStatus = "Initializing system modules...";
  let clockTime = "12:00 PM";
  let clockDate = "July 14, 2026";
  let showStartMenu = false;
  let activeWindowId = null;
  let showContextMenu = false;
  let contextMenuX = 0;
  let contextMenuY = 0;

  // Virtual File System (mock)
  let fs = {
    'documents': {
      'welcome.txt': 'Welcome to VIZON OS!\n\nThis is a fully customizable Svelte-powered WebOS workspace. Enjoy built-in workspace templates!',
      'ideas.txt': 'Project Goals:\n1. Build fully responsive desktop layouts.\n2. Optimize code workspace using Svelte.\n3. Implement local state management.'
    }
  };

  // List of active windows running on the screen
  let windows = [];

  // Desktop Wallpapers
  const wallpapers = [
    'https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?auto=format&fit=crop&w=1920&q=80',
    'https://images.unsplash.com/photo-1634017839464-5c339ebe3cb4?auto=format&fit=crop&w=1920&q=80',
    'https://images.unsplash.com/photo-1635070041078-e363dbe005cb?auto=format&fit=crop&w=1920&q=80'
  ];
  let currentWallpaperIdx = 0;

  // Registered Applications metadata
  const APPS_REGISTRY = {
    editor: {
      id: 'editor',
      title: 'Monaco Workspace',
      icon: 'file-code',
      accentColor: 'text-indigo-400',
      bgGradient: 'from-indigo-500/10 to-transparent',
      width: 780,
      height: 520,
    },
    notepad: {
      id: 'notepad',
      title: 'Notepad Text',
      icon: 'file-text',
      accentColor: 'text-amber-400',
      bgGradient: 'from-amber-500/10 to-transparent',
      width: 500,
      height: 380,
    }
  };

  onMount(() => {
    // 1. Simulating System Boot loader sequence
    const bootSteps = [
      { prg: 20, msg: "Loading kernel components..." },
      { prg: 55, msg: "Initializing window manager interface..." },
      { prg: 85, msg: "Mounting local filesystem..." },
      { prg: 100, msg: "Ready." }
    ];

    let stepIdx = 0;
    const bootTimer = setInterval(() => {
      if (stepIdx < bootSteps.length) {
        bootProgress = bootSteps[stepIdx].prg;
        bootStatus = bootSteps[stepIdx].msg;
        stepIdx++;
      } else {
        clearInterval(bootTimer);
        setTimeout(() => {
          showBoot = false;
        }, 300);
      }
    }, 300);

    // 2. Setup Clock update loop
    const updateTime = () => {
      const now = new Date();
      let hours = now.getHours();
      const minutes = String(now.getMinutes()).padStart(2, '0');
      const ampm = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12 || 12;
      clockTime = `${hours}:${minutes} ${ampm}`;
      clockDate = now.toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });
    };
    updateTime();
    const clockTimer = setInterval(updateTime, 1000);

    return () => {
      clearInterval(bootTimer);
      clearInterval(clockTimer);
    };
  });

  // App Execution Core
  function openApp(appKey) {
    const meta = APPS_REGISTRY[appKey];
    if (!meta) return;

    const winId = `${appKey}_${Date.now()}`;
    const newWindow = {
      id: winId,
      appKey,
      title: meta.title,
      icon: meta.icon,
      accentColor: meta.accentColor,
      bgGradient: meta.bgGradient,
      width: meta.width,
      height: meta.height,
      minimized: false,
      active: true
    };

    // Bring previous windows out of active focus
    windows = windows.map(w => ({ ...w, active: false }));
    windows = [...windows, newWindow];
    activeWindowId = winId;
    showStartMenu = false;
  }

  function focusWindow(winId) {
    windows = windows.map(w => ({
      ...w,
      active: w.id === winId
    }));
    activeWindowId = winId;
  }

  function minimizeWindow(winId) {
    windows = windows.map(w => w.id === winId ? { ...w, minimized: true, active: false } : w);
  }

  function closeWindow(winId) {
    windows = windows.filter(w => w.id !== winId);
    if (activeWindowId === winId) {
      activeWindowId = windows.length > 0 ? windows[windows.length - 1].id : null;
      if (activeWindowId) {
        focusWindow(activeWindowId);
      }
    }
  }

  function restoreWindow(winId) {
    windows = windows.map(w => {
      if (w.id === winId) {
        return { ...w, minimized: false, active: true };
      }
      return { ...w, active: false };
    });
    activeWindowId = winId;
  }

  // Desktop right-click management
  function handleContextMenu(e) {
    e.preventDefault();
    contextMenuX = e.clientX;
    contextMenuY = e.clientY;
    showContextMenu = true;
  }

  function closeContextMenu() {
    showContextMenu = false;
  }

  function changeWallpaper() {
    currentWallpaperIdx = (currentWallpaperIdx + 1) % wallpapers.length;
    closeContextMenu();
  }

  function rebootSystem() {
    showBoot = true;
    bootProgress = 0;
    bootStatus = "Rebooting OS kernel...";
    windows = [];
    setTimeout(() => {
      window.location.reload();
    }, 1000);
  }
</script>

<div class="h-screen w-screen overflow-hidden select-none bg-slate-950 font-sans text-slate-100" on:click={closeContextMenu}>
  {#if showBoot}
    <div class="absolute inset-0 bg-slate-950 z-[99999] flex flex-col items-center justify-center transition-all duration-700">
      <div class="flex flex-col items-center space-y-6">
        <div class="relative w-20 h-20 flex items-center justify-center bg-gradient-to-tr from-blue-600 via-indigo-600 to-purple-600 rounded-2xl shadow-2xl animate-pulse">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-10 h-10 text-white" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275Z"/></svg>
        </div>
        <div class="text-center">
          <h1 class="text-2xl font-bold tracking-widest text-slate-200">VIZON OS</h1>
          <p class="text-xs text-slate-500 tracking-wider mt-1 font-mono uppercase">Initializing Boot Sequence</p>
        </div>
        <div class="w-48 bg-slate-800 h-1.5 rounded-full overflow-hidden">
          <div class="bg-gradient-to-r from-blue-500 to-indigo-500 h-full transition-all duration-300" style="width: {bootProgress}%"></div>
        </div>
        <p class="text-xs text-slate-400 font-mono">{bootStatus}</p>
      </div>
    </div>
  {:else}
    <main 
      class="w-full h-full relative flex flex-col bg-cover bg-center transition-all duration-700" 
      style="background-image: url('{wallpapers[currentWallpaperIdx]}');"
      on:contextmenu={handleContextMenu}
    >
      <div class="absolute inset-0 bg-slate-950/20 backdrop-brightness-[0.93] pointer-events-none"></div>

      <div class="absolute top-6 left-6 flex flex-col gap-5 z-10">
        {#each Object.keys(APPS_REGISTRY) as key}
          <button 
            on:click={() => openApp(key)} 
            class="flex flex-col items-center justify-center w-20 p-2 rounded-xl border border-transparent hover:border-blue-500/20 hover:bg-slate-900/40 hover:backdrop-blur-md group transition-all duration-200"
          >
            <div class="p-3 bg-slate-950/60 border border-slate-800/50 rounded-xl group-hover:scale-105 shadow-md transition-all text-slate-300">
              <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a2 2 0 0 0 2 2h4"/><path d="M10 9H8"/><path d="M16 13H8"/><path d="M16 17H8"/></svg>
            </div>
            <span class="text-[10px] font-semibold text-slate-200 mt-1.5 drop-shadow-md select-none text-center leading-snug">{APPS_REGISTRY[key].title}</span>
          </button>
        {/each}
      </div>

      {#if showContextMenu}
        <div 
          class="absolute glass bg-slate-900/90 border border-slate-700/50 rounded-xl shadow-2xl p-1 w-44 z-[9999] text-xs font-medium"
          style="left: {contextMenuX}px; top: {contextMenuY}px;"
          on:click|stopPropagation
        >
          <button on:click={changeWallpaper} class="w-full flex items-center px-3 py-2 text-slate-300 hover:bg-slate-800/80 rounded-lg text-left">
            Change Wallpaper
          </button>
          <button on:click={rebootSystem} class="w-full flex items-center px-3 py-2 text-rose-400 hover:bg-rose-950/30 rounded-lg text-left">
            Reboot OS
          </button>
        </div>
      {/if}

      <div class="flex-grow w-full h-full relative overflow-hidden pointer-events-none">
        {#each windows as win (win.id)}
          <Window 
            id={win.id}
            title={win.title}
            icon={win.icon}
            accentColor={win.accentColor}
            bgGradient={win.bgGradient}
            active={win.active}
            minimized={win.minimized}
            width={win.width}
            height={win.height}
            on:active={() => focusWindow(win.id)}
            on:minimize={() => minimizeWindow(win.id)}
            on:close={() => closeWindow(win.id)}
          >
            {#if win.appKey === 'editor'}
              <EditorApp />
            {:else if win.appKey === 'notepad'}
              <div class="p-4 flex-1 flex flex-col bg-slate-950">
                <textarea 
                  class="flex-grow w-full bg-transparent border-none outline-none text-slate-300 font-mono text-sm resize-none" 
                  placeholder="Start typing your ideas here..."
                  value={fs.documents['welcome.txt']}
                ></textarea>
              </div>
            {/if}
          </Window>
        {/each}
      </div>

      {#if showStartMenu}
        <div class="absolute bottom-16 left-4 w-80 h-[400px] glass bg-slate-950/90 border border-slate-800/80 shadow-2xl rounded-2xl z-[9000] flex flex-col overflow-hidden">
          <div class="p-4 bg-gradient-to-r from-blue-900/40 to-slate-900 border-b border-slate-800/60 flex items-center space-x-3">
            <div class="w-8 h-8 rounded-full bg-blue-500/20 border border-blue-400/30 flex items-center justify-center text-xs font-bold text-blue-400">G</div>
            <div>
              <h4 class="font-bold text-xs">Vizon Workspace</h4>
              <span class="text-[10px] text-emerald-400">System Ready</span>
            </div>
          </div>
          <div class="flex-grow p-4 grid grid-cols-2 gap-3 overflow-y-auto">
            {#each Object.keys(APPS_REGISTRY) as key}
              <button 
                on:click={() => openApp(key)}
                class="flex items-center space-x-2 p-2 rounded-xl bg-slate-900 hover:bg-slate-800 border border-slate-800/60 transition-colors"
              >
                <div class="p-2 rounded-lg bg-slate-950 {APPS_REGISTRY[key].accentColor}">
                  <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a2 2 0 0 0 2 2h4"/></svg>
                </div>
                <span class="text-xs text-slate-300 font-medium">{APPS_REGISTRY[key].title}</span>
              </button>
            {/each}
          </div>
          <div class="p-3 bg-slate-900/80 border-t border-slate-800/60 flex justify-between items-center text-xs">
            <span class="text-slate-400 select-none">v1.0.0-stable</span>
            <button on:click={rebootSystem} class="text-rose-400 hover:text-rose-300">Shut Down</button>
          </div>
        </div>
      {/if}

      <footer class="w-[calc(100%-2rem)] h-12 glass bg-slate-950/65 border border-slate-800/60 rounded-xl mb-3 mx-auto flex items-center justify-between px-4 z-[9999] pointer-events-auto shadow-2xl">
        <div class="flex items-center space-x-3">
          <button 
            on:click|stopPropagation={() => showStartMenu = !showStartMenu} 
            class="flex items-center justify-center p-2 rounded-lg bg-blue-600 hover:bg-blue-500 text-white shadow-md transition-all active:scale-95"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><rect width="7" height="9" x="3" y="3" rx="1"/><rect width="7" height="5" x="14" y="3" rx="1"/><rect width="7" height="9" x="14" y="12" rx="1"/><rect width="7" height="5" x="3" y="16" rx="1"/></svg>
          </button>
          <div class="h-5 w-[1px] bg-slate-800"></div>

          <div class="flex items-center space-x-1.5">
            {#each windows as win (win.id)}
              <button 
                on:click={() => win.minimized ? restoreWindow(win.id) : focusWindow(win.id)}
                class="flex items-center space-x-2 px-3 py-1 rounded-lg border text-xs font-medium transition-all {win.active ? 'bg-blue-500/10 border-blue-500/30 text-blue-400' : 'bg-slate-900/40 border-slate-800/50 text-slate-400 hover:bg-slate-800/60'}"
              >
                <span class="w-1.5 h-1.5 rounded-full {win.active ? 'bg-blue-400 animate-pulse' : 'bg-slate-500'}"></span>
                <span class="max-w-[80px] truncate select-none">{win.title}</span>
              </button>
            {/each}
          </div>
        </div>

        <div class="flex items-center space-x-4 text-xs font-medium text-slate-300">
          <div class="hidden sm:flex items-center space-x-1.5 bg-slate-900/40 px-2.5 py-1 rounded-lg border border-slate-800/60">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5 text-emerald-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/></svg>
            <span class="text-[10px]">Cloud Active</span>
          </div>
          <div class="text-right flex flex-col justify-center">
            <span class="font-bold text-slate-200">{clockTime}</span>
            <span class="text-[9px] text-slate-400 leading-none mt-0.5">{clockDate}</span>
          </div>
        </div>
      </footer>
    </main>
  {/if}
</div>
