<script lang="ts">
	export let stream: MediaStream;

	// const audioTrack = new MediaStreamTrack()
	// audioTrack.
	// context.createMediaElementSource
	// const audio = new Audio()
	const audioContext = new AudioContext();
	// audioContext.
	const audioURL = 'src/lib/sounds/magic-escape-room.mp3';
	const audio = new Audio(audioURL);
	// audio.play();
	const source = audioContext.createMediaElementSource(audio);
	const dest = audioContext.createMediaStreamDestination();
	const [originalAudio] = stream.getAudioTracks();
	source.connect(dest);
	// const originalSource = audioContext.createMediaStreamSource(originalAudio)
	const originalSource = audioContext.createMediaStreamSource(stream);
	originalSource.connect(dest);
	const [combinedAudio] = dest.stream.getAudioTracks();
	stream.removeTrack(originalAudio);
	stream.addTrack(combinedAudio);

	// console.log(audio.captureStream());

	// stream.addTrack

	let recording = false;

	const recorder = new MediaRecorder(stream, { mimeType: 'video/webm' });

	let video: HTMLVideoElement;
	let recordingSrc = '';

	recorder.addEventListener('dataavailable', async (e) => {
		const blob = e.data;
		// Save blob?
		recordingSrc = URL.createObjectURL(blob);
	});

	$: if (video && recordingSrc === '') video.srcObject = stream;
</script>

{#if recordingSrc}
	<!-- svelte-ignore a11y-media-has-caption -->
	<video src={recordingSrc} autoplay />

	<button
		on:click={() => {
			recordingSrc = '';
		}}
	>
		Restart
	</button>
{:else}
	<!-- svelte-ignore a11y-media-has-caption -->
	<video bind:this={video} class="screen" autoplay muted />

	<button
		on:click={() => {
			if (recording) {
				recorder.stop();
			} else {
				recorder.start();
				audio.currentTime = 0;
				audio.play();
			}
			recording = !recording;
		}}
	>
		{recording ? 'Stop' : 'Start'} recording
	</button>

	{#if recording}
		<audio src={audioURL} autoplay />
	{/if}
{/if}

<style>
	video.screen {
		scale: -1 1;
	}
</style>
