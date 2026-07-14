<script>
  import Window from './components/Window.svelte';

  let windows = [
    { id: 'terminal', title: 'Debian Minimal Terminal Core', x: 60, y: 60, type: 'webvm' },
    { id: 'editor', title: 'Monaco Code Workspace', x: 200, y: 150, type: 'editor' }
  ];

  let activeWindowId = 'terminal';

  function bringToFront(id) {
    activeWindowId = id;
  }
</script>

<main class="desktop">
  {#each windows as win (win.id)}
    <Window 
      title={win.title} 
      bind:x={win.x} 
      bind:y={win.y}
      active={activeWindowId === win.id}
      on:pointerdown={() => bringToFront(win.id)}
    >
      {#if win.type === 'webvm'}
        <iframe 
          src="https://YOUR_GITHUB_USERNAME.github.io/webvm/" 
          title="WebVM Engine"
          class="core-frame"
        ></iframe>
      {:else if win.type === 'editor'}
        <div class="editor-placeholder">
          <p>Native Monaco Workspace Initialization Layer</p>
        </div>
      {/if}
    </Window>
  {/each}
</main>

<style>
  .desktop {
    width: 100vw;
    height: 100vh;
    background: #111115;
    position: relative;
    overflow: hidden;
  }
  .core-frame {
    width: 100%;
    height: 100%;
    border: none;
  }
  .editor-placeholder {
    padding: 20px;
    color: #aaa;
  }
</style>
