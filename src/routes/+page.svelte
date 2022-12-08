<script lang="ts">
	import { onMount } from "svelte";

	interface Tree<T> {
		value: T;
		children: Tree<T>[];
	}

	function createTree<T>(value: T, children: Tree<T>[] = []) {
		return {
			value,
			children,
		};
	}

	let canvas: HTMLCanvasElement;

	let tree: Tree<string> = createTree("root");

	let treeNodeSize = 50;
	let xGap = 10;
	let yGap = 20;

	function getTreeWidth(tree: Tree<any>): number {
		if (tree.children.length == 0) return treeNodeSize;

		return tree.children
			.map((child) => getTreeWidth(child))
			.reduce((pv, cv) => pv + cv + xGap, 0);
	}

	function drawTree(
		ctx: CanvasRenderingContext2D,
		tree: Tree<any>,
		x: number,
		y: number
	) {
		let width = getTreeWidth(tree);
		let startingPoint = x - width / 2;

		tree.children.forEach((child) => {
			let width = getTreeWidth(child);

			drawTree(
				ctx,
				child,
				startingPoint + width / 2 + xGap / 2,
				y + treeNodeSize + yGap
			);

			ctx.strokeStyle = "#ffffff";
			ctx.lineWidth = 4;
			ctx.beginPath();
			ctx.moveTo(x + treeNodeSize / 2, y + treeNodeSize);
			ctx.lineTo(
				startingPoint + width / 2 + xGap / 2 + treeNodeSize / 2,
				y + treeNodeSize + yGap
			);
			ctx.stroke();

			startingPoint += width + xGap;
		});

		drawTreeNode(ctx, tree.value, x, y);
	}

	function drawTreeNode(
		ctx: CanvasRenderingContext2D,
		text: string,
		x: number,
		y: number
	) {
		ctx.fillStyle = "#444444";
		ctx.fillRect(x, y, treeNodeSize, treeNodeSize);

		ctx.fillStyle = "#ffffff";
		ctx.font = "24px Arial";
		ctx.fillText(text, x, y + treeNodeSize / 2);
	}

	function drawEverything() {
		let ctx = canvas.getContext("2d");

		if (!ctx) return;

		ctx.fillStyle = "#000000";
		ctx.fillRect(0, 0, 1000, 1000);

		drawTree(ctx, tree, 500, 100);
	}

	async function copyTreeToClipboard() {
		try {
			let str = JSON.stringify(tree);
			await navigator.clipboard.writeText(str);
			alert("Tree copied to clipboard");
		} catch {
			alert("Invalid JSON");
		}
	}

	async function pasteTreeFromClipboard() {
		let string = await navigator.clipboard.readText();

		try {
			tree = JSON.parse(string);
			drawEverything();
		} catch {
			alert("Invalid JSON provided");
		}
	}

	onMount(drawEverything);
</script>

<div>
	<button on:click={copyTreeToClipboard}>Copy tree to clipboard (JSON)</button>
	<button on:click={pasteTreeFromClipboard}
		>Paste tree from clipboard (JSON)</button
	>

	<canvas bind:this={canvas} width="1000" height="1000" />
</div>

<style>
	:global(body),
	:global(html) {
		margin: 0;
	}
	div {
		display: flex;
		flex-direction: column;

		width: 99vmin;
		height: 99vmin;
	}
	canvas {
		width: 100%;
		height: 100%;
	}
</style>
