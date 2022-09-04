<script>
    import { createEventDispatcher } from "svelte";

    export let position;

    export let data = {
        hasBall: false,
        isMarkedAsBall: false,
        isMarkedAsClear: false,
        isMissingBall: false,
        isMarkedAsBallIncorrectly: false,
        isMarkedAsBallCorrectly: false,
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
        dispatch("clicked", { pos: position });
    }
</script>

<div
    class="grid-square"
    class:marked-clear={data.isMarkedAsClear}
    on:click={handleClick}
>
    {#if data.isMarkedAsBall || data.isMarkedAsBallIncorrectly || data.isMissingBall}
        <div
            class="grid-ball"
            class:missing-ball={data.isMissingBall}
            class:marked-incorrectly={data.isMarkedAsBallIncorrectly}
            class:marked-correctly={data.isMarkedAsBallCorrectly}
        />
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

    .missing-ball {
        background-color: white !important;
        border: 1px solid black !important;
    }

    .marked-incorrectly {
        background-color: red !important;
    }

    .marked-correctly {
        background-color: greenyellow !important;
    }

    .grid-ball {
        background-color: black;
        height: 90%;
        width: 90%;
        border-radius: 50%;
        margin: auto;
    }
</style>
