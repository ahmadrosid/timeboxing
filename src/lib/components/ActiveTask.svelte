<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import Play from "lucide-svelte/icons/play";
  import Pause from "lucide-svelte/icons/pause";
  
  export let activeTask;
  export let timeLeft: number;
  export let isRunning: boolean;
  export let toggleTimer;

  let lastUpdateTime: number;
  let animationFrameId: number;

  $: minutes = Math.floor(timeLeft / 60);
  $: remainingSeconds = timeLeft % 60;
  $: progress = (1 - timeLeft / (activeTask.duration * 60)) * 100;

  function updateTimer() {
    if (isRunning) {
      const now = Date.now();
      const delta = (now - lastUpdateTime) / 1000;
      timeLeft = Math.max(0, timeLeft - delta);
      lastUpdateTime = now;
    }
    
    if (timeLeft > 0 && isRunning) {
      animationFrameId = requestAnimationFrame(updateTimer);
    }
  }

  function handleVisibilityChange() {
    if (!document.hidden && isRunning) {
      lastUpdateTime = Date.now();
      updateTimer();
    }
  }

  onMount(() => {
    lastUpdateTime = Date.now();
    document.addEventListener('visibilitychange', handleVisibilityChange);
    updateTimer();
  });

  onDestroy(() => {
    document.removeEventListener('visibilitychange', handleVisibilityChange);
    cancelAnimationFrame(animationFrameId);
  });
</script>

<div class="mb-6 p-4 rounded-lg shadow-lg bg-white border border-gray-500">
  <h2 class="text-2xl font-semibold mb-4">{activeTask.name}</h2>
  <div class="text-4xl font-bold mb-4">
    {String(Math.floor(minutes)).padStart(2, "0")}:{String(Math.floor(remainingSeconds)).padStart(2, "0")}
  </div>
  <div class="mb-4">
    <div class="w-full h-2 bg-gray-200 rounded-full">
      <div
        class="h-2 bg-black rounded-full"
        style="width: {progress}%;"
      ></div>
    </div>
  </div>
  <div>
    <button
      on:click={toggleTimer}
      class="w-full py-2 px-4 mt-4 rounded-lg font-semibold text-white bg-black hover:bg-gray-700 transition duration-300 flex items-center justify-center"
    >
      {#if isRunning}
        <Pause class="h-4 w-4 mr-2" /> Pause
      {:else}
        <Play class="h-4 w-4 mr-2" /> Resume
      {/if}
    </button>
  </div>
</div>
