<script lang="ts">
	import { Math as M } from 'svelte-math';

	type Props = {
		formula: string;
		number: string;
	};

	let { formula, number }: Props = $props();
</script>

<M latex={`${formula} = ${number}`} displayMode />
