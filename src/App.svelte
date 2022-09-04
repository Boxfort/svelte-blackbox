<script>
    import Header from "./Header.svelte";
    import Board from "./Board.svelte";

    let data = {
        width: 5,
        height: 5,
        balls: 1,
    };
</script>

<main>
    <Header />

    <div class="content">
        <Board {...data} />

        <p>Width</p>
        <input bind:value={data.width} type="number" />
        <p>Height</p>
        <input bind:value={data.height} type="number" />
        <p>Balls</p>
        <input bind:value={data.balls} type="number" />
    </div>
</main>

<style>
    main {
        text-align: center;
        margin: 0 auto;
    }

    h1 {
        color: #ff3e00;
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    }

    .content {
        padding: 10px;
    }
</style>
