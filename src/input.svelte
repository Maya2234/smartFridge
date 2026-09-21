<script>
  import { fly } from 'svelte/transition';

  export let value = '';
  export let onSubmit = () => {};

  /** @param {number} v */
  const select = (v) => {
    if (v === 0 && !value) return;
    if (value.length >= 6) return;

    value += v;
  };

  function handleSubmit() {
    onSubmit();
  }
</script>

<form in:fly={{ y: -10, delay: 120 }} on:submit|preventDefault={handleSubmit}>
  {#each Array.from({ length: 10 }, (_, i) => i) as i}
    <button type="button" on:click={() => select(i)}>
      {i}
    </button>
  {/each}

  <button aria-label="Start timer" disabled={!value} type="submit" id="timer-button">
    <svg viewBox="-50 -50 100 100" width="30" height="30">
      <g
        fill="none"
        stroke="currentColor"
        stroke-width="20"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <path d="M -25 -40 l 60 40 -60 40z" />
      </g>
    </svg>
  </button>
</form>

