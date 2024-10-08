<script lang="ts">
  import { createEventDispatcher, onMount } from "svelte";

  export let options: string[] = [];
  export let selectedOption: string = "";

  let isOpen: boolean = false;
  let dropdownRef: HTMLElement;

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

  function handleClickOutside(event: MouseEvent) {
    if (dropdownRef && !dropdownRef.contains(event.target as Node) && isOpen) {
      isOpen = false;
    }
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Escape" && isOpen) {
      isOpen = false;
    }
  }

  onMount(() => {
    document.addEventListener("click", handleClickOutside);
    document.addEventListener("keydown", handleKeydown);
    return () => {
      document.removeEventListener("click", handleClickOutside);
      document.removeEventListener("keydown", handleKeydown);
    };
  });
</script>

<div class="relative inline-block text-left" bind:this={dropdownRef}>
  <div>
    <button
      type="button"
      on:click={toggleDropdown}
      class="inline-flex items-center justify-center w-full pl-4 bg-transparent text-base pt-2 pr-1 focus:outline-none"
      aria-haspopup="true"
      aria-expanded={isOpen}
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="size-4"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M8.25 15 12 18.75 15.75 15m-7.5-6L12 5.25 15.75 9"
        />
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
