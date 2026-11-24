<script lang="ts">
	import Formula from '#lib/components/Formula.svelte';
	import { Computer, Mail } from '@lucide/svelte';

	type Param = {
		Wt: number;
		Hct: number;
		Uf: number;
		Qb: number;
		QrfIndex: number;
		hrs: number;
		CRRTNa: number;
		targetNa: number;
	};

	let parameters = $state<Param>({
		Wt: 90,
		Hct: 25,
		Uf: 200,
		Qb: 200,
		QrfIndex: 25,
		hrs: 24,
		CRRTNa: 140,
		targetNa: 140
	});

	const Qrf = $derived(parameters.QrfIndex * parameters.Wt);
	const plasmaFlow = $derived((parameters.Qb * 60 * (100 - parameters.Hct)) / 100);
	const filtrationFraction = $derived(((Qrf + parameters.Uf) / (Qrf + plasmaFlow)) * 100);
	const dilutionFactor = $derived(plasmaFlow / (plasmaFlow + Qrf));
	const Kurea = $derived((Qrf + parameters.Uf) * dilutionFactor);
	const dosing = $derived(((Kurea / parameters.Wt) * parameters.hrs) / 24);
	const D5WRate = $derived(((parameters.CRRTNa - parameters.targetNa) / parameters.targetNa) * Qrf);
</script>

{#snippet snippetInput(
	title: string,
	description: string,
	parameters: Param,
	key: keyof Param,
	min: number,
	max: number,
	step: number,
	unit: string
)}
	<fieldset>
		<legend class="fieldset-legend">{title} <span class="font-normal">{description}</span></legend>
		<div class="grid grid-cols-12 gap-golden-lg items-center">
			<input
				bind:value={parameters[key]}
				type="range"
				{min}
				{max}
				{step}
				class="range w-full range-primary col-span-7 md:col-span-9"
			/>
			<input type="number" bind:value={parameters[key]} class="input col-span-3 md:col-span-2" />
			<div class="col-span-2 md:col-span-1">{unit}</div>
		</div>
	</fieldset>
{/snippet}

{#snippet snippetCalc(name: string, value: number, roundTo: number, unit: string)}
	<fieldset>
		<legend class="fieldset-legend">{name}</legend>
		<span class="text-lg">{value.toFixed(roundTo)}</span>

		<span class="text-base-content/80">{unit}</span>
	</fieldset>
{/snippet}

<div class="card">
	<h1 class="card-title text-primary">CRRT Calculator</h1>
	<p class="prose max-w-none pb-4 card-body">Use this page to quickly calculate CRRT parameters.</p>

	<div class="flex flex-col gap-y-4 card-body">
		{@render snippetInput('Patient Ideal Body Weight', '', parameters, 'Wt', 10, 300, 1, 'kg')}
		{@render snippetInput('Hematocrit', '', parameters, 'Hct', 1, 100, 1, '%')}
		{@render snippetInput(
			'Replacement Fuid Rate (Qrf)',
			'25 to 35 mL/kg/hr',
			parameters,
			'QrfIndex',
			0,
			50,
			1,
			'mL/kg/hr'
		)}
		{@render snippetInput(
			'Ultra Filtration (UF)',
			'0 to 300 mL/hr',
			parameters,
			'Uf',
			0,
			500,
			1,
			'mL/hr'
		)}
		{@render snippetInput(
			'Blood Flow Rate (Qb)',
			'200 to 300 mL/hr',
			parameters,
			'Qb',
			0,
			500,
			1,
			'mL/min'
		)}
		{@render snippetInput(
			'Hours of CRRT per Day',
			'0 to 24 hours',
			parameters,
			'hrs',
			0,
			24,
			1,
			'hr(s)'
		)}
		{@render snippetInput('CRRT Na', '', parameters, 'CRRTNa', 100, 160, 1, '')}
		{@render snippetInput('Target Na', '', parameters, 'targetNa', 100, 160, 1, '')}
	</div>

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-golden-md card-body">
		{@render snippetCalc('Plasma Flow Rate', plasmaFlow, 0, 'mL/hr')}
		{@render snippetCalc('Replacement Fluid Rate', Qrf, 0, 'mL/hr')}
		{@render snippetCalc('Filtration Fraction', filtrationFraction, 1, '%')}
		{@render snippetCalc('Dilution Factor', dilutionFactor, 3, '')}
		{@render snippetCalc('Kurea', Kurea, 0, 'mL/hr')}
		{@render snippetCalc('CVVH Dosing', dosing, 2, 'mL/kg/hr')}
		{@render snippetCalc('D5W Rate', D5WRate, 0, 'mL/hr')}
	</div>

	<!-- <Formula formula="Q_RF * Patient Weight" number={`${parameters.QrfIndex} * ${parameters.Wt}`} /> -->

	<div class="mt-4 justify-end text-sm text-right flex gap-x-2">
		Made by Kang.
		<a href="mailto:kang.xiang@wfusm.edu"><Mail /></a>
		<a href="https://github.com/kangruixiang/CRRTcalculator"><Computer /></a>
	</div>
</div>
