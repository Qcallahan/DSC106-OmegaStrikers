<script>
	let tempData = [];

	onMount(async () => {

		const res = await fetch('global.csv'); 

		const csv = await res.text();

		tempData = d3.csvParse(csv, d3.autoType)

		console.log(tempData);

	});

</script>

<main>

</main>
