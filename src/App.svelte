<script>
  import data from "$data/data.js";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { sum, rollups } from "d3-array";
  import {
    forceSimulation,
    forceX,
    forceY,
    forceCollide
  } from "d3-force";
  
  const RADIUS = 5;
  let nodes = [];
  let simulation = forceSimulation();

  $: simulation.nodes(data)
    .force("x", forceX().x(d => xScale(d.Year)).strength(1))
    .force("y", forceY().y(d => innerHeight / 2).strength(0.1))
    .force("collide", forceCollide().radius((d) => radiusScale(d.Deaths) + 2))
    .alpha(0.6)
    .alphaDecay(0.04)
    .on("tick", () => {
      nodes = simulation.nodes();
    })
    .restart();

  let width = 400,
    height = 550;
  
  const margin = {
    top: 25,
    right: window.innerWidth < 568 ? 65 : 100,
    bottom: 25,
    left: window.innerWidth < 568 ? 15 : 100,
  }

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: xScale = scaleLinear()
    .domain([1940, 2024])
    .range([0, innerWidth]);

  $: continents = rollups(
      data,
      v => sum(v, d => d.Deaths),
      d => d.Type
    ) // Agrupar los datos por tipo de democracia y devolver el promedio de edad
      .sort((a, b) => b[1] - a[1]) // Ordenar los datos por valor promedio
      .map(d => d[0]); // Obtener el nombre del continente

      const colorRange = { 
    "Temperatura extrema": "#e6ba2e",
    "Inundación": "#3983bf",
    "Tormenta": "#696969",
    "Incendio": "#B22222",
    "Terremoto": "#8B4513",
};
  
  $: colorScale = scaleOrdinal()
      .domain(continents)
      .range(continents.map(type => colorRange[type] || colorRange["Other"]));

  $: yScale = scaleBand()
      .domain(continents)
      .range([innerHeight, 0])
      .paddingOuter(0.5);
  
  $: radiusScale = scaleSqrt()
    .domain([1, 15090])
    .range(width < 568 ? [2 , 60] : [3, 100]);
  
  import AxisX from "$components/AxisX.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip from "$components/Tooltip.svelte";

  let hovered;

  import { fade } from "svelte/transition";

  let hoveredContinent;
</script>

<h1>Fallecidos según el tipo de catástrofe</h1>
<Legend {colorScale} {colorRange} bind:hoveredContinent />
<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<div class='chart-container'
  bind:clientWidth={width}
  on:click={() => {hovered = null}}>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})"
    on:mouseleave={() => {
      hovered = null;
    }}>
      <AxisX {xScale} height={innerHeight} />
      {#each nodes as node}
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <circle in:fade={{ delay: 400 }}
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.Deaths)}
          fill={colorScale(node.Type)}
          stroke={hovered || hoveredContinent
            ? hovered === node || hoveredContinent === node.Type
            ? "black"
            : "transparent"
            : "transparent"}
          opacity={hovered || hoveredContinent
            ? hovered === node || hoveredContinent === node.Type
            ? 1
            : 0.3
            : 1}
          on:mouseover={() => {
            hovered = node;
          }}
          on:focus={() => {
            hovered = node;
          }}
          tabindex="0"
          on:click={(event) => {
            event.stopPropagation();
          }}
        />
      {/each}
    </g>
  </svg>
    {#if hovered}
      <Tooltip data={hovered} {colorScale} {width}/>
    {/if}
</div>

<style>
  :global(.tick text, .axis-title) {
    font-size: 12px;
    font-weight: 400;
    fill: hsla(212, 10%, 53%, 1);
    user-select: none;
  }

  :global(.axos-title) {
    font-size: 14px;
  }

  h1 {
    font-size: 1.2em;
    margin-bottom: 3px;
    font-weight: 0;
    text-align: center;
  }

  circle {
    transition: stroke 300ms ease, opacity 300ms ease;
  }
</style>