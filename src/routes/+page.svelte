<script lang="ts">
	import { browser } from '$app/environment';
	import Recorder from './Recorder.svelte';

	let streamPromise: Promise<MediaStream>;

	if (browser) {
		streamPromise = navigator.mediaDevices.getUserMedia({
			video: true,
			audio: true
		});
	}
</script>

{#if browser}
	{#await streamPromise then stream}
		<Recorder {stream} />
	{:catch}
		You need to enable your camera
	{/await}
{/if}
