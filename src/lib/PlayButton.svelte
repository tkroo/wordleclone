<script lang="ts">
  import { onMount } from "svelte";
  let { setup } = $props();
  let b: HTMLButtonElement;
  onMount(() => {
    // const button = document.querySelector('.playagain');
    b.focus();
  });
  
</script>

<button bind:this={b} onclick={setup} class="playagain">Play again</button>

<style>
  .playagain {
    font-size: 1.2rem;
    background-color: #538d4e;
    color: #fff;
    margin-left: 1rem;
    padding: 0.5rem 0.75rem;
    border-radius: 0.25rem;
  }
</style>