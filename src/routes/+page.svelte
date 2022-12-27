<script lang="ts">
  import Greet from '$lib/Home.svelte';
  import CallRoom from '$lib/CallRoom.svelte';

  let url: string | undefined;
</script>

<div class="row">
  <Greet
    on:submit={e => {
      url = e.detail;
    }}
  />

  {#if url}
    <CallRoom {url} />
  {/if}
</div>

<style>
</style>
