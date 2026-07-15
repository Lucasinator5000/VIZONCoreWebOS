<script>
  import { onMount } from 'svelte';
  import Window from './components/Window.svelte';
  import WebVMApp from './apps/WebVMApp.svelte';
  import EditorApp from './apps/EditorApp.svelte';

  // State
  let showBoot = true;
  let bootProgress = 0;
  let activeWindowId = null;
  let startMenuOpen = false;
  let windows = []; // Dynamic process tree

  // System Apps Setup
  const APPS_REGISTRY = {
    terminal: { title: 'Terminal CLI', icon: 'terminal', accentColor: 'text-emerald-400', bgGradient: 'from-emerald-500/10 to-transparent', width: 680, height: 420 },
    editor: { title: 'Notepad', icon: 'file-text', accentColor: 'text-amber-400', bgGradient: 'from-amber-500/10 to-transparent', width: 600, height: 450 }
  };

  onMount(() => {
    // Simulated boot progression sequence
    const interval = setInterval(() => {
      if (bootProgress < 100) {
        bootProgress += 20;
      } else {
        clearInterval(interval);
        setTimeout(() => showBoot = false, 500);
      }
    }, 300);
  });

  function openApp(appId) {
    const meta = APPS_REGISTRY[appId];
    const winId = `${appId}_${Date.now()}`;
    
    windows = [...windows, {
      id: winId,
      appId,
      ...meta,
      minimized: false
    }];
    activeWindowId = winId;
    startMenuOpen = false;
  }

  function closeApp(winId) {
    windows = windows.filter(w => w.id !== winId);
  }
</script>

{#if showBoot}
  {:else}
  <main class="w-full h-full relative flex flex-col bg-cover bg-center transition-all duration-700" style="background-image: url('https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?auto=format&fit=crop&w=1920&q=80');">
    <div class="absolute top-4 left-4 flex flex-col flex-wrap content-start items-start gap-4 h-[calc(100%-80px)] pointer-events-auto z-10">
      {#each Object.entries(APPS_REGISTRY) as [id, app]}
        <button on:click={() => openApp(id)} class="flex flex-col items-center justify-center w-24 p-2.5 rounded-xl hover:bg-slate-900/40 hover:backdrop-blur-sm group transition-all duration-200">
          <div class="p-3 bg-slate-950/70 border border-slate-800/80 rounded-2xl group-hover:scale-110 shadow-lg transition-all text-slate-300 {app.accentColor}">
            </div>
          <span class="text-xs font-semibold text-slate-200 text-center tracking-wide mt-2 drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)] truncate w-full">{app.title}</span>
        </button>
      {/each}
    </div>

    <div class="flex-grow w-full h-full relative overflow-hidden">
      {#each windows as win (win.id)}
        <Window 
          {...win}
          active={activeWindowId === win.id}
          on:active={() => activeWindowId = win.id}
          on:close={() => closeApp(win.id)}
          on:minimize={() => win.minimized = true}
        >
          {#if win.appId === 'terminal'}
            <WebVMApp />
          {:else if win.appId === 'editor'}
            <EditorApp />
          {/if}
        </Window>
      {/each}
    </div>

    </main>
{/if}
