<script>
  import { onMount } from 'svelte';
  import Window from './components/Window.svelte';
  import TerminalClient from './apps/TerminalClient.svelte';
  import EditorApp from './apps/EditorApp.svelte';

  let showBoot = true;
  let bootProgress = 0;
  let bootStatus = "Booting Linux Kernel...";
  let clockTime = "12:00 PM";
  let clockDate = "July 15, 2026";
  let showStartMenu = false;
  let activeWindowId = null;

  let windows = [];
  
  // Reference to our hidden background WebVM core
  let backgroundWebVMFrame;

  // App Registry (Notice how "terminal" is now a client connected to the master core!)
  const APPS_REGISTRY = {
    terminal: {
      id: 'terminal',
      title: 'Linux Shell Session',
      icon: 'terminal',
      accentColor: 'text-emerald-400',
      bgGradient: 'from-emerald-500/15 to-transparent',
      width: 700,
      height: 400,
    },
    editor: {
      id: 'editor',
      title: 'Monaco Workspace',
      icon: 'file-code',
      accentColor: 'text-indigo-400',
      bgGradient: 'from-indigo-500/10 to-transparent',
      width: 780,
      height: 520,
    }
  };

  onMount(() => {
    // Simulated system load
    const bootSteps = [
      { prg: 30, msg: "Booting Master Linux Container in background..." },
      { prg: 70, msg: "Binding virtual loopback interfaces..." },
      { prg: 100, msg: "Background Kernel Ready." }
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
          // Spawn our first clean terminal session instantly
          openApp('terminal');
        }, 400);
      }
    }, 200);

    // Clock update tick
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

    // Communication bridge between master kernel and terminal client windows
    const handleMessage = (event) => {
      // Catch terminal messages and multiplex them to the right window session
      if (event.data && event.data.type === 'WEBSHELL_DATA') {
        const targetWindow = windows.find(w => w.id === event.data.sessionId);
        if (targetWindow) {
          // Send terminal updates straight to the correct window tab
        }
      }
    };
    window.addEventListener('message', handleMessage);

    return () => {
      clearInterval(bootTimer);
      clearInterval(clockTimer);
      window.removeEventListener('message', handleMessage);
    };
  });

  function openApp(appKey) {
    const meta = APPS_REGISTRY[appKey];
    if (!meta) return;

    const winId = `${appKey}_${Date.now()}`;
    const newWindow = {
      id: winId,
      appKey,
      title: `${meta.title} (tty-${windows.filter(w => w.appKey === 'terminal').length + 1})`,
      icon: meta.icon,
      accentColor: meta.accentColor,
      bgGradient: meta.bgGradient,
      width: meta.width,
      height: meta.height,
      minimized: false,
      active: true
    };

    windows = windows.map(w => ({ ...w, active: false }));
    windows = [...windows, newWindow];
    activeWindowId = winId;
    showStartMenu = false;
  }

  function focusWindow(winId) {
    windows = windows.map(w => ({ ...w, active: w.id === winId }));
    activeWindowId = winId;
  }

  function minimizeWindow(winId) {
    windows = windows.map(w => w.id === winId ? { ...w, minimized: true, active: false } : w);
  }

  function closeWindow(winId) {
    windows = windows.filter(w => w.id !== winId);
    if (activeWindowId === winId) {
      activeWindowId = windows.length > 0 ? windows[windows.length - 1].id : null;
      if (activeWindowId) focusWindow(activeWindowId);
    }
  }

  function restoreWindow(winId) {
    windows = windows.map(w => w.id === winId ? { ...w, minimized: false, active: true } : { ...w, active: false });
    activeWindowId = winId;
  }
</script>

