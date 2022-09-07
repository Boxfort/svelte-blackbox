<script>
    import { createEventDispatcher } from "svelte";

    export let position;

    export let data = {
        text: "",
        highlighted: false,
        otherHighlight: null,
    };

    const dispatch = createEventDispatcher();

    function onClickDown(event) {
        data.highlighted = true;
        dispatch("click-down", { pos: position });
    }

    function onClickUp() {
        data.highlighted = false;
        if (data.otherHighlight != null) {
            data.otherHighlight.highlighted = false;
            data.otherHighlight = null;
        }
        dispatch("click-up", { pos: position });
    }
</script>

<div
    class="laser-button"
    on:mousedown={onClickDown}
    on:mouseup={onClickUp}
    on:mouseleave={onClickUp}
    bind:this={data.domElement}
    class:active={data != null ? data.highlighted : false}
>
    {#if data != null}
        {data.text}
    {/if}
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

    .active {
        background-color: white;
        box-shadow: 0px 0px 5px 3px #f00;
        z-index: 1;
    }
</style>
