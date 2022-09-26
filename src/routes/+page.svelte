<script lang="ts">
	import { onMount } from "svelte";

	let canvas: HTMLCanvasElement;

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

	onMount(() => {
		let ctx = canvas.getContext("2d");

		if (!ctx) return;

		ctx.fillStyle = "#000000";
		ctx.fillRect(0, 0, 1000, 1000);

		drawTree(
			ctx,
			createTree("root", [
				createTree("a", [createTree("e"), createTree("f")]),
				createTree("b"),
				createTree("c"),
				createTree("d"),
			]),
			500,
			100
		);

		drawTree(
			ctx,
			createTree("root", [
				createTree("a", [createTree("d"), createTree("e")]),
				createTree("b", []),
				createTree("c", [
					createTree("f", [createTree("i"), createTree("j")]),
					createTree("g"),
					createTree("h"),
				]),
			]),
			500,
			500
		);
	});
</script>

<canvas bind:this={canvas} width="1000" height="1000" />

<style>
	:global(body) {
		margin: 0;
	}
	:global(html, body, body > div) {
		height: 100%;
	}
	canvas {
		width: 100vmin;
		height: 100vmin;
	}
</style>
