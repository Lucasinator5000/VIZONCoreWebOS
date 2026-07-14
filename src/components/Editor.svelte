<script>
  import { onMount, onDestroy } from 'svelte';

  let editorContainer;
  let editorInstance;

  onMount(() => {
    // Tell require.js where Monaco is hosted
    require.config({ paths: { vs: 'https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.43.0/min/vs' } });

    require(['vs/editor/editor.main'], () => {
      // Create the editor instance inside our Svelte container
      editorInstance = monaco.editor.create(editorContainer, {
        value: [
          '# Welcome to VIZONCoreWebOS Workspace!',
          'import sys',
          '',
          'def hello_world():',
          '    print("Hello from native Monaco workspace!")',
          '',
          'hello_world()'
        ].join('\n'),
        language: 'python',
        theme: 'vs-dark',
        automaticLayout: true, // Auto-resizes when the window scales
        fontSize: 14,
        minimap: { enabled: false }
      });
    });
  });

  onDestroy(() => {
    if (editorInstance) {
      editorInstance.dispose();
    }
  });
</script>

<div class="editor-wrapper">
  <div bind:this={editorContainer} class="monaco-container"></div>
</div>

<style>
  .editor-wrapper {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    background: #1e1e1e;
  }
  .monaco-container {
    width: 100%;
    height: 100%;
  }
</style>
