<script>
  import Race from '../../../Race.svelte'
  
	/** @type {{ data: import('./$types').PageData }} */
	let { data } = $props();

</script>

<Race id={data.params.id} />