<div class="h-screen w-screen overflow-hidden select-none bg-slate-950 font-sans text-slate-100">
  <iframe 
    src="https://lucasinator5000.github.io/VIZONwebvm/" 
    title="Background Linux Master Kernel"
    class="hidden w-0 h-0 border-none"
    bind:this={backgroundWebVMFrame}
  ></iframe>

  {#if showBoot}
    <div class="absolute inset-0 bg-slate-950 z-[99999] flex flex-col items-center justify-center">
      <div class="flex flex-col items-center space-y-6">
        <div class="relative w-20 h-20 flex items-center justify-center bg-gradient-to-tr from-emerald-600 via-teal-600 to-blue-600 rounded-2xl shadow-2xl animate-pulse">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-10 h-10 text-white" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect width="20" height="12" x="2" y="4" rx="2" /><path d="m7 8 3 2-3 2" /><path d="M12 12h4" /></svg>
        </div>
        <div class="text-center">
          <h1 class="text-2xl font-bold tracking-widest text-slate-200">VIZON Core OS</h1>
          <p class="text-xs text-emerald-500 tracking-wider mt-1 font-mono uppercase">Initializing Multi-Session Kernel</p>
        </div>
        <div class="w-48 bg-slate-800 h-1.5 rounded-full overflow-hidden">
          <div class="bg-gradient-to-r from-emerald-500 to-blue-500 h-full transition-all duration-300" style="width: {bootProgress}%"></div>
        </div>
        <p class="text-xs text-slate-400 font-mono">{bootStatus}</p>
      </div>
    </div>
  {:else}
    <main class="w-full h-full relative flex flex-col bg-cover bg-center" style="background-image: url('https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?auto=format&fit=crop&w=1920&q=80');">
      <div class="absolute inset-0 bg-slate-950/25 backdrop-brightness-[0.93] pointer-events-none"></div>

      <div class="absolute top-6 left-6 flex flex-col gap-5 z-10">
        {#each Object.keys(APPS_REGISTRY) as key}
          <button on:click={() => openApp(key)} class="flex flex-col items-center justify-center w-20 p-2 rounded-xl hover:bg-slate-900/40 hover:backdrop-blur-md group transition-all">
            <div class="p-3 bg-slate-950/60 border border-slate-800/50 rounded-xl group-hover:scale-105 shadow-md text-slate-300 {APPS_REGISTRY[key].accentColor}">
              {#if key === 'terminal'}
                <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><rect width="20" height="12" x="2" y="4" rx="2" /><path d="m7 8 3 2-3 2" /><path d="M12 12h4" /></svg>
              {:else}
                <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a2 2 0 0 0 2 2h4"/></svg>
              {/if}
            </div>
            <span class="text-[10px] font-semibold text-slate-200 mt-1.5 text-center">{APPS_REGISTRY[key].title}</span>
          </button>
        {/each}
      </div>

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
            {#if win.appKey === 'terminal'}
              <TerminalClient sessionId={win.id} />
            {:else if win.appKey === 'editor'}
              <EditorApp />
            {/if}
          </Window>
        {/each}
      </div>

      <footer class="w-[calc(100%-2rem)] h-12 glass bg-slate-950/65 border border-slate-800/60 rounded-xl mb-3 mx-auto flex items-center justify-between px-4 z-[9999] pointer-events-auto shadow-2xl">
        <div class="flex items-center space-x-3">
          <button on:click|stopPropagation={() => showStartMenu = !showStartMenu} class="flex items-center justify-center p-2 rounded-lg bg-emerald-600 hover:bg-emerald-500 text-white shadow-md active:scale-95">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><rect width="7" height="9" x="3" y="3" rx="1"/><rect width="7" height="5" x="14" y="3" rx="1"/><rect width="7" height="9" x="14" y="12" rx="1"/><rect width="7" height="5" x="3" y="16" rx="1"/></svg>
          </button>
          <div class="h-5 w-[1px] bg-slate-800"></div>

          <div class="flex items-center space-x-1.5">
            {#each windows as win (win.id)}
              <button 
                on:click={() => win.minimized ? restoreWindow(win.id) : focusWindow(win.id)}
                class="flex items-center space-x-2 px-3 py-1 rounded-lg border text-xs font-medium transition-all {win.active ? 'bg-emerald-500/10 border-emerald-500/30 text-emerald-400' : 'bg-slate-900/40 border-slate-800/50 text-slate-400 hover:bg-slate-800/60'}"
              >
                <span class="w-1.5 h-1.5 rounded-full {win.active ? 'bg-emerald-400 animate-pulse' : 'bg-slate-500'}"></span>
                <span class="max-w-[120px] truncate">{win.title}</span>
              </button>
            {/each}
          </div>
        </div>

        <div class="flex items-center space-x-4 text-xs font-medium text-slate-300">
          <div class="hidden sm:flex items-center space-x-1.5 bg-slate-900/40 px-2.5 py-1 rounded-lg border border-slate-800/60">
            <span class="relative flex h-2 w-2">
              <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
              <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"></span>
            </span>
            <span class="text-[10px] text-emerald-400 font-mono">KERNEL DAEMON ON</span>
          </div>
          <div class="text-right flex flex-col justify-center">
            <span class="font-bold text-slate-200">{clockTime}</span>
            <span class="text-[9px] text-slate-400 mt-0.5">{clockDate}</span>
          </div>
        </div>
      </footer>
    </main>
  {/if}
</div>
