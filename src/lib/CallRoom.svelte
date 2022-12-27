<script lang="ts">
  import { onMount, tick, createEventDispatcher } from 'svelte';
  import Daily, { type DailyCall } from '@daily-co/daily-js';
  import type { DailyParticipant } from '@daily-co/daily-js';
  import CallParticipant from './CallParticipant.svelte';

  export let url: string;

  let call: DailyCall | null = null;
  let participants: DailyParticipant[] = [];
  let loading = true;
  let micOn = true;
  let cameraOn = true;

  const dispatch = createEventDispatcher();

  $: length = getParticipantsLengthInWords(participants.length);

  function getParticipantsLengthInWords(length: number) {
    const words = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'];
    return words[length - 1];
  }

  async function startAndJoinCall() {
    const _call = Daily.createCallObject({
      url,
      startVideoOff: !cameraOn,
      startAudioOff: !micOn,
      dailyConfig: {
        experimentalChromeVideoMuteLightOff: true,
      },
    })
      .on('participant-joined', onUpdateParticipants)
      .on('participant-updated', onUpdateParticipants)
      .on('participant-left', onUpdateParticipants)
      .on('left-meeting', onUpdateParticipants)
      .on('joined-meeting', onJoinedMeeting);

    call = _call;

    tick().then(() => {
      _call.join();
    });
  }

  function onJoinedMeeting() {
    loading = false;
  }

  function onUpdateParticipants() {
    if (call) {
      const _participants = Object.values(call.participants()).filter(({ tracks }) => {
        const { audio, video } = tracks;
        return audio.persistentTrack || video.persistentTrack;
      });
      participants = _participants;
    }
  }

  function toggleVideo() {
    if (call) {
      call.setLocalVideo(!call.localVideo());
      cameraOn = !cameraOn;
    }
  }

  function toggleAudio() {
    if (call) {
      call.setLocalAudio(!call.localAudio());
      micOn = !micOn;
    }
  }

  function onLeaveMeeting(fireEvent = true) {
    return async () => {
      if (call) {
        loading = true;

        call.off('participant-joined', onUpdateParticipants);
        call.off('participant-updated', onUpdateParticipants);
        call.off('participant-left', onUpdateParticipants);
        call.off('left-meeting', onUpdateParticipants);
        call.off('joined-meeting', onJoinedMeeting);

        await call.leave();
        await call.destroy();

        fireEvent && dispatch('leave');
      }
    };
  }

  onMount(() => {
    startAndJoinCall();

    return onLeaveMeeting(false);
  });
</script>

{#if loading}
  <h1>Loading...</h1>
{:else}
  <div class="participants {length}">
    {#each participants as participant (participant.user_id)}
      <div style="aspect-ratio: 16 / 9;">
        <CallParticipant {participant} />
      </div>
    {/each}
  </div>

  <div class="actions">
    <button on:click={onLeaveMeeting()}>end call</button>

    <button class="toggle-video" on:click={toggleVideo}>
      {#if cameraOn}
        turn off video
      {:else}
        turn on video
      {/if}
    </button>

    <button class="toggle-audio" on:click={toggleAudio}>
      {#if micOn}
        turn off Mic
      {:else}
        turn on Mic
      {/if}
    </button>
  </div>
{/if}

<style>
  .participants {
    @apply grid grid-cols-4 gap-2 mb-5;
  }

  .participants.one {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  .participants.two,
  .participants.three,
  .participants.four {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .participants.five,
  .participants.six,
  .participants.seven,
  .participants.eight,
  .participants.nine {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }

  /* Row and Column span */

  .participants.two > div,
  .participants.one > div,
  .participants.three > div {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  .participants.ten > div:nth-child(9) {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  .participants.ten > div:last-of-type {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }

  .actions {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 20px;
  }

  button {
    background-color: red;
    color: white;
    padding: 1rem;
    border-radius: 0.5rem;
    border: none;
    font-size: 1.5rem;
    font-weight: 700;
    outline: none;
  }
  button:not(:last-of-type) {
    margin-right: 10px;
  }

  .toggle-video {
    background-color: green;
  }

  .toggle-audio {
    background-color: blue;
  }
</style>
