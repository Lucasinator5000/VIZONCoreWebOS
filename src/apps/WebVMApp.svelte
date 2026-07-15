<script>
  // WebVM instance link 
  export let url = "https://lucasinator5000.github.io/VIZONwebvm/";
</script>

<div class="w-full h-full bg-black relative flex flex-col">
  <div class="absolute inset-0 pointer-events-none opacity-[0.06] bg-[linear-gradient(rgba(18,16,16,0)_50%,_rgba(0,0,0,0.25)_50%)] bg-[size:100%_4px] z-50"></div>
  <iframe 
    src={url} 
    title="WebVM Console"
    class="w-full h-full border-none"
    allow="clipboard-read; clipboard-write; fullscreen"
  ></iframe>
</div>
