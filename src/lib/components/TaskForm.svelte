<script lang="ts">
  import Dropdown from './Dropdown.svelte'; 
  interface Task {
    name: string;
    duration: number | null;
    id: number;
    description: string;
    status: 'todo' | 'in-progress' | 'paused' | 'done';
    date: string;
  }

  export let newTask: Task;
  export let addTask: () => void;
  export let exportTasks: () => void;
  export let importTasks: (event: Event) => void;

  let options: string[] = ['Export', 'Import'];
  let selectedOption = '';

  function handleSelect(event: CustomEvent<string>): void {
    const option = event.detail;
    if (option === 'Export') {
      exportTasks();
    } else if (option === 'Import') {
      const fileInput = document.getElementById('fileInput') as HTMLInputElement | null;
      if (fileInput) {
        fileInput.click();
      }
    }
  }
</script>

<div class="mb-6 space-y-2">
  <div class="flex justify-between items-center mb-2">
    <h2 class="text-xl font-bold">Add New Task</h2>
    <div>
      <Dropdown {options} bind:selectedOption on:select={handleSelect} />
      <input 
        id="fileInput"
        type="file" 
        on:change={importTasks} 
        accept=".json" 
        class="hidden"
      />
    </div>
  </div>
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