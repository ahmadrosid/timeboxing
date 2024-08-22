<script lang="ts">
  import { onMount } from "svelte";
  import Plus from "lucide-svelte/icons/plus";
  import Play from "lucide-svelte/icons/play";
  import Pause from "lucide-svelte/icons/pause";
  import RotateCcw from "lucide-svelte/icons/rotate-ccw";

  interface Task {
    name: string;
    duration: number;
    id: number;
    description: string;
  }

  let tasks: Task[] = [];
  let newTask: Task = { name: "", duration: 0, id: 0, description: "" }; 
  let activeTask: Task | null = null;
  let timeLeft = 0;
  let isRunning = false;

  $: minutes = Math.floor(timeLeft / 60);
  $: remainingSeconds = timeLeft % 60;

  function saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(tasks));
  }

  function loadTasks() {
    const storedTasks = localStorage.getItem('tasks');
    if (storedTasks) {
      tasks = JSON.parse(storedTasks);
    }
  }

  function addTask() {
    if (newTask.name && newTask.duration) {
      tasks = [...tasks, { ...newTask, id: Date.now() }];
      newTask = { name: "", duration: 0, id: 0, description: "" };
      saveTasks();
    }
  }

  function startTask(task: Task) {
    activeTask = task;
    timeLeft = task.duration * 60;
    isRunning = true;
  }

  function resetTask(task: Task) {
    if (activeTask && activeTask.id === task.id) {
      activeTask = null;
      timeLeft = 0;
      isRunning = false;
    }
  }

  function toggleTimer() {
    isRunning = !isRunning;
  }

  onMount(() => {
    loadTasks();
    let interval: ReturnType<typeof setInterval>;
    return () => {
      if (interval) clearInterval(interval);
    };
  });

  $: if (isRunning && timeLeft > 0) {
    setTimeout(() => {
      timeLeft -= 1;
      if (timeLeft === 0) {
        isRunning = false;
        activeTask = null;
      }
    }, 1000);
  }
</script>

<div class="max-w-4xl mx-auto p-4 flex gap-4 flex-row justify-between">
  <div class="flex-1">
    <div class="mb-6 space-y-2">
      <h2 class="text-xl font-bold">Add New Task</h2>
      <div class="grid gap-2 mb-2 w-full">
        <input
          placeholder="Task name"
          bind:value={newTask.name}
          class="border border-gray-300 bg-white h-10 px-2 rounded-lg text-sm focus:outline-black"
        />
        <input
          placeholder="Duration (min)"
          type="number"
          bind:value={newTask.duration}
          class="border border-gray-300 bg-white h-10 px-2 rounded-lg text-sm focus:outline-black"
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
    {#if activeTask}
      <div class="mb-6 p-4 rounded-lg shadow-lg bg-white border border-gray-300">
        <h2 class="text-2xl font-semibold mb-4">{activeTask.name}</h2>
        <div class="text-4xl font-bold mb-4">
          {String(minutes).padStart(2, "0")}:{String(remainingSeconds).padStart(
            2,
            "0"
          )}
        </div>
        <div class="mb-4">
          <div class="w-full h-2 bg-gray-200 rounded-full">
            <div
              class="h-2 bg-black rounded-full"
              style="width: {(1 - timeLeft / (activeTask.duration * 60)) * 100}%;"
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
    {/if}
  </div>
  <div class="flex-1">
    <h2 class="text-xl font-bold mb-4">Tasks</h2>
    {#each tasks as task (task.id)}
      <div class="mb-4 p-4 rounded-lg shadow-lg bg-white border border-gray-300">
        <div class="flex items-center justify-between">
          <div>
            <h3 class="font-semibold">{task.name}</h3>
            <p class="text-sm text-gray-500">{task.duration} minutes</p>
          </div>
          <div class="flex gap-1">
            <button on:click={() => startTask(task)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-gray-700">
              <Play class="h-4 w-4" />
            </button>
            <button on:click={() => resetTask(task)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-gray-700">
              <RotateCcw class="h-4 w-4" />
            </button>
          </div>
        </div>
      </div>
    {/each}
  </div>
</div>