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
        status: task.status || 'todo'
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
      newTask = { name: "", duration: 0, id: 0, description: "", status: 'todo' };
      sortTasks();
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
    saveTasks();
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
        activeTask = null;
      }
      saveTasks();
    }, 1000);
  }
</script>

<style>
  .scrollable {
    max-height: 95vh; /* Set the height to 80vh */
    overflow-y: auto;
    scrollbar-width: none; /* For Firefox */
  }

  .scrollable::-webkit-scrollbar {
    display: none; /* For Chrome, Safari, and Opera */
  }
</style>

<div class="max-w-4xl mx-auto p-4 flex gap-4 flex-row justify-between">
  <div class="flex-1">
    <TaskForm {newTask} {addTask} />
    {#if activeTask}
      <ActiveTask {activeTask} {timeLeft} {isRunning} {toggleTimer} />
    {/if}
  </div>
  <div class="flex-1">
    <h2 class="text-xl font-bold mb-4">Tasks</h2>
    <div class="scrollable">
      {#each tasks as task (task.id)}
        <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} {timeLeft} {activeTask} />
      {/each}
    </div>
  </div>
</div>
