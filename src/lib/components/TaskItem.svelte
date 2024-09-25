<script lang="ts">
    import { onMount } from 'svelte';
    import Play from "lucide-svelte/icons/play";
    import Pause from "lucide-svelte/icons/pause";
    import Trash2 from "lucide-svelte/icons/trash-2";
    import Check from "lucide-svelte/icons/check";

    interface Task {
        name: string;
        duration: number | null;
        id: number;
        description: string;
        status: 'todo' | 'in-progress' | 'paused' | 'done';
        date: string; // Optional date field
    }

    export let task: Task;
    export let startTask: (task: Task) => void;
    export let pauseTask: (task: Task) => void;
    export let deleteTask: (id: number) => void;
    export let markTaskAsDone: (task: Task) => void;

    let internalStatus: Task['status'];

    onMount(() => {
        internalStatus = task.status;
    });

    function handleStartTask(): void {
        internalStatus = 'in-progress';
        startTask(task);
    }

    function handlePauseTask(): void {
        internalStatus = 'paused';
        pauseTask(task);
    }

    function handleMarkAsDone(): void {
        internalStatus = 'done';
        markTaskAsDone(task);
    }

    $: formattedDate = new Date(task.date).toLocaleDateString(undefined, {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
    });

    $: statusColor = {
        'todo': 'bg-gray-500',
        'in-progress': 'bg-blue-500',
        'paused': 'bg-yellow-500',
        'done': 'bg-green-500'
    }[internalStatus];
</script>

<div class="mb-4 p-4 rounded-lg shadow-lg bg-white border border-gray-500">
    <h3 class="font-semibold">{task.name}</h3>
    <div class="flex items-center justify-between pt-2">
        <div>
            <p class="text-sm text-gray-500">{formattedDate} - {task.duration} minutes</p>
            <p class="text-sm text-gray-500 pt-1">
                Status: 
                <span 
                    class="py-0.5 px-1 rounded-sm text-xs text-white {statusColor}"
                >
                    {internalStatus}
                </span>
            </p>
        </div>
        <div class="flex gap-1">
            {#if internalStatus === 'done'}
                <button disabled class="bg-transparent text-green-500 font-bold py-2 px-2 rounded">
                    <Check class="h-4 w-4" />
                </button>
            {:else if internalStatus === 'in-progress'}
                <button on:click={handlePauseTask} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-yellow-500">
                    <Pause class="h-4 w-4" />
                </button>
                <button on:click={handleMarkAsDone} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-green-500">
                    <Check class="h-4 w-4" />
                </button>
            {:else}
                <button on:click={handleStartTask} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-blue-500">
                    <Play class="h-4 w-4" />
                </button>
                <button on:click={handleMarkAsDone} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-green-500">
                    <Check class="h-4 w-4" />
                </button>
            {/if}
            <button on:click={() => deleteTask(task.id)} class="bg-transparent text-black font-bold py-2 px-2 rounded hover:text-red-500">
                <Trash2 class="h-4 w-4" />
            </button>
        </div>
    </div>
</div>