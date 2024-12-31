<script>
  import Race from '../../../Race.svelte'  
	import Nav from '../../../Nav.svelte'
  
	/** @type {{ data: import('./$types').PageData }} */
	let { data } = $props();

</script>
<Nav/>
<Race id={data.params.id} />
