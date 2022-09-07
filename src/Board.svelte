<script>
    import { afterUpdate, tick } from "svelte";

    import GridSquare from "./GridSquare.svelte";
    import LaserButton from "./LaserButton.svelte";

    export let width;
    export let height;
    export let balls;

    let gridSquares = {};
    let laserButtons = {};
    let laserPaths = {};

    let checking = false;
    let dirtyBoard = false;
    let buttonCounter = 1;

    let board;

    $: OnDataChanged(width, height, balls);

    async function OnDataChanged(...args) {
        await tick();

        // Reset the board
        for (let key of Object.keys(gridSquares)) {
            gridSquares[key].isMarkedAsBall = false;
            gridSquares[key].isMarkedAsClear = false;
            gridSquares[key].hasBall = false;
            removeCheckMarks();
        }

        // Reset Top and bottom buttons
        for (let i = 0; i < width; i++) {
            if (laserButtons[[i, 0]] == null) laserButtons[[i, 0]] = {};

            if (laserButtons[[i, height + 1]] == null)
                laserButtons[[i, height + 1]] = {};

            laserButtons[[i, 0]].text = "";
            laserButtons[[i, height + 1]].text = "";

            laserPaths[[i, 0]] = [];
            laserPaths[[i, height + 1]] = [];
        }
        // Reset Middle Buttons
        for (let i = 1; i <= height; i++) {
            if (laserButtons[[0, i]] == null) laserButtons[[0, i]] = {};
            if (laserButtons[[width, i]] == null) laserButtons[[width, i]] = {};

            laserButtons[[0, i]].text = "";
            laserButtons[[width, i]].text = "";

            laserPaths[[0, i]] = [];
            laserPaths[[width, i]] = [];
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
                placedBalls++;
            }
        }
    }

    function onCheckButtonPressed() {
        if (checking) return;

        checkBoard();
    }

    export function checkBoard() {
        checking = true;

        let success = checkResults();
        for (let key of Object.keys(gridSquares)) {
            if (gridSquares[key].isMarkedAsBall && gridSquares[key].hasBall) {
                gridSquares[key].isMarkedAsBallCorrectly = true;
            } else if (
                gridSquares[key].isMarkedAsBall &&
                !gridSquares[key].hasBall
            ) {
                gridSquares[key].isMarkedAsBallIncorrectly = true;
            } else if (
                !gridSquares[key].isMarkedAsBall &&
                gridSquares[key].hasBall
            ) {
                gridSquares[key].isMissingBall = true;
            }
        }

        dirtyBoard = true;
        checking = false;

        return success;
    }

    function onLaserButtonDown(event) {
        if (dirtyBoard) removeCheckMarks();

        shootLaser(event.detail.pos);
    }

    function onGridSquareClicked(event) {
        if (dirtyBoard) removeCheckMarks();
    }

    function removeCheckMarks() {
        for (let key of Object.keys(gridSquares)) {
            gridSquares[key].isMarkedAsBallCorrectly = false;
            gridSquares[key].isMarkedAsBallIncorrectly = false;
            gridSquares[key].isMissingBall = false;
        }

        dirtyBoard = false;
    }

    function shootLaser(buttonPosition) {
        let direction = [0, 0];
        let startingPosition = [0, 0];
        startingPosition[0] = buttonPosition[0];
        startingPosition[1] = buttonPosition[1];

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

        let shouldCalculate = false;
        if (laserPaths[buttonPosition].length == 0) {
            let startPos = [
                startingPosition[0] - direction[0],
                startingPosition[1] - direction[1],
            ];
            console.log(startPos);
            laserPaths[buttonPosition].push({ pos: startPos });
            shouldCalculate = true;
        }

        let result = resolveLaser(
            startingPosition,
            direction,
            true,
            shouldCalculate ? buttonPosition : null
        );

        if (result.result == "Hit") {
            laserButtons[buttonPosition].text = "H";
        } else if (result.result == "Reflected") {
            laserButtons[buttonPosition].text = "R";
        } else if (result.result == "Exited") {
            let targetButtonPos = [0, 0];
            if (arePositionsEqual(result.dir, [0, 1])) {
                // The laser exited the bottom
                targetButtonPos = addPositions(result.pos, [0, 2]);
            } else if (arePositionsEqual(result.dir, [0, -1])) {
                // The laser exited the top
                targetButtonPos = result.pos;
            } else if (arePositionsEqual(result.dir, [1, 0])) {
                // The laser exited the right
                targetButtonPos = addPositions(result.pos, [1, 1]);
            } else if (arePositionsEqual(result.dir, [-1, 0])) {
                // The laser exited the left
                targetButtonPos = addPositions(result.pos, [0, 1]);
            }

            if (laserPaths[targetButtonPos].length > 0) {
                console.log("clear target");
                laserPaths[targetButtonPos] = [];
            }

            if (laserButtons[buttonPosition].text == "") {
                laserButtons[buttonPosition].text = buttonCounter.toString();
                laserButtons[targetButtonPos].text = buttonCounter.toString();

                buttonCounter++;
            }

            if (targetButtonPos != null) {
                laserButtons[targetButtonPos].highlighted = true;
                laserButtons[buttonPosition].otherHighlight =
                    laserButtons[targetButtonPos];
            }
        }
    }

    function resolveLaser(
        position,
        direction,
        justFired = false,
        buttonPosition = null
    ) {
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

        // Store this step of the path so we can draw it later
        if (buttonPosition != null) {
            laserPaths[buttonPosition] = laserPaths[buttonPosition].concat({
                pos: position,
            });
        }

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
            if (buttonPosition != null) {
                laserPaths[buttonPosition] = laserPaths[buttonPosition].concat({
                    pos: addPositions(position, direction),
                });
            }
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
            return resolveLaser(
                nextPosition,
                nextDirection,
                false,
                buttonPosition
            );
        } else {
            // We've left the board
            if (buttonPosition != null) {
                laserPaths[buttonPosition] = laserPaths[buttonPosition].concat({
                    pos: nextPosition,
                });
            }
            return {
                result: "Exited",
                pos: position,
                dir: nextDirection,
            };
        }
    }

    function checkResults() {
        let expectedResults = {};
        let actualResults = {};
        let actualBallLocations = [];

        expectedResults = getBoardResults();

        // Modify the board to be how it's set up
        for (let key of Object.keys(gridSquares)) {
            if (gridSquares[key].hasBall) {
                actualBallLocations.push(key);
                gridSquares[key].hasBall = false;
            }

            if (gridSquares[key].isMarkedAsBall) {
                gridSquares[key].hasBall = true;
            }
        }

        actualResults = getBoardResults();

        let success = doResultsMatch(expectedResults, actualResults);

        // Reset the board
        for (let key of Object.keys(gridSquares)) {
            gridSquares[key].hasBall = false;
        }

        // Replace the balls
        actualBallLocations.forEach((position) => {
            gridSquares[position].hasBall = true;
        });

        return success;
    }

    function doResultsMatch(expectedResults, actualResults) {
        for (let key of Object.keys(expectedResults)) {
            if (expectedResults[key].result == actualResults[key].result) {
                if (expectedResults[key].result == "Exited") {
                    if (
                        !arePositionsEqual(
                            expectedResults[key].pos,
                            actualResults[key].pos
                        ) ||
                        !arePositionsEqual(
                            expectedResults[key].dir,
                            actualResults[key].dir
                        )
                    ) {
                        console.log(
                            "Exit location for " +
                                key +
                                " do not match: " +
                                expectedResults[key].pos +
                                " != " +
                                actualResults[key].pos
                        );
                        return false;
                    }
                }
            } else {
                console.log(
                    "Results for " +
                        key +
                        " do not match: " +
                        expectedResults[key].result +
                        " != " +
                        actualResults[key].result
                );
                return false;
            }
        }

        return true;
    }

    function getBoardResults() {
        let results = {};
        for (let i = 0; i < width; i++) {
            // Top Row
            results[(i, 0)] = resolveLaser([i, 0], [0, 1], true);
            // Bottom Row
            results[(i, height - 1)] = resolveLaser(
                [i, height - 1],
                [0, -1],
                true
            );
        }

        for (let i = 1; i < height - 1; i++) {
            // Left Side
            results[(0, i)] = resolveLaser([0, i], [1, 0], true);

            // Right Side
            results[(width - 1, i)] = resolveLaser(
                [width - 1, i],
                [-1, 0],
                true
            );
        }

        return results;
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

    function getPointsFromLaserPath(laserPath) {
        let points = [];

        laserPath.forEach((element) => {
            let firstGridSquare = gridSquares[[0, 0]];
            let gridSquareWidth = firstGridSquare.domElement.clientWidth;
            let gridSquareHeight = firstGridSquare.domElement.clientHeight;
            let padding = 1; // We need to offset with regards to the padding around each grid square
            let borderSize = 1; // we also need to offset with regards to the border around each grid square

            points.push(
                (
                    element.pos[0] * gridSquareWidth +
                    gridSquareWidth * 1.5 +
                    padding * (element.pos[0] + 2) +
                    borderSize * ((element.pos[0] + 1) * 2)
                ).toString() +
                    "," +
                    (
                        element.pos[1] * gridSquareHeight +
                        gridSquareHeight * 1.5 +
                        padding * (element.pos[1] + 2) +
                        borderSize * ((element.pos[1] + 1) * 2)
                    ).toString()
            );
        });

        return points.join(",");
    }
</script>

<div
    class="grid-container"
    style="--width: {width}; --height: {height}"
    bind:this={board}
>
    <!-- Top buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton
            position={[i, 0]}
            bind:data={laserButtons[[i, 0]]}
            on:click-down={onLaserButtonDown}
        />
    {/each}
    <div class="grid-item-empty" />

    <!-- Main grid -->
    {#each Array(height) as _, i (i)}
        <LaserButton
            position={[0, i + 1]}
            bind:data={laserButtons[[0, i + 1]]}
            on:click-down={onLaserButtonDown}
        />

        {#each Array(width) as _, j (j)}
            <GridSquare
                position={[j, i]}
                bind:data={gridSquares[[j, i]]}
                on:clicked={onGridSquareClicked}
            />
        {/each}

        <LaserButton
            position={[width, i + 1]}
            bind:data={laserButtons[[width, i + 1]]}
            on:click-down={onLaserButtonDown}
        />
    {/each}

    <!-- Bottom buttons -->
    <div class="grid-item-empty" />
    {#each Array(width) as _, i (i)}
        <LaserButton
            position={[i, height + 1]}
            bind:data={laserButtons[[i, height + 1]]}
            on:click-down={onLaserButtonDown}
        />
    {/each}
    <div class="grid-item-empty" />
    <svg class="laser-svg">
        {#if dirtyBoard}
            {#each Object.values(laserPaths) as laserPath}
                <polyline
                    class="laser-line"
                    points={getPointsFromLaserPath(laserPath)}
                />
            {/each}
        {/if}
    </svg>
</div>

<button on:click={onCheckButtonPressed} disabled={checking}>Check</button>

<style>
    .grid-container {
        position: relative;
        display: grid;
        grid-template-columns: repeat(calc(var(--width) + 2), auto);
        grid-template-rows: repeat(calc(var(--height) + 2), auto);
        width: fit-content;
        margin: auto;
        gap: 1px;
    }

    .laser-svg {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }

    .laser-line {
        fill: none;
        stroke-width: 3;
        stroke-linecap: round;
        stroke: rgba(255, 0, 0, 1);
        stroke-dasharray: 100;
        animation: dash 2s infinite;
        animation-timing-function: linear;
    }

    @keyframes dash {
        to {
            stroke-dashoffset: -1000;
        }
    }
</style>
