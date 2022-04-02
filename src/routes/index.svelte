<script lang="ts">
	import { onMount } from "svelte";

	let canvas: HTMLCanvasElement;

	class Tree<T> {
		constructor(public value: T, public children: Tree<T>[] = []) {}
	}

	let treeNodeSize = 50;
	let gap = 10;

	function getTreeWidth(tree: Tree<any>) {
		if (tree.children.length == 0) return treeNodeSize;

		return tree.children
			.map((child) => getTreeWidth(child))
			.reduce((pv, cv) => pv + cv + gap, 0);
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
				startingPoint + width / 2 + gap / 2,
				y + treeNodeSize + gap
			);

			startingPoint += width + gap;
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

		ctx.fillStyle = "#000000";
		ctx.fillRect(0, 0, 1000, 1000);

		drawTree(
			ctx,
			new Tree("root", [
				new Tree("a", [new Tree("e"), new Tree("f")]),
				new Tree("b"),
				new Tree("c"),
				new Tree("d")
			]),
			500,
			100
		);

		drawTree(
			ctx,
			new Tree("root", [
				new Tree("a", [new Tree("d"), new Tree("e")]),
				new Tree("b", []),
				new Tree("c", [
					new Tree("f", [new Tree("i"), new Tree("j")]),
					new Tree("g"),
					new Tree("h")
				])
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
