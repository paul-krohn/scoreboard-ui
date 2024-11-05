<script>
  import Race from '../../../Race.svelte'
  
	/** @type {{ data: import('./$types').PageData }} */
	let { data } = $props();

</script>

<h1>Race</h1>

<Race id={data.params.id} />
