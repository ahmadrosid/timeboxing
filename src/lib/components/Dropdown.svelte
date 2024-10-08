<script lang="ts">
  import { createEventDispatcher } from "svelte";

  export let options: string[] = [];
  export let selectedOption: string = "";
  export let placeholder: string = "Select an option";

  let isOpen: boolean = false;

  const dispatch = createEventDispatcher<{
    select: string;
  }>();

  function toggleDropdown(): void {
    isOpen = !isOpen;
  }

  function selectOption(option: string): void {
    selectedOption = option;
    isOpen = false;
    dispatch("select", option);
  }
</script>

<div class="relative inline-block text-left">
  <div>
    <button
      type="button"
      on:click={toggleDropdown}
      class="inline-flex iterms-center justify-center w-full pl-4 bg-transparent text-base pt-3"
      aria-haspopup="true"
      aria-expanded={isOpen}
    >
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-5">
        <path stroke-linecap="round" stroke-linejoin="round" d="M12 3c2.755 0 5.455.232 8.083.678.533.09.917.556.917 1.096v1.044a2.25 2.25 0 0 1-.659 1.591l-5.432 5.432a2.25 2.25 0 0 0-.659 1.591v2.927a2.25 2.25 0 0 1-1.244 2.013L9.75 21v-6.568a2.25 2.25 0 0 0-.659-1.591L3.659 7.409A2.25 2.25 0 0 1 3 5.818V4.774c0-.54.384-1.006.917-1.096A48.32 48.32 0 0 1 12 3Z" />
      </svg>
      
    </button>
  </div>

  {#if isOpen}
    <div
      class="origin-top-right absolute right-0 mt-2 w-56 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5 focus:outline-none"
      role="menu"
      aria-orientation="vertical"
      aria-labelledby="options-menu"
    >
      <div class="py-1" role="none">
        {#each options as option}
          <button
            on:click={() => selectOption(option)}
            class="block w-full text-left px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 hover:text-gray-900"
            role="menuitem"
          >
            {option}
          </button>
        {/each}
      </div>
    </div>
  {/if}
</div>
