<script>
    import { createEventDispatcher } from "svelte";

    export let position;

    export let data = {
        text: "pain",
        highlighted: false,
    };

    const dispatch = createEventDispatcher();

    function onClickDown() {
        dispatch("click-down", { pos: position });
    }

    function onClickUp() {
        dispatch("click-up", { pos: position });
    }
</script>

<div
    class="laser-button"
    on:mousedown={onClickDown}
    on:mouseup={onClickUp}
    class:active={data.highlighted}
>
    {data.text}
</div>

<style>
    .laser-button {
        display: flex;
        align-items: center;
        justify-content: center;

        width: 3em;
        height: 3em;

        border: 1px solid darkgrey;
        background-color: lightgrey;

        cursor: pointer;
        user-select: none;
    }

    .laser-button:hover {
        background-color: white;
    }

    .laser-button:active {
        background-color: white;
        box-shadow: 0px 0px 5px 3px #f00;
        z-index: 1;
    }
</style>
