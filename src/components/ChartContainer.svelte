<script>
    import LineChart from "./LineChart.svelte";

    export let type;
    export let chartProps;
    const CHART_TYPES = {
        lineChart: LineChart,
    };

    console.log("Chart Type:", type);  // Debugging: check the chart type being passed
    console.log("Chart Props:", chartProps);  // Debugging: check the chart props
</script>

<section class="container">
    <svelte:component this={CHART_TYPES[type]} {...chartProps} />
</section>

<style>
    .container {
        width: 80%; /* Make container take up 80% of the parent width */
        /* height: 40%; */
        /* position: center; */
        border: solid 1px black;
        margin: auto;
        background-color: peachpuff;
        /* tan,darkgseagreen, palegreen... */
        /* Add responsive padding or borders as needed */
        padding: 5%;
    }
</style>
