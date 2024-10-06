<script lang="ts">
  import { onMount } from "svelte";
  import TaskForm from "$lib/components/TaskForm.svelte";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import ActiveTask from "$lib/components/ActiveTask.svelte";

  interface Task {
    name: string;
    duration: number | null;
    id: number;
    description: string;
    status: 'todo' | 'in-progress' | 'paused' | 'done';
    date: string;
  }

  let tasks: Task[] = [];
  let newTask: Task = { name: "", duration: null, id: 0, description: "", status: 'todo', date: new Date().toISOString() }; 
  let activeTask: Task | null = null;
  let timeLeft = 0;
  let isRunning = false;

  $: minutes = Math.floor(timeLeft / 60);
  $: remainingSeconds = timeLeft % 60;

  function saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(tasks));
    localStorage.setItem('activeTask', JSON.stringify(activeTask));
    localStorage.setItem('timeLeft', JSON.stringify(timeLeft));
    localStorage.setItem('isRunning', JSON.stringify(isRunning));
  }

  function loadTasks() {
    const storedTasks = localStorage.getItem('tasks');
    const storedActiveTask = localStorage.getItem('activeTask');
    const storedTimeLeft = localStorage.getItem('timeLeft');
    const storedIsRunning = localStorage.getItem('isRunning');

    if (storedTasks) {
      tasks = JSON.parse(storedTasks).map((task: any) => ({
        ...task,
        status: task.status || 'todo',
        date: task.date || new Date().toISOString()
      }));
      sortTasks();
    }

    if (storedActiveTask) {
      activeTask = JSON.parse(storedActiveTask);
    }

    if (storedTimeLeft) {
      timeLeft = JSON.parse(storedTimeLeft);
    }

    if (storedIsRunning) {
      isRunning = JSON.parse(storedIsRunning);
    }
  }

  function addTask() {
    if (newTask.name && newTask.duration) {
      tasks = [...tasks, { ...newTask, id: Date.now() }];
      newTask = { name: "", duration: null, id: 0, description: "", status: 'todo', date: new Date().toISOString() };
      sortTasks();
      saveTasks();
    }
  }

  function startTask(task: Task) {
    if (activeTask && activeTask.id !== task.id) {
      return toggleTimer();
    }
    activeTask = task;
    timeLeft = task.status === 'paused' ? timeLeft : (task.duration ? task.duration * 60 : 0);
    isRunning = true;
    task.status = 'in-progress';
    saveTasks();
  }

  function toggleTimer() {
    if (activeTask) {
      isRunning = !isRunning;
      activeTask.status = isRunning ? 'in-progress' : 'paused';
      saveTasks();
    }
  }

  function pauseTask(task: Task) {
    toggleTimer();
  }

  function deleteTask(taskId: number) {
    tasks = tasks.filter(task => task.id !== taskId);
    if (activeTask && activeTask.id === taskId) {
      activeTask = null;
      timeLeft = 0;
      isRunning = false;
    }
    saveTasks();
  }

  function markTaskAsDone(task: Task) {
    task.status = 'done';
    if (activeTask && activeTask.id === task.id) {
      activeTask = null;
      timeLeft = 0;
      isRunning = false;
    }
    sortTasks();
    saveTasks();
  }

  function sortTasks() {
    tasks = tasks.sort((a, b) => {
      if (a.status === 'done' && b.status !== 'done') return 1;
      if (a.status !== 'done' && b.status === 'done') return -1;
      return 0;
    });
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
        if (activeTask) {
          activeTask.status = 'done';
        }
        activeTask = null;
      }
      saveTasks();
    }, 1000);
  }
</script>

<style>
  .scrollable {
    max-height: 90vh;
    overflow-y: auto;
    scrollbar-width: none;
  }

  .scrollable::-webkit-scrollbar {
    display: none;
  }
</style>

<div class="container mx-auto p-4 flex gap-6 flex-row">
  <div class="w-full max-w-md">
    <TaskForm {newTask} {addTask} />
    {#if activeTask}
      <ActiveTask {activeTask} {timeLeft} {isRunning} {toggleTimer} />
    {/if}
  </div>
  <div class="w-full max-w-md">
    <h2 class="text-xl font-bold mb-4">
      Tasks
      <span class="font-light">({tasks.filter(task => task.status !== 'done').reduce((acc, task) => acc + task.duration!, 0)} minutes)</span>
    </h2>
    <div class="scrollable">
      {#each tasks.filter(task => task.status !== 'done') as task (task.id)}
        <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} {pauseTask} />
      {/each}
      {#if tasks.filter(task => task.status !== 'done').length === 0}
        <div class="text-gray-900 text-sm">No tasks added yet</div>
      {/if}
    </div>
  </div>
  <div class="w-full max-w-md">
    <h2 class="text-xl font-bold mb-4">
      Finished Tasks
      <span class="font-light">({tasks.filter(task => task.status === 'done').reduce((acc, task) => acc + task.duration!, 0)} minutes)</span>
    </h2>
    <div class="scrollable">
      {#each tasks.filter(task => task.status === 'done') as task (task.id)}
        <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} {pauseTask} />
      {/each}

      {#if tasks.filter(task => task.status === 'done').length === 0}
        <div class="text-gray-900 text-sm">No finished tasks yet</div>
      {/if}
    </div>
  </div>
</div>