<script>
    import { tick } from "svelte";

    import GridSquare from "./GridSquare.svelte";
    import LaserButton from "./LaserButton.svelte";

    export let width;
    export let height;
    export let balls;

    let gridSquares = {};
    let laserButtons = {};

    $: OnDataChanged(width, height, balls);

    async function OnDataChanged(...args) {
        // Reset the board
        for (let key of Object.keys(gridSquares)) {
            gridSquares[key].isMarkedAsBall = false;
            gridSquares[key].isMarkedAsClear = false;
            gridSquares[key].hasBall = false;
        }
        // Reset the buttons
        for (let i = 0; i <= width; i++) {
            laserButtons[[i, 0]] = "";
            laserButtons[[i, height + 1]] = "";
        }
        for (let i = 0; i <= height + 1; i++) {
            laserButtons[[0, i]] = "";
            laserButtons[[width, i]] = "";
        }
        buttonCounter = 1;

        await tick();

        // Distribute some balls
        let placedBalls = 0;
        while (placedBalls < balls) {
            let randX = Math.floor(Math.random() * width);
            let randY = Math.floor(Math.random() * height);

            // If this square doesn't already have a ball
            if (!gridSquares[[randX, randY]].hasBall) {
                // Place a ball
                gridSquares[[randX, randY]].hasBall = true;
                gridSquares[[randX, randY]].isMarkedAsBall = true;
                placedBalls++;

                console.log("BALL AT: " + [randX, randY]);
            }
        }
    }

    let buttonCounter = 1;

    function onButtonClicked(event) {
        if (laserButtons[event.detail.pos] == "") {
            // New button pressed
            laserButtons[event.detail.pos] = buttonCounter.toString();
            buttonCounter++;
        }

        shootLaser(event.detail.pos);
    }

    function shootLaser(startingPosition) {
        let direction = [0, 0];

        // Which way should we shoot the laser?
        if (startingPosition[1] == 0) {
            // If we're on the top row
            direction = [0, 1];
        } else if (startingPosition[1] == height + 1) {
            // If we're on the bottom
            direction = [0, -1];
            startingPosition[1] -= 2;
        } else if (startingPosition[0] == 0) {
            // If we're on the left side
            direction = [1, 0];
            startingPosition[1]--;
        } else if (startingPosition[0] == width) {
            // If we're on the right side
            direction = [-1, 0];
            startingPosition[0]--;
            startingPosition[1]--;
        }

        let result = resolveLaser(startingPosition, direction, true);
        console.log(result);
    }

    function resolveLaser(position, direction, justFired = false) {
        let relativeLeft;
        let relativeRight;

        if (arePositionsEqual(direction, [0, 1])) {
            relativeLeft = [1, 0];
            relativeRight = [-1, 0];
        } else if (arePositionsEqual(direction, [0, -1])) {
            relativeLeft = [-1, 0];
            relativeRight = [1, 0];
        } else if (arePositionsEqual(direction, [1, 0])) {
            relativeLeft = [0, -1];
            relativeRight = [0, 1];
        } else if (arePositionsEqual(direction, [-1, 0])) {
            relativeLeft = [0, 1];
            relativeRight = [0, -1];
        }

        console.log("Position: " + position);
        console.log("Direction: " + direction);
        console.log("JustFired: " + justFired);

        let frontPos = addPositions(position, direction);
        let leftPos = addPositions(position, relativeLeft);
        let rightPos = addPositions(position, relativeRight);
        let frontLeftPos = addPositions(
            addPositions(position, direction),
            relativeLeft
        );
        let frontRightPos = addPositions(
            addPositions(position, direction),
            relativeRight
        );

        gridSquares[position].isMarkedAsClear = true;

        // Special case for just entering the board.
        if (justFired) {
            if (gridSquares[position].hasBall) {
                return {
                    result: "Hit",
                };
            }
            if (
                (isInBounds(leftPos) && gridSquares[leftPos].hasBall) ||
                (isInBounds(rightPos) && gridSquares[rightPos].hasBall)
            ) {
                return {
                    result: "Reflected",
                };
            }
        }

        if (isInBounds(frontPos)) {
            if (gridSquares[frontPos].hasBall) {
                return {
                    result: "Hit",
                };
            }
        } else {
            // We've left the board
            return {
                result: "Exited",
                pos: position,
                dir: direction,
            };
        }

        let nextPosition;
        let nextDirection;

        if (isInBounds(frontLeftPos)) {
            if (isInBounds(frontRightPos)) {
                if (
                    gridSquares[frontLeftPos].hasBall &&
                    gridSquares[frontRightPos].hasBall
                ) {
                    return {
                        result: "Reflected",
                    };
                }
            }

            if (gridSquares[frontLeftPos].hasBall) {
                nextPosition = addPositions(position, relativeRight);
                nextDirection = relativeRight;
            }
        }

        if (isInBounds(frontRightPos)) {
            if (gridSquares[frontRightPos].hasBall) {
                nextPosition = addPositions(position, relativeLeft);
                nextDirection = relativeLeft;
            }
        }

        if (nextPosition == null) {
            // Let's keep going!
            nextPosition = addPositions(position, direction);
            nextDirection = direction;
        }

        if (isInBounds(nextPosition)) {
            return resolveLaser(nextPosition, nextDirection);
        } else {
            // We've left the board
            return {
                result: "Exited",
                pos: position,
                dir: direction,
            };
        }
    }

    function arePositionsEqual(a, b) {
        return a[0] == b[0] && a[1] == b[1];
    }

    function addPositions(a, b) {
        let c = [0, 0];
        c[0] = a[0] + b[0];
        c[1] = a[1] + b[1];

        return c;
    }

    function isInBounds(position) {
        return !(
            position[0] >= width ||
            position[0] < 0 ||
            position[1] >= height ||
            position[1] < 0
        );
    }
</script>

<div class="grid-container" style="--width: {width}; --height: {height}">
    <!-- Top buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton
            position={[i, 0]}
            bind:text={laserButtons[[i, 0]]}
            on:clicked={onButtonClicked}
        />
    {/each}
    <div class="grid-item-empty" />

    <!-- Main grid -->
    {#each Array(height) as _, i (i)}
        <LaserButton
            position={[0, i + 1]}
            bind:text={laserButtons[[0, i + 1]]}
            on:clicked={onButtonClicked}
        />

        {#each Array(width) as _, j (j)}
            <GridSquare position={[j, i]} bind:data={gridSquares[[j, i]]} />
        {/each}

        <LaserButton
            position={[width, i + 1]}
            bind:text={laserButtons[[width, i + 1]]}
            on:clicked={onButtonClicked}
        />
    {/each}

    <!-- Bottom buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton
            position={[i, height + 1]}
            bind:text={laserButtons[[i, height + 1]]}
            on:clicked={onButtonClicked}
        />
    {/each}
    <div class="grid-item-empty" />
</div>

<style>
    .grid-container {
        display: grid;
        grid-template-columns: repeat(calc(var(--width) + 2), auto);
        grid-template-rows: repeat(calc(var(--height) + 2), auto);
        width: fit-content;
        margin: auto;
        gap: 1px;
    }
</style>
