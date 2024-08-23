<script lang="ts">
    import Play from "lucide-svelte/icons/play";
  import Trash2 from "lucide-svelte/icons/trash-2";
  import Check from "lucide-svelte/icons/check"; // Import the checkmark icon
  export let task;
  export let startTask;
  export let deleteTask;
  export let markTaskAsDone;
  export let timeLeft;
  export let activeTask;

  let progress = 0;
  if (task.status === 'in-progress' && activeTask && activeTask.id === task.id) {
    progress = (1 - timeLeft / (task.duration * 60)) * 100;
  } else if (task.status === 'done') {
    progress = 100;
  }

  const formattedDate = new Date(task.date).toLocaleDateString(undefined, {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
</script>

<div class="mb-4 p-4 rounded-lg shadow-lg bg-white border border-gray-500">
    <div class="flex items-center justify-between">
        <div>
            <h3 class="font-semibold">{task.name}</h3>
            <p class="text-sm text-gray-500">{formattedDate} - {task.duration} minutes</p>
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

    <div class="w-full h-2 bg-gray-200 rounded-full mt-2">
        <div class="h-2 bg-black rounded-full" style="width: {progress}%;"></div>
    </div>
</div>