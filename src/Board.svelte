<script>
    import { element } from "svelte/internal";

    import GridSquare from "./GridSquare.svelte";
    import LaserButton from "./LaserButton.svelte";

    export let width;
    export let height;
    export let balls;

    let gridSquares = {};
    let laserButtons = {};

    $: OnDataChanged(width, height, balls);

    function OnDataChanged(...args) {
        gridSquares.forEach((item) => {
            // blah
        });
    }
</script>

<div class="grid-container" style="--width: {width}; --height: {height}">
    <!-- Top buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton position={[i, 0]} bind:domElement={laserButtons[[i, 0]]} />
    {/each}
    <div class="grid-item-empty" />

    <!-- Main grid -->
    {#each Array(height) as _, i (i)}
        <LaserButton position={[0, i]} bind:domElement={laserButtons[[0, i]]} />

        {#each Array(width) as _, j (j)}
            <GridSquare
                position={[i, j]}
                bind:domElement={gridSquares[[i, j]]}
            />
        {/each}

        <LaserButton position={[1, i]} bind:domElement={laserButtons[[1, i]]} />
    {/each}

    <!-- Bottom buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton position={[i, 1]} bind:domElement={laserButtons[[i, 1]]} />
    {/each}
    <div class="grid-item-empty" />
</div>

<style>
    .grid-container {
        display: grid;
        grid-template-columns: repeat(calc(var(--width) + 2), auto);
        grid-template-rows: repeat(calc(var(--height) + 2), auto);
        width: fit-content;
        margin: auto;
        gap: 1px;
    }
</style>
