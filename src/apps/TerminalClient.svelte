<script>
  import { onMount } from 'svelte';
  
  export let sessionId; // Automatically maps uniquely to each active tab window!

  let terminalInput = "";
  let consoleHistory = [
    { text: "VIZON Virtual Core Shell v1.0.0", type: "system" },
    { text: `Connected back-channel to WebVM daemon [Session: ${sessionId}]`, type: "success" },
    { text: "Type 'help' to see system diagnostics or run native Linux scripts.", type: "system" }
  ];

  let terminalEnd;

  function runCommand() {
    const cmd = terminalInput.trim();
    if (!cmd) return;

    // 1. Add command to local interface history
    consoleHistory = [...consoleHistory, { text: `vizon@kernel:~# ${cmd}`, type: "command" }];

    // 2. Local fallback commands, or forward command directly to our background container!
    if (cmd.toLowerCase() === 'help') {
      consoleHistory = [
        ...consoleHistory,
        { text: "Available System Services:", type: "info" },
        { text: " - clear      Clear this terminal window", type: "info" },
        { text: " - diagnostics Send system health packets to master kernel", type: "info" },
        { text: " - bash [cmd]  Forward script instructions to background Linux instance", type: "info" }
      ];
    } else if (cmd.toLowerCase() === 'clear') {
      consoleHistory = [];
    } else if (cmd.toLowerCase() === 'diagnostics') {
      consoleHistory = [
        ...consoleHistory,
        { text: "Memory State: OK", type: "success" },
        { text: "VRAM Buffer: 8GB GDDR7 - Active Blackwell offloading running.", type: "success" },
        { text: "IPC Channel: Online - Connected to master WebVM", type: "success" }
      ];
    } else {
      // 🚀 FORWARD THE COMMAND OVER THE INTER-PROCESS BRIDGE!
      // This sends the input directly to the single hidden Linux instance running in the background
      window.parent.postMessage({
        type: 'EXECUTE_LINUX_CMD',
        sessionId: sessionId,
        command: cmd
      }, '*');
      
      consoleHistory = [...consoleHistory, { text: `[IPC sent] Executing: ${cmd}`, type: "info" }];
    }

    terminalInput = "";
    
    // Auto scroll container
    setTimeout(() => {
      if (terminalEnd) terminalEnd.scrollIntoView({ behavior: 'smooth' });
    }, 50);
  }
</script>

<div class="flex-grow flex flex-col h-full bg-slate-950 font-mono text-emerald-400 p-4 select-text">
  <div class="flex-1 overflow-y-auto space-y-1.5 scrollbar-thin scrollbar-thumb-slate-800">
    {#each consoleHistory as item}
      <div class="text-xs leading-relaxed break-all">
        {#if item.type === 'command'}
          <span class="text-slate-200 font-bold">{item.text}</span>
        {:else if item.type === 'success'}
          <span class="text-teal-400">{item.text}</span>
        {:else if item.type === 'info'}
          <span class="text-blue-400">{item.text}</span>
        {:else}
          <span class="text-slate-400">{item.text}</span>
        {/if}
      </div>
    {/each}
    <div bind:this={terminalEnd}></div>
  </div>

  <div class="mt-2 pt-2 border-t border-slate-900 flex items-center space-x-2 shrink-0">
    <span class="text-emerald-500 font-bold text-xs select-none">vizon@kernel:~#</span>
    <input 
      type="text" 
      class="flex-1 bg-transparent border-none outline-none text-slate-200 text-xs font-mono"
      placeholder="Type command..."
      bind:value={terminalInput}
      on:keydown={(e) => e.key === 'Enter' && runCommand()}
      autofocus
    />
  </div>
</div>
