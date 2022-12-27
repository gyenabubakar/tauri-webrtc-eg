<script lang="ts">
  import { onDestroy } from 'svelte';
  import type { DailyParticipant } from '@daily-co/daily-js';

  let videoElement: HTMLVideoElement | undefined;
  let audioElement: HTMLAudioElement | undefined;

  export let participant: DailyParticipant;

  let videoSrc: MediaStream | null;
  let audioSrc: MediaStream | null;

  $: getVideoSrc(participant);
  $: getAudioSrc(participant);

  $: if (videoElement) {
    videoElement.srcObject = videoSrc;
  }
  $: if (audioElement) {
    audioElement.srcObject = audioSrc;
  }

  function getVideoSrc(participant: DailyParticipant) {
    if (participant.video) {
      const videoTrack = participant.tracks.video.persistentTrack;
      const newStream = getUpdatedStream(videoSrc, videoTrack!);
      if (newStream) {
        videoSrc = newStream;
      }
    }
  }

  function getAudioSrc(participant: DailyParticipant) {
    if (participant.local || !participant.audio) return;

    const audioTrack = participant.tracks.audio.persistentTrack;
    const newStream = getUpdatedStream(audioSrc, audioTrack!);
    if (newStream) {
      audioSrc = newStream;
    }
  }

  /** `getUpdatedStream()` updates the given stream with new tracks OR
   * returns an entirely new stream to the caller.
   */
  function getUpdatedStream(stream: MediaStream | null, newTrack: MediaStreamTrack) {
    const existingTracks = stream?.getTracks();
    // If the stream parameter contains no existing tracks,
    // just return a new MediaStream to set. This should
    // only happen the first time the tile is initialized.
    if (!existingTracks || existingTracks.length === 0) {
      return new MediaStream([newTrack]);
    }

    if (existingTracks.length > 1) {
      console.warn(`expected 1 track, found ${existingTracks.length}. Only using the first one.`);
    }

    const existingTrack = existingTracks[0];

    // If existing track is different from the new track,
    // remove the existing track and add the new one.
    if (newTrack.id !== existingTrack.id) {
      stream?.removeTrack(existingTrack);
      stream?.addTrack(newTrack);
    }
    return null;
  }

  onDestroy(() => {
    videoSrc?.getTracks().forEach(track => track.stop());
    audioSrc?.getTracks().forEach(track => track.stop());

    videoSrc = null;
    audioSrc = null;
  });
</script>

<div class="call-participant" class:video={participant.video} class:audio={participant.audio}>
  <audio bind:this={audioElement} autoplay playsinline />

  {#if participant.video}
    <video bind:this={videoElement} autoplay muted playsinline />
  {/if}

  <div class="call-participant__name">
    {#if participant.local}
      You
    {:else}
      {participant.user_name}
    {/if}
  </div>
</div>

<style>
  .call-participant {
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    border-radius: 0.5rem;
    background-color: #1f2937;
    position: relative;
    overflow: hidden;
  }

  .call-participant:not(.video) {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  video {
    pointer-events: none;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  @keyframes ripple {
    from {
      opacity: 1;
      transform: scale3d(0.75, 0.75, 1);
    }
    to {
      opacity: 0;
      transform: scale3d(1.5, 1.5, 1);
    }
  }
</style>
