<script>
    export let data;
    export let colorScale;
    export let width;

    import { fly, fade } from "svelte/transition";

    let tooltipWidth;

    let xPosition;
    let yPosition;
  document.addEventListener('mousemove', function(event) {
    xPosition = event.clientX + tooltipWidth + 10 > width
        ? event.clientX - tooltipWidth - 10
        : event.clientX + 10;
    yPosition = event.clientY - 100;
  });

    $: isWhiteText = ["Democracia plena", "Democracia imperfecta", "Régimen autoritario"].includes(data.Type);
</script>

<div
    class="tooltip"
    in:fly={{ y: 10, duration: 120, delay: 120 }}
    out:fade
    style="position: absolute;
        top: {yPosition}px;
        left: {xPosition}px;"
    bind:clientWidth={tooltipWidth}
>
    <h1>{data.Subtype}</h1><h2>{data.Location} ({data.Year})</h2>
    <div class="info">
        <span style="color: {colorScale(data.Type)};" class="score">{data.Deaths.toLocaleString('es-ES')}{data.Deaths > 1 ? " muertos" : " muerto"}</span>
    </div>
    <div>
        <span class="bar background" />
        <span class="bar foreground"
        style="background: {colorScale(data.Type)};
        width: {tooltipWidth}" />
    </div>
</div>

<style>
.tooltip {
    background-color: white;
    box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
    padding: 8px 6px;
    border-radius: 3px;
    pointer-events: none;
    transition: top 50ms ease, left 50ms ease;
    max-width: 200px;
    word-wrap: break-word;
    overflow-wrap: break-word;
    white-space: normal;
}

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
        align-items: center;
    }

    .score {
        font-size: 12px;
        font-weight: 600;
    }

    h1 {
        font-size: 1em;
        font-weight: 500;
        margin-bottom: 4px;
    }

    h2 {
        font-size: 14px;
        font-weight: 400;
        font-style: italic;
        margin-bottom: 6px;
    }

    .bar {
        position: absolute;
        bottom: 0;
        left: 0;
        height: 3px;
        width: 100%;
    }

    .bar.background {
        background: #eee;
    }
</style>