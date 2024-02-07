<script lang="ts">
	import * as Tone from 'tone';
	import { Pane, Slider, Button, Monitor } from 'svelte-tweakpane-ui';

	const waveform = 'fatsawtooth';
	const notes = ['C5', 'A3', 'D4', 'G4', 'A4', 'F4'];
	const filter_min = 100;
	const filter_max = 8000;

	let filter_freq = 200;
	let effect_wet = 0.5;

	let meterValue: number | number[] = 0;
	let synth: Tone.Synth;
	let volume: Tone.Volume;
	let filter: Tone.Filter;
	let effect: Tone.FeedbackDelay;
	let reverb: Tone.Reverb;
	let meter: Tone.Meter;

	function startTone() {
		if (Tone.context.state !== 'running') Tone.start();

		if (synth) synth.triggerAttackRelease(notes[Math.floor(Math.random() * notes.length)], '8n');

		filter.frequency.value = filter_freq;
		effect.wet.value = effect_wet;
	}

	function toneInit(node: HTMLDivElement) {
		synth = new Tone.Synth({
			oscillator: {
				type: `${waveform}`,
				count: 3,
				spread: 30
			},
			envelope: {
				attack: 0.001,
				decay: 0.1,
				sustain: 0.5,
				release: 0.1,
				attackCurve: 'exponential'
			}
		});

		volume = new Tone.Volume(-3);

		filter = new Tone.Filter();
		filter.type = 'lowpass';

		effect = new Tone.FeedbackDelay(0.4, 0.85);

		reverb = new Tone.Reverb({
			decay: 0.5,
			wet: 0.5,
			preDelay: 0.1
		});
		reverb.generate();

		synth.connect(volume);
		volume.connect(effect);
		effect.connect(reverb);
		reverb.connect(filter);
		filter.connect(meter);
		meter.toDestination();

		filter.frequency.value = filter_freq;
		effect.wet.value = effect_wet;
		meterValue = meter.getValue();
	}
</script>

{#if meterValue}
	<Pane>
		{#if Array.isArray(meterValue)}
			{#each meterValue as mv}
				<Monitor label="Meter" value={mv} graph={true} />
			{/each}
		{:else}
			<Monitor label="Meter" value={meterValue} graph={true} />
		{/if}
		<Slider label="Filter cutoff" bind:value={filter_freq} min={filter_min} max={filter_max} />
		<Slider label="Effect wet" bind:value={effect_wet} min={0} max={1} />
		<Button label="Play" on:click={startTone} />
	</Pane>
{/if}

<div use:toneInit style="transform: scale({meterValue})" />

<style>
	div {
		width: 100px;
		height: 100px;
		background-color: teal;
	}
</style>
