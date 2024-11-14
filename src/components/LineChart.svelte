<script>
    // @ts-nocheck
    
        import { scaleLinear } from "d3-scale";
        import { line } from "d3-shape";
        import Tick from "./Tick.svelte";
      
        const paddings = {
          top: 50,
          left: 50,
          right: 170,
          bottom: 70,
        };
      
        export let chartWidth;
        export let chartHeight;
        export let data;
        export let xVar;
        export let yVars;
      
        let hoveredRegion = null;  // Track hovered region
      
        // Tooltip state
        let tooltip = { visible: false, content: "", x: 0, y: 0 };
      
        // Reactive scale definitions
        $: xScale = scaleLinear()
          .domain([Math.min(...data.map(d => d[xVar])), Math.max(...data.map(d => d[xVar]))])
          .range([paddings.left, chartWidth - paddings.right]);
      
        $: yScale = scaleLinear()
          .domain([Math.min(...data.map(d => d[yVars[0]])), Math.max(...data.map(d => d[yVars[0]]))])
          .range([chartHeight - paddings.bottom, paddings.top])
          .nice(10);
      
        // Line generator (must be reactive as it depends on xScale and yScale)
        const lineGenerator = line()
          .x(d => xScale(d[xVar]))
          .y(d => yScale(d[yVars[0]]));
        
        // Get unique regions
        const regions = Array.from(new Set(data.map(d => d.Region)));
      
        // Predefined colors for regions
        const colorPalette = ['#e6194B', '#3cb44b',  '#4363d8', '#f58231', '#42d4f4', '#f032e6', '#469990', '#9A6324', '#800000', '#000000'];
        const regionColors = {};
        regions.forEach((region, index) => {
          regionColors[region] = colorPalette[index];
        });
      
        // Prepare path data for each region
        $: paths = regions.map(region => {
          const filteredData = data.filter(d => d.Region === region);
          return {
            region,
            pathData: lineGenerator(filteredData),
            color: regionColors[region],  // Assign color based on region
            data: filteredData
          };
        });
      
        // Handle mouseover event for the tooltip
        const handleMouseOver = (datum, event, region) => {
          hoveredRegion = region;  // Set the hovered region
          tooltip = {
            visible: true,
            content: `Year: ${datum.Year}, Income: $${datum.Income}`,
            x: event.clientX + 10,
            y: event.clientY + 10
          };
        };
      
        const handleMouseOut = () => {
          hoveredRegion = null;  // Reset hovered region
          tooltip = { ...tooltip, visible: false };
        };
      
        // Get the last data point for label positioning
        const getLastDataPoint = (region) => {
          const regionData = data.filter(d => d.Region === region);
          return regionData[regionData.length - 1];
        };
      
        // Adjust label position for "MENA" region to avoid vertical overlap
        const adjustVerticalPosition = (region, currentLabelY) => {
          if (region === "MENA") {
            return currentLabelY - 7;  // Move "MENA" label 7 pixels higher
          }
          if (region === "Eastern Europe") {
            return currentLabelY - 7;  // Move "Eastern Europe" label 7 pixels higher as well
          }
          return currentLabelY;  // Keep other regions unchanged
        };
      
        // Define custom tick values
        const customTicks = [1850, 1900, 1950, 2000];
      
      </script>
      
      <svg width={chartWidth} height={chartHeight}>
        <!-- Title and Subtitle -->
        <text 
        x={chartWidth / 2} 
        y={paddings.top / 2} 
        text-anchor="middle" 
        font-size="clamp(16px, 3vw, 28px)"  
        font-weight="bold"
        class="chart-title"
        >
        Average Income Per Adult
        </text>
    
        <text 
        x={chartWidth / 2} 
        y={paddings.top / 0.9} 
        text-anchor="middle" 
        font-size="clamp(12px, 2vw, 16px)"  
        font-weight="normal"
        class="chart-subtitle"
        >
        Euro € | ppp | constant (2023)
        </text>
        <!-- Chart  -->
        <g>
          <!-- Loop through the paths and render one path for each region -->
          {#each paths as { region, pathData, color, data }}
            {#if pathData}
              <!-- svelte-ignore a11y-mouse-events-have-key-events -->
              <!-- svelte-ignore a11y-no-static-element-interactions -->
              <path
                d={pathData}
                fill="none"
                stroke={region === hoveredRegion ? color : "gray"}  
                stroke-width={4}
                opacity={region === hoveredRegion ? 1 : 0.3}  
                on:mouseover={(event) => handleMouseOver(data.find(d => xScale(d[xVar]) <= event.offsetX && xScale(d[xVar]) + 5 >= event.offsetX), event, region)}
                on:mouseout={handleMouseOut}
              />
      
              <!-- Permanent Region Labels (Always visible) -->
              <text
                x={xScale(getLastDataPoint(region)[xVar]) + 10}
                y={adjustVerticalPosition(region, yScale(getLastDataPoint(region)[yVars[0]]))}
                fill={regionColors[region]}  
                font-size="14"
                font-weight="bold"
                alignment-baseline="middle"
              >
                {region}  <!-- Display the region name -->
              </text>
            {/if}
          {/each}
        </g>
      
        <!-- Draw axes -->
        <g>
          <line
            x1={paddings.left}
            x2={chartWidth - paddings.right}
            y1={chartHeight - paddings.bottom}
            y2={chartHeight - paddings.bottom}
            stroke="black"
            stroke-width="2"
          />
          {#each customTicks as tick}
            <Tick
              position={xScale(tick)}
              axis="x"
              label={tick}
              scale={xScale}
              axisLength={chartHeight - paddings.bottom}
            />
          {/each}
      
          <line
            x1={paddings.left}
            x2={paddings.left}
            y1={paddings.top}
            y2={chartHeight - paddings.bottom}
            stroke="black"
            stroke-width="2"
          />
          {#each yScale.ticks(10) as tick}
            <Tick
              position={yScale(tick)}
              axis="y"
              label={tick}
              scale={yScale}
              axisLength={paddings.left}
            />
          {/each}
        </g>
        <!-- Add captions -->
        <text 
            x={paddings.left} 
            y={chartHeight - paddings.bottom + 50} 
            font-size="12"
            text-anchor="start"
            >
            Visualization: Alexandre Bovey
        </text>
    
        <text 
            x={paddings.left} 
            y={chartHeight - paddings.bottom + 68} 
            font-size="12"
            text-anchor="start"
            >
            Data:  World Inequality Database, ranging from 1820 to 2023
        </text>
      </svg>
      
      <!-- Custom Tooltip -->
      {#if tooltip.visible}
        <div
          class="tooltip"
          style="left: {tooltip.x}px; top: {tooltip.y}px;"
        >
          {tooltip.content}
        </div>
      {/if}
      
      <style>
        .tooltip {
          position: absolute;
          background-color: rgba(0, 0, 0, 0.7);
          color: white;
          padding: 5px 10px;
          border-radius: 3px;
          pointer-events: none;
          font-size: 14px;
        }
      
        /* Optional smooth transition for opacity and stroke */
        path {
          transition: opacity 0.3s ease, stroke 0.3s ease;
        }

        @font-face {
          font-family: 'Ruda';
          src: url('./assets/Ruda-VariableFont_wght.ttf') format('ttf');
          font-weight: normal;
          font-style: normal;
        }
        @font-face {
          font-family: 'Rockwell';
          src: url('./assets/ROCK.ttf') format('ttf');
          font-weight: normal;
          font-style: normal;
        }
    
        /* General style for the chart title */
        .chart-title {
            font-family: "Rockwell", sans-serif;
            fill: #333;  /* Dark color for better contrast */
            transition: font-size 0.3s ease;  /* Smooth transition for font size */
        }
    
        .chart-subtitle {
            font-family: "Ruda", sans-serif; /* Same font as the labels for consistency */
            fill: #333;
            transition: font-size 0.3s ease;
        }
    
        /* Font for all other text elements (labels, ticks, etc.) */
        text {
            font-family: "Ruda", sans-serif; /* Use Ruda for the rest of the text */
            
        }
    
        /* Responsive title */
        @media (max-width: 1200px) {
            .chart-title {
            font-size: 20px; /* Smaller font size for medium screens */
            }
        }
    
        @media (max-width: 768px) {
            .chart-title {
            font-size: 18px;  /* Smaller font size for small screens */
            }
        }
    
        @media (max-width: 480px) {
            .chart-title {
            font-size: 16px;  /* Even smaller font size for extra small screens */
            }
        }
        
    
    
        
      </style>
      