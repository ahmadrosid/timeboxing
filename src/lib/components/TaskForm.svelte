<script lang="ts">
  export let newTask;
  export let addTask;
</script>

<div class="mb-6 space-y-2">
  <h2 class="text-xl font-bold mb-4">Add New Task</h2>
  <div class="grid p-2 w-full border border-gray-500 bg-white rounded-lg">
    <input
      placeholder="Task name"
      bind:value={newTask.name}
      class="h-10 px-2 text-sm focus:outline-none"
    />
    <input
      placeholder="Duration (min)"
      type="number"
      bind:value={newTask.duration}
      class="h-10 px-2 rounded-lg text-sm focus:outline-none"
    />
    <div class="flex justify-end">
      <button
        on:click={addTask}
        class="text-xs bg-black hover:bg-gray-700 text-white font-bold py-2 px-3 rounded-lg flex items-center gap-2 justify-center"
        >Add Task
      </button>
    </div>
  </div>
</div>
