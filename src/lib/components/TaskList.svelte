<script lang="ts">
  import TaskItem from './TaskItem.svelte';
  export let tasks;
  export let startTask;
  export let deleteTask;
  export let markTaskAsDone;
</script>

<h2 class="text-xl font-bold mb-4">Tasks</h2>
{#each tasks as task (task.id)}
  <TaskItem {task} {startTask} {deleteTask} {markTaskAsDone} />
{/each}
