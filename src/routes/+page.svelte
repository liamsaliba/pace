<!-- YOU CAN DELETE EVERYTHING IN THIS PAGE -->
<script lang="ts">
	import CurrentPaceLockedToAveragePaceButton from '$lib/CurrentPaceLockedToAveragePaceButton.svelte';
import { RangeSlider, SlideToggle, getToastStore } from '@skeletonlabs/skeleton';

	const toastStore = getToastStore();
	const MARATHON_KM = 42.195;
	const KM_TO_MILES = 0.621371;

	const DISTANCES = [
		{ name: 'Mile', km: 1.60934 },
		{ name: '5 km', km: 5 },
		{ name: '10 km', km: 10 },
		{ name: '15 km', km: 15 },
		{ name: 'Half marathon', km: 21.0975 },
		{ name: 'Marathon', km: MARATHON_KM },
		{ name: '50 km', km: 50 },
		{ name: '100 km', km: 100 },
		{ name: '100 miles', km: 160.934 },
		{ name: '200 miles', km: 321.869 },
		{ name: '1000 miles', km: 1609.34 }
	];

	let time = '0:00:00';
	// let startTime = new Date();
	let startTime = new Date();

	let time_secs = (Date.now() - startTime.getTime()) / 1000;
	$: time_h = Math.floor(time_secs / 60 / 60);
	$: time_mm = Math.floor((time_secs / 60) % 60);
	$: time_ss = Math.floor(time_secs % 60);

	let dist_km = MARATHON_KM / 2;
	$: dist_miles = dist_km * KM_TO_MILES;
	$: average_pace_km_secs = time_secs / dist_km;
	$: average_pace_mile_secs = time_secs / dist_miles;

	const toTimeString = (time_secs: number) =>
		`${Math.floor(time_secs / 60)}'${Math.floor(time_secs % 60)
			.toString()
			.padStart(2, '0')}"`;

	$: average_pace_km = toTimeString(average_pace_km_secs);
	$: average_pace_mile = toTimeString(average_pace_mile_secs);

	let target_distance = MARATHON_KM;
	$: dist_target_pct = (dist_km / target_distance) * 100;

	let current_pace_km_secs = average_pace_km_secs;
	$: current_pace_mile_secs = current_pace_km_secs / KM_TO_MILES;
	$: current_pace_km = toTimeString(current_pace_km_secs);
	$: current_pace_miles = toTimeString(current_pace_mile_secs);

	let predicted_finish = "0:00:00";
	$: remaining_secs = current_pace_km_secs * (target_distance - dist_km);
	$: predicted_finish_secs = time_secs + remaining_secs;
	$: predicted_finish = toTimeHrString(predicted_finish_secs);
	$: predicted_finish_pace_km_secs = predicted_finish_secs / target_distance;
	$: predicted_finish_pace_mile_secs = predicted_finish_pace_km_secs / KM_TO_MILES;
	$: predicted_finish_pace_km = toTimeString(predicted_finish_pace_km_secs);
	$: predicted_finish_pace_mile = toTimeString(predicted_finish_pace_mile_secs);

	let current_pace_locked_to_average_pace = true;
	$: if (current_pace_locked_to_average_pace) {
		current_pace_km_secs = average_pace_km_secs;
	}

	const toTimeHrString = (time_secs: number) => {
		let hh = Math.floor(time_secs / 60 / 60);
		let mm = Math.floor((time_secs / 60) % 60);
		let ss = Math.floor(time_secs % 60);
		let shh = `${hh}`.padStart(1, '0');
		let smm = `${mm}`.padStart(2, '0');
		let sss = `${ss}`.padStart(2, '0');
		return `${shh}:${smm}:${sss}`;
	}

	$: time = toTimeHrString(time_secs);

	const stepTime = () => {
		time_secs = (Date.now() - startTime.getTime()) / 1000;
		if (distticking) {
			dist_km += 1 / current_pace_km_secs;
		}
	}

	// render new time every second
	let timeticking = true;
	let distticking = true;
	let timeinterval: number | undefined = undefined;
	const tickingChanged = () => {
		if (!timeticking) {
			clearInterval(timeinterval);
			timeinterval = undefined;
			distticking = false;
		} else {
			clearInterval(timeinterval);
			timeinterval = setInterval(stepTime, 1000);
		}
	}
	tickingChanged();

	const changeTime = (e) => {
		let newTime: string = e.target.value;
		let match = newTime.match(/^([0-9]{1,3}):([0-9]{1,2}):([0-9]{1,2})$/);
		if (match) {
			time_h = parseInt(match[1]);
			time_mm = parseInt(match[2]);
			if (time_mm >= 60) {
				time_h += Math.floor(time_mm / 60);
				time_mm = time_mm % 60;
			}
			time_ss = parseInt(match[3]);
			time_secs = time_h * 60 * 60 + time_mm * 60 + time_ss;
			startTime = new Date(Date.now() - time_secs * 1000);
			return
		}

		match = newTime.match(/^([0-9]{1,3}):([0-9]{1,2})$/);
		if (match) {
			time_mm = parseInt(match[1]);
			if (time_mm >= 60) {
				time_h = Math.floor(time_mm / 60);
				time_mm = time_mm % 60;
			}
			time_ss = parseInt(match[2]);
			time_secs = time_h * 60 * 60 + time_mm * 60 + time_ss;
			startTime = new Date(Date.now() - time_secs * 1000);
			return
		}

		toastStore.trigger({
			message: 'Invalid time format',
			background: 'variant-filled-error',
			timeout: 5000,
			hideDismiss: true
		});
	};

	let timeFocussed = false;
	let inputTime = time;
	$: if (!timeFocussed) {
		inputTime = time;
	}

	let distFocussed = false;
	let dist_slider: number;
	$: console.log(distFocussed);
	$: if (!distFocussed) {
		console.log("change slider")
		dist_slider = dist_target_pct;
	}

	const change_range = (e) => {
		let newDist = e.target.value;
		dist_km = (newDist / 100) * target_distance;
	};
