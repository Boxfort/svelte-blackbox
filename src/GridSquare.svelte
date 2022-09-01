<script>
    import { createEventDispatcher } from "svelte";

    export let domElement;
    export let position;

    let hasBall = false;
    let isMarkedAsBall = false;
    let isMarkedAsClear = false;

    const dispatch = createEventDispatcher();

    function handleClick() {
        dispatch("clicked", { pos: position });
    }
</script>

<div class="grid-square" bind:this={domElement} on:click={handleClick} />

<style>
    .grid-square {
        width: 3em;
        height: 3em;
        border: 1px solid black;
        cursor: pointer;
    }
</style>
