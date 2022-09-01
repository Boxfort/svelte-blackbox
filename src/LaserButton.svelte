<script>
    export let position;
    export let domElement;

    const dispatch = createEventDispatcher();

    function handleClick() {
        dispatch("clicked", { pos: position });
    }
</script>

<div class="laser-button" bind:this={domElement} on:click={handleClick} />

<style>
    .laser-button {
        width: 3em;
        height: 3em;
        border: 1px solid black;
        background-color: grey;
        cursor: pointer;
    }
</style>