</script>

<div class="container h-full mx-auto flex justify-center items-center">
	<div class="space-y-2 text-center flex flex-col items-center">
		<!-- <h2 class="h2">Welcome to Skeleton.</h2> -->
		<!-- Animated Logo -->
		<section class="img-bg" />
		<p>Started at: {startTime.toLocaleString()} {startTime.getTime()}</p>

		<SlideToggle name="stopwatch" bind:checked={timeticking} on:change={tickingChanged}>Tick from start time</SlideToggle>
		{#if timeticking}
			<SlideToggle name="distwatch" bind:checked={distticking} on:change={tickingChanged}>Live distance</SlideToggle>
		{/if}
		<input
			class="bg-transparent border-transparent border-b text-center font-bold text-2xl"
			value={inputTime}
			on:change={changeTime}
			on:focus={() => (timeFocussed = true)}
			on:blur={() => (timeFocussed = false)}
		/>

		<p>{dist_km.toFixed(2)} km - {dist_miles.toFixed(2)} miles</p>

		<div class="w-full">
			<RangeSlider
				name="range-slider"
				bind:value={dist_slider}
				min={0}
				max={100}
				step={1}
				on:change={change_range}
				on:focus={() => (distFocussed = true)}
				on:blur={() => (distFocussed = false)}
				><div class="flex justify-between items-center">
					<span>
						{dist_target_pct == 0 || dist_target_pct == 100 ? dist_target_pct : dist_target_pct.toFixed(2)}%
					</span>
					<span>
						<select class="select" bind:value={target_distance}>
							{#each DISTANCES as distance}
								<option value={distance.km}>{distance.name}</option>
							{/each}
						</select>
					</span>
				</div></RangeSlider
			>
		</div>

		{#if dist_km != 0}
			<p>Average pace: {average_pace_km} / km : {average_pace_mile} / mile</p>
			{#if remaining_secs > 0}
				<p class="flex flex-row items-center gap-1"><CurrentPaceLockedToAveragePaceButton bind:current_pace_locked_to_average_pace={current_pace_locked_to_average_pace} showLabel={false} /> <span>Current pace: {current_pace_km} / km : {current_pace_miles} / mile</span></p>
				<p>Predicted finish: <span class='font-bold'>{predicted_finish}</span></p>
				<p>@ total avg pace {predicted_finish_pace_km} / km : {predicted_finish_pace_mile} / mile</p>
			{/if}
			
		{/if} <!-- dist_km != 0 -->


		<!-- <p>{time}</p> -->
		<div class="flex justify-center space-x-2">
			<!-- <a
				class="btn variant-filled"
				href="https://skeleton.dev/"
				target="_blank"
				rel="noreferrer"
			>
				Launch Documentation
			</a> -->
		</div>
		<!-- <div class="space-y-2">
			<p>Try editing the following:</p>
			<p><code class="code">/src/routes/+layout.svelte</code></p>
			<p><code class="code">/src/routes/+page.svelte</code></p>
		</div> -->
	</div>
</div>

<style lang="postcss">
	.img-bg {
		@apply w-64 h-64 md:w-80 md:h-80;
	}
	.img-bg {
		@apply absolute z-[-1] rounded-full blur-[50px] transition-all;
		animation: pulse 5s cubic-bezier(0, 0, 0, 0.5) infinite, glow 5s linear infinite;
	}
	@keyframes glow {
		0% {
			@apply bg-primary-400/50;
		}
		33% {
			@apply bg-secondary-400/50;
		}
		66% {
			@apply bg-tertiary-400/50;
		}
		100% {
			@apply bg-primary-400/50;
		}
	}
	@keyframes pulse {
		50% {
			transform: scale(1.5);
		}
	}
</style>
