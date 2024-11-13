<script>
    export let position; // The position of the tick on the axis
    export let axis; // 'x' or 'y' to determine where the tick is on
    export let label; // The label to display for the tick (e.g., a year or value)
    export let axisLength; // Length of the axis (to know where the tick should appear)
  </script>
  
  {#if axis === 'x'}
    <!-- X-axis ticks -->
    <g>
      <line
        x1={position}
        x2={position}
        y1={axisLength}
        y2={axisLength + 5}
        stroke="black"
      />
      <text
        x={position}
        y={axisLength + 20}
        text-anchor="middle"
        font-size="13"
      >
        {label}
      </text>
    </g>
  {:else}
    <!-- Y-axis ticks -->
    <g>
      <line
        x1={axisLength}
        x2={axisLength - 5}
        y1={position}
        y2={position}
        stroke="black"
      />
      <text
        x={axisLength - 15}
        y={position}
        text-anchor="end"
        font-size="13"
      >
        {label}
      </text>
    </g>
  {/if}
  
  <style>
    text {
          font-family: "Ruda", sans-serif; 
      }
  </style>