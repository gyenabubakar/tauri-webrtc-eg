<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  let url = '';

  const dispatch = createEventDispatcher();
</script>

<div>
  <form
    class="row"
    on:submit|preventDefault={() => {
      dispatch('submit', url);
    }}
  >
    <input id="greet-input" placeholder="Enter meeting URL..." bind:value={url} />
    <button type="submit"> Start Call </button>
  </form>
</div>
