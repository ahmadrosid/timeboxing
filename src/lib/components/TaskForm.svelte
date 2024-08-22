<script lang="ts">
  import Plus from "lucide-svelte/icons/plus";
  export let newTask;
  export let addTask;
</script>

<div class="mb-6 space-y-2">
  <h2 class="text-xl font-bold">Add New Task</h2>
  <div class="grid gap-2 mb-2 w-full">
    <input
      placeholder="Task name"
      bind:value={newTask.name}
      class="border border-gray-500 bg-white h-10 px-2 rounded-lg text-sm focus:outline-black"
    />
    <input
      placeholder="Duration (min)"
      type="number"
      bind:value={newTask.duration}
      class="border border-gray-500 bg-white h-10 px-2 rounded-lg text-sm focus:outline-black"
    />
  </div>
  <div class="w-full">
    <button
      on:click={addTask}
      class="w-full bg-black hover:bg-gray-700 text-white font-bold py-2 px-4 rounded flex items-center gap-2 justify-center"
    >
      <Plus class="h-4 w-4 mr-2" /> Add Task
    </button>
  </div>
</div>
