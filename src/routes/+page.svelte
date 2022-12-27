<script lang="ts">
  import Greet from '$lib/Home.svelte';
  import CallRoom from '$lib/CallRoom.svelte';
  import { onMount } from 'svelte';

  let url: string | null;

  onMount(() => {
    const storedUrl = localStorage.getItem('CALL_ROOM_URL');
    url = storedUrl;
  });
</script>

<div>
  {#if !url}
    <Greet
      on:submit={e => {
        localStorage.setItem('CALL_ROOM_URL', e.detail);
        url = e.detail;
      }}
    />
  {:else}
    <CallRoom
      {url}
      on:leave={() => {
        localStorage.removeItem('CALL_ROOM_URL');
        url = null;
      }}
    />
  {/if}
</div>

<style>
</style>
