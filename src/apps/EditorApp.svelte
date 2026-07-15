<script>
  import { onMount, onDestroy } from 'svelte';

  let editorContainer;
  let editor;

  onMount(async () => {
    // Load Monaco Editor dynamically on mount
    const monaco = await import('monaco-editor');
    
    editor = monaco.editor.create(editorContainer, {
      value: `// Welcome to VIZON Monaco Workspace\nfunction greet() {\n  console.log("Hello, Horizon!");\n}`,
      language: 'javascript',
      theme: 'vs-dark',
      automaticLayout: true,
      minimap: { enabled: false }
    });
  });

  onDestroy(() => {
    if (editor) editor.dispose();
  });
</script>

<div class="flex-grow flex flex-col h-full bg-slate-950">
  <div class="flex items-center justify-between px-4 py-2 bg-slate-900 border-b border-slate-800">
    <span class="text-xs text-slate-400 font-mono">workspace.js</span>
    <button class="px-2 py-1 bg-blue-600 hover:bg-blue-500 rounded text-xs text-white font-medium">Save</button>
  </div>
  <div bind:this={editorContainer} class="flex-1 w-full h-full"></div>
</div>
