<script>
	import { onMount } from 'svelte';

	/** Size of the grid (n x n)
	 * @type {number}
	 */
	let size = 9;

	/** Size of the canvas
	 * @type {number}
	 */
	const canvasSize = 850;

	let canvas;
	let ctx;

	/** Create the initial grid, can also be called if a reset is required
	 * @returns {number[][]} The grid
	 */
	const createGrid = () =>
		Array(size)
			.fill(null)
			.map(() => Array(size).fill(0));

	/** Store the grid state in a reactive variable
	 * @type {number[][]}
	 */
	let cells = $state(createGrid());

	/// Set the first cell to be alive, the very center
	cells[Math.floor(size / 2)][Math.floor(size / 2)] = 1;

	/** Handle the click event on the canvas
	 * @param {MouseEvent} event - The click event
	 */
	function clickHandler(event) {
		const [clickX, clickY] = [event.offsetX, event.offsetY];
		const cellSize = 850 / size;
		const x = Math.floor(clickX / cellSize);
		const y = Math.floor(clickY / cellSize);
		cells[x][y] = cells[x][y] ? 0 : 1;
	}

	onMount(() => {
		canvas.addEventListener('click', clickHandler);
		ctx = canvas.getContext('2d');
	});

	/** Draw a cell on the canvas
	 * @param {number} x - The x coordinate (of the array)
	 * @param {number} y - The y coordinate (of the array)
	 * @param {number} status - The status of the cell (0 or 1)
	 */
	function drawCell(x, y, status) {
		const cellSize = 850 / size;
		ctx.fillStyle = status ? 'black' : 'white';
		ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
	}

	function update() {
		for (let i = 0; i < cells.length; i++) {
			for (let j = 0; j < cells[i].length; j++) {
				// For now, ignore the edges
				if (j != 0 && j != size - 1 && i != 0 && i != size - 1) {
					if (cells[i][j]) {
						cells[i][j] =
							!(cells[i - 1][j] && cells[i + 1][j]) || (cells[i][j - 1] && cells[i][j + 1]);
					}
				}
			}
		}
	}

	/**
	 * Run update every second
	 */
	const intervalID = setInterval(update, 1000);

	$effect(() => {
		ctx.clearRect(0, 0, canvasSize, canvasSize);
		let grid = '';
		for (let i = 0; i < cells.length; i++) {
			let str = '';
			for (let j = 0; j < cells[i].length; j++) {
				str += cells[i][j] ? ' 1' : ' 0';
				drawCell(i, j, cells[i][j]);
			}
			grid += str + '\n';
		}
		console.log(grid);
	});
</script>

<div class="flex h-screen items-center justify-center">
	<canvas bind:this={canvas} class="border-1 border-black" width={canvasSize} height={canvasSize}
	></canvas>
</div>
