<script>
    import { onMount } from 'svelte';
    import { writable, get } from 'svelte/store';

    let size = 81; // Adjust grid size as needed
    const canvasSize = 800; // Adjust canvas size as needed

    let canvas;
    let ctx;
    let running = false;
    let intervalId;

    let simulationSpeed = 200; // Initial speed (milliseconds)

    const createGrid = () => Array(size).fill(null).map(() => Array(size).fill(0));
    const cells = writable(createGrid());

    // Initialize with a simple pattern (optional)
    $cells[Math.floor(size / 2)][Math.floor(size / 2)] = 1;
    $cells[Math.floor(size / 2) - 1][Math.floor(size / 2)] = 1;
    $cells[Math.floor(size / 2) + 1][Math.floor(size / 2) + 1] = 1;
    $cells[Math.floor(size / 2)][Math.floor(size / 2) - 1] = 1;
    $cells[Math.floor(size / 2)][Math.floor(size / 2) + 1] = 1;

    function clickHandler(event) {
        if (!ctx) return; // Check if ctx is available
        const rect = canvas.getBoundingClientRect(); // Get canvas position
        const x = Math.floor((event.clientX - rect.left) / (canvasSize / size));
        const y = Math.floor((event.clientY - rect.top) / (canvasSize / size));

        if (x >= 0 && x < size && y >= 0 && y < size) { // Check boundaries
            // $cells = $cells.map((row, i) =>
            //     row.map((cell, j) => (i === y && j === x ? (cell ? 0 : 1) : cell))
            // );
            console.log($cells[y][x]);
            $cells[y][x] = $cells[y][x] ? 0 : 1;
            drawGrid();
        }
    }

    onMount(() => {
        canvas = document.querySelector('canvas');
        if (canvas) {
            canvas.addEventListener('click', clickHandler);
            ctx = canvas.getContext('2d');
            drawGrid();
        }
    });

    function drawCell(x, y, status) {
        if (!ctx) return;
        const cellSize = canvasSize / size;
        ctx.fillStyle = status ? 'black' : 'white';
        ctx.fillRect(y * cellSize, x * cellSize, cellSize, cellSize); // Corrected x and y
    }

    function drawGrid() {
        if (!ctx) return;
        ctx.clearRect(0, 0, canvasSize, canvasSize);
        for (let i = 0; i < size; i++) {
            for (let j = 0; j < size; j++) {
                drawCell(i, j, $cells[i][j]);
            }
        }
    }

    function getAliveNeighbors(row, col) {
        let count = 0;
        for (let i = -1; i <= 1; i++) {
            for (let j = -1; j <= 1; j++) {
                if (i === 0 && j === 0) continue; // Skip self
                const r = row + i;
                const c = col + j;
                if (r >= 0 && r < size && c >= 0 && c < size && $cells[r][c]) {
                    count++;
                }
            }
        }
        return count;
    }

    function conwayStep() {
        const nextCells = createGrid(); // Create next grid

        for (let i = 0; i < size; i++) {
            for (let j = 0; j < size; j++) {
                const aliveNeighbors = getAliveNeighbors(i, j);
                const isAlive = $cells[i][j];

                if (isAlive) {
                    nextCells[i][j] = (aliveNeighbors === 2 || aliveNeighbors === 3) ? 1 : 0;
                } else {
                    nextCells[i][j] = (aliveNeighbors === 3) ? 1 : 0;
                }
            }
        }

        $cells = nextCells; // Update cells with the new generation
        drawGrid();
    }

function toggleSimulation() {
        running = !running;
        if (running) {
            clearInterval(intervalId); // Clear any existing interval
            intervalId = setInterval(conwayStep, simulationSpeed);
        } else {
            clearInterval(intervalId);
        }
    }

    function adjustSpeed(event) {
        simulationSpeed = event.target.value;
        if (running) {
            clearInterval(intervalId); // Restart with new speed
            intervalId = setInterval(conwayStep, simulationSpeed);
        }
    }

</script>

<div class="flex flex-col h-screen items-center justify-center">  <canvas bind:this={canvas} class="border-1 border-black" width={canvasSize} height={canvasSize}></canvas>
    <div class="mt-4">  <button on:click={toggleSimulation}>{running ? 'Stop' : 'Start'}</button>

        <div class="mt-2">  <label for="speedSlider">Speed:</label>
            <input type="range" id="speedSlider" min="10" max="1000" step="10" bind:value={simulationSpeed} on:input={adjustSpeed}>
            <span>{simulationSpeed} ms</span>
        </div>
    </div>
</div>
