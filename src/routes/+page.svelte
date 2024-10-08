<script lang="ts">
  import { onMount } from "svelte";
  import TaskForm from "$lib/components/TaskForm.svelte";
  import TaskItem from "$lib/components/TaskItem.svelte";
  import ActiveTask from "$lib/components/ActiveTask.svelte";
  import Dropdown from "$lib/components/Dropdown.svelte";

  interface Task {
    name: string;
    duration: number | null;
    id: number;
    description: string;
    status: 'todo' | 'in-progress' | 'paused' | 'done';
    date: string;
  }

  let tasks: Task[] = [];
  let newTask: Task = { name: "", duration: null, id: Date.now(), description: "", status: 'todo', date: new Date().toISOString() }; 
  let activeTask: Task | null = null;
  let timeLeft = 0;
  let isRunning = false;
  let finishedTasks: Task[] = [];
  let filterFinishedTasks: string[] = ['Today', 'This Month', 'All Time'];
  let selectedFilter: string = 'All Time';

  function saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(tasks));
    localStorage.setItem('activeTask', JSON.stringify(activeTask));
    localStorage.setItem('timeLeft', JSON.stringify(timeLeft));
    localStorage.setItem('isRunning', JSON.stringify(isRunning));
    localStorage.setItem('selectedFilter', selectedFilter);
  }

  function loadTasks() {
    const storedTasks = localStorage.getItem('tasks');
    const storedActiveTask = localStorage.getItem('activeTask');
    const storedTimeLeft = localStorage.getItem('timeLeft');
    const storedIsRunning = localStorage.getItem('isRunning');
    const storedSelectedFilter = localStorage.getItem('selectedFilter');

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

    if (storedSelectedFilter) {
      selectedFilter = JSON.parse(storedSelectedFilter);
    }

    applyFilter(selectedFilter);
  }

  function addTask() {
    if (newTask.name && newTask.duration) {
      tasks = [...tasks, { ...newTask, id: Date.now() }];
      newTask = { name: "", duration: null, id: 0, description: "", status: 'todo', date: new Date().toISOString() };
      sortTasks();
      applyFilter(selectedFilter);
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
    applyFilter(selectedFilter);
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
    applyFilter(selectedFilter);
    saveTasks();
  }

  function sortTasks() {
    tasks = tasks.sort((a, b) => {
      if (a.status === 'done' && b.status !== 'done') return 1;
      if (a.status !== 'done' && b.status === 'done') return -1;
      return 0;
    });
  }

  function applyFilter(filter: string) {
    const today = new Date().toISOString().slice(0, 10);
    const doneTasks = tasks.filter(task => task.status === 'done');

    switch (filter) {
      case 'Today':
        finishedTasks = doneTasks.filter(task => task.date.slice(0, 10) === today);
        break;
      case 'This Month':
        finishedTasks = doneTasks.filter(task => task.date.slice(0, 7) === today.slice(0, 7));
        break;
      default: // 'All Time'
        finishedTasks = doneTasks;
    }
    selectedFilter = filter;
  }

  function handleSelect(event: CustomEvent<string>) {
    applyFilter(event.detail);
    saveTasks();
  }

  // New function to export tasks
  function exportTasks() {
    const tasksToExport = JSON.stringify(tasks, null, 2);
    const blob = new Blob([tasksToExport], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'tasks.json';
    a.click();
    URL.revokeObjectURL(url);
  }

  // New function to import tasks
  function importTasks(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e: ProgressEvent<FileReader>) => {
        const content = e.target?.result as string;
        try {
          const importedTasks = JSON.parse(content);
          tasks = importedTasks.map((task: any) => ({
            ...task,
            status: task.status || 'todo',
            date: task.date || new Date().toISOString()
          }));
          sortTasks();
          applyFilter(selectedFilter);
          saveTasks();
        } catch (error) {
          console.error('Error parsing imported tasks:', error);
          alert('Invalid file format. Please select a valid JSON file.');
        }
      };
      reader.readAsText(file);
    }
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
          applyFilter(selectedFilter);
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
    <TaskForm {newTask} {addTask} {exportTasks} {importTasks} />
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
    <div class="flex justify-between items-center mb-2">
      <h2 class="text-xl font-bold">
        Finished Tasks
        <span class="font-light">({finishedTasks.reduce((acc, task) => acc + task.duration!, 0)} minutes)</span>
      </h2>
      <Dropdown
        options={filterFinishedTasks}
        selectedOption={selectedFilter}
        on:select={handleSelect}
      />
    </div>
    
    <div class="scrollable">
      {#each finishedTasks as task (task.id)}
        <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} {pauseTask} />
      {/each}

      {#if finishedTasks.length === 0}
        <div class="text-gray-900 text-sm">No finished tasks yet</div>
      {/if}
    </div>
  </div>
</div>