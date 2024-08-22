<script lang="ts">
  import Play from "lucide-svelte/icons/play";
  import Trash2 from "lucide-svelte/icons/trash-2";
  import Check from "lucide-svelte/icons/check"; // Import the checkmark icon
  export let task;
  export let startTask;
  export let deleteTask;
  export let markTaskAsDone;
</script>

<div class="mb-4 p-4 rounded-lg shadow-lg bg-white border border-gray-300">
  <div class="flex items-center justify-between">
    <div>
      <h3 class="font-semibold">{task.name}</h3>
      <p class="text-sm text-gray-500">{task.duration} minutes</p>
      <p class="text-sm text-gray-500">Status: {task.status}</p>
    </div>
    <div class="flex gap-1">
      {#if task.status !== 'done'}
        <button on:click={() => startTask(task)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-blue-500">
          <Play class="h-4 w-4" />
        </button>
      {/if}
      <button on:click={() => deleteTask(task.id)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-red-500">
        <Trash2 class="h-4 w-4" />
      </button>
      {#if task.status === 'in-progress'}
        <button on:click={() => markTaskAsDone(task)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-green-500">
          <Check class="h-4 w-4" /> <!-- Use the checkmark icon here -->
        </button>
      {/if}
    </div>
  </div>
</div>
