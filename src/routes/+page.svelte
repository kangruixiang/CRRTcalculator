<script lang="ts">
	import { Computer, Mail } from '@lucide/svelte';

	type Param = {
		Wt: number;
		Hct: number;
		Uf: number;
		Qb: number;
		Qrf: number;
	};

	let parameters = $state<Param>({
		Wt: 90,
		Hct: 25,
		Uf: 200,
		Qb: 200,
		Qrf: 1500
	});

	const plasmaFlow = $derived((parameters.Qb * 60 * (100 - parameters.Hct)) / 100);
	const filtrationFraction = $derived(
		((parameters.Qrf + parameters.Uf) / (parameters.Qrf + plasmaFlow)) * 100
	);
	const dilutionFactor = $derived(plasmaFlow / (plasmaFlow + parameters.Qrf));
	const Kurea = $derived((parameters.Qrf + parameters.Uf) * dilutionFactor);
	const dosing = $derived(Kurea / parameters.Wt);
</script>

{#snippet snippetInput(
	title: string,
	parameters: Param,
	key: keyof Param,
	min: number,
	max: number,
	step: number,
	unit: string
)}
	<fieldset>
		<legend class="fieldset-legend">{title}</legend>
		<div class="grid grid-cols-12 gap-golden-lg items-center">
			<input
				bind:value={parameters[key]}
				type="range"
				{min}
				{max}
				{step}
				class="range w-full range-primary col-span-9"
			/>
			<input type="number" bind:value={parameters[key]} class="input col-span-2" />
			<div class="col-span-1">{unit}</div>
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

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-golden-md card-body">
		{@render snippetCalc('Plasma Flow Rate', plasmaFlow, 0, 'mL/hr')}
		{@render snippetCalc('Filtration Fraction', filtrationFraction, 1, '%')}
		{@render snippetCalc('Dilution Factor', dilutionFactor, 3, '')}
		{@render snippetCalc('Kurea', Kurea, 0, 'mL/hr')}
		{@render snippetCalc('CVVH Dosing', dosing, 2, 'mL/kg/hr')}
	</div>

	<div class="flex flex-col gap-y-4 card-body">
		{@render snippetInput('Patient Ideal Body Weight', parameters, 'Wt', 10, 300, 1, 'kg')}
		{@render snippetInput('Hematocrit', parameters, 'Hct', 1, 100, 1, '%')}
		{@render snippetInput('Replacement Fuid Rate (Qrf)', parameters, 'Qrf', 0, 3000, 1, 'mL/hr')}
		{@render snippetInput('Ultra Filtration (UF)', parameters, 'Uf', 0, 500, 1, 'mL/hr')}
		{@render snippetInput('Blood Flow Rate (Qb)', parameters, 'Qb', 0, 500, 1, 'mL/min')}
	</div>

	<div class="mt-4 justify-end text-sm text-right flex gap-x-2">
		Made by Kang.
		<a href="mailto:kang.xiang@wfusm.edu"><Mail /></a>
		<a href="https://github.com/kangruixiang/CRRTcalculator"><Computer /></a>
	</div>
</div>
