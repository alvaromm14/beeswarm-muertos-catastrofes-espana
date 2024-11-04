<script>
    export let colorScale;
    export let hoveredContinent;
    export let colorRange;
</script>

<div class="legend" on:mouseleave={() => {hoveredContinent = null;}}>
    {#each Object.keys(colorRange) as continent (continent)}
        <!-- svelte-ignore a11y-mouse-events-have-key-events -->
        <p on:mouseover={() => {hoveredContinent = continent;}}
            class:unhovered={hoveredContinent &&
            hoveredContinent !== continent}>
            <span style="background-color: {colorScale(continent)}"></span>
            {continent}</p>
    {/each}
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        justify-content: center;
        flex-wrap: wrap;
        column-gap: 10px;
        row-gap: 5px;
        margin-bottom: 5px;
        user-select: none;
    }

    span {
        width: 10px;
        height: 10px;
        display: inline-block;
        border-radius: 50%;
        transform: translateY(-1px);
    }

    p {
        font-size: 0.9em;
        color: rgba(0, 0, 0, 0.8);
        display: flex;
        align-items: center;
        column-gap: 3px;
        transition: opacity 300ms ease;
    }

    .unhovered {
        opacity: 0.3;
    }
</style>