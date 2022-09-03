<script>
    import Board from "./Board.svelte";

    let data = {
        width: 5,
        height: 5,
        balls: 1,
    };
</script>

<main>
    <h1>Black Box</h1>
    <p>Width</p>
    <input bind:value={data.width} type="number" />
    <p>Height</p>
    <input bind:value={data.height} type="number" />
    <p>Balls</p>
    <input bind:value={data.balls} type="number" />

    <Board {...data} />
</main>

<style>
    main {
        text-align: center;
        padding: 1em;
        max-width: 240px;
        margin: 0 auto;
    }

    h1 {
        color: #ff3e00;
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    }

    @media (min-width: 640px) {
        main {
            max-width: none;
        }
    }
</style>
