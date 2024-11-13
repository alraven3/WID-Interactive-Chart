<script>
  import { onMount } from "svelte";
  import ChartContainer from "./components/ChartContainer.svelte";
  import WID_data from "./data/WID_data";

  // Transform data: Convert Year to a number and Income to a float
  const incomes_data = WID_data.map(d => ({
      ...d, // Spread the original data
      Year: +d.Year, // Convert Year to a number
      Income: parseFloat(d.Income), // Convert Income to a float
  }));

  const data = incomes_data.slice(0, 1270.5);  // Slice to just the first 1270 items

  let width = 1000;
  let height = width * 0.5625;  // 16:9 aspect ratio (height = width * 9 / 16)

const MIN_HEIGHT = 400;  // Minimum height for the chart (e.g., 400px)

  // Dynamically update width and height based on the window size
  const updateDimensions = () => {
      width = window.innerWidth * 0.8;  // 80% of window width
      height = Math.max(window.innerHeight * 0.7, MIN_HEIGHT); // 60% of window height
  };

  // Update dimensions on mount and on window resize
  onMount(() => {
      updateDimensions();
      window.addEventListener("resize", updateDimensions);
      return () => window.removeEventListener("resize", updateDimensions); // Cleanup
  });

</script>

<main>
  <!-- <h1>Hello!</h1> -->
  <div id="chart">
      <ChartContainer 
          type={"lineChart"} 
          chartProps={{
              chartWidth: width,
              chartHeight: height,
              data: data,
              xVar: "Year", 
              yVars: ["Income"]
          }} 
      />
  </div>
</main>

<style>
  main {
      text-align: center;
      padding: .5em;
      margin: auto;
  }

  /* h1 {
      color: #ff3e00;
      text-transform: uppercase;
      font-size: 20px;
      font-weight: 50;
  } */

  

  /* Ensure responsiveness on smaller screens */
  @media (max-width: 640px) {
      main {
          max-width: 100%;
      }
  }
</style>
