<script>
    import { createEventDispatcher } from "svelte";

    export let position;
    export let text = "";

    const dispatch = createEventDispatcher();

    function handleClick() {
        dispatch("clicked", { pos: position });
    }
</script>

<div class="laser-button" on:click={handleClick}>
    {text}
</div>

<style>
    .laser-button {
        display: flex;
        align-items: center;
        justify-content: center;

        width: 3em;
        height: 3em;

        border: 1px solid black;
        background-color: darkgrey;

        cursor: pointer;
        user-select: none;
    }

    .laser-button:hover {
        background-color: lightgrey;
    }

    .laser-button:active {
        background-color: white;
    }
</style>
