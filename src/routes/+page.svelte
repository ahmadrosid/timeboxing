<script lang="ts">
  import { onMount } from "svelte";
  import TaskForm from "$lib/components/TaskForm.svelte";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import ActiveTask from "$lib/components/ActiveTask.svelte";

  interface Task {
    name: string;
    duration: number;
    id: number;
    description: string;
    status: 'todo' | 'in-progress' | 'done';
  }

  let tasks: Task[] = [];
  let newTask: Task = { name: "", duration: 0, id: 0, description: "", status: 'todo' }; 
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
      tasks = JSON.parse(storedTasks).map((task: any) => ({
        ...task,
        status: task.status || 'todo'
      }));
    }
  }

  function addTask() {
    if (newTask.name && newTask.duration) {
      tasks = [...tasks, { ...newTask, id: Date.now() }];
      newTask = { name: "", duration: 0, id: 0, description: "", status: 'todo' };
      saveTasks();
    }
  }

  function startTask(task: Task) {
    activeTask = task;
    timeLeft = task.duration * 60;
    isRunning = true;
    task.status = 'in-progress';
    saveTasks();
  }

  function toggleTimer() {
    isRunning = !isRunning;
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
    saveTasks();
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
    <TaskForm {newTask} {addTask} />
    {#if activeTask}
      <ActiveTask {activeTask} {timeLeft} {isRunning} {toggleTimer} />
    {/if}
  </div>
  <div class="flex-1">
    <h2 class="text-xl font-bold mb-4">Tasks</h2>
    {#each tasks as task (task.id)}
    <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} />
    {/each}
  </div>
</div>
