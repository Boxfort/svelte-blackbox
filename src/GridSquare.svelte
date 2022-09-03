<script>
    import { createEventDispatcher } from "svelte";

    export let position;

    export let data = {
        hasBall: false,
        isMarkedAsBall: false,
        isMarkedAsClear: false,
    };

    const dispatch = createEventDispatcher();

    function handleClick() {
        if (data.isMarkedAsClear) {
            data.isMarkedAsBall = true;
            data.isMarkedAsClear = false;
        } else if (data.isMarkedAsBall) {
            data.isMarkedAsBall = false;
            data.isMarkedAsClear = false;
        } else {
            data.isMarkedAsClear = true;
        }
        console.log("PRESSED: " + position.toString());
        dispatch("clicked", { pos: position });
    }
</script>

<div
    class="grid-square"
    class:marked-clear={data.isMarkedAsClear}
    on:click={handleClick}
>
    {#if data.isMarkedAsBall}
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
