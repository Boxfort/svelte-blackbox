<script>
    import { createEventDispatcher } from "svelte";

    export let domElement;
    export let position;

    let hasBall = false;
    let isMarkedAsBall = false;
    let isMarkedAsClear = false;

    const dispatch = createEventDispatcher();

    function handleClick() {
        if (isMarkedAsClear) {
            isMarkedAsBall = true;
            isMarkedAsClear = false;
        } else if (isMarkedAsBall) {
            isMarkedAsBall = false;
            isMarkedAsClear = false;
        } else {
            isMarkedAsClear = true;
        }
        dispatch("clicked", { pos: position });
    }
</script>

<div
    class="grid-square"
    class:marked-clear={isMarkedAsClear}
    bind:this={domElement}
    on:click={handleClick}
>
    {#if isMarkedAsBall}
        <div class="grid-ball" />
    {/if}
</div>

<style>
    .grid-square {
        display: flex;
        align-items: center;
        justify-content: center;

        width: 3em;
        height: 3em;
        border: 1px solid black;

        cursor: pointer;
        user-select: none;
    }

    .grid-square:active {
        background-color: lightgrey;
    }

    .marked-clear {
        background-color: grey;
    }

    .grid-ball {
        background-color: black;
        height: 90%;
        width: 90%;
        border-radius: 50%;
        margin: auto;
    }
</style>
