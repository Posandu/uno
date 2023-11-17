<script lang="ts">
	import { fly } from "svelte/transition";

	export let card: {
		type: string;
		color: string;
	};

	export let showCard = false;

	let numberMap: Record<string, string> = {
		one: "1",
		two: "2",
		three: "3",
		four: "4",
		five: "5",
		six: "6",
		seven: "7",
		eight: "8",
		nine: "9",
		zero: "0",
	};
</script>

<button class="card {showCard ? card.color : 'black'}" on:click>
	{#if showCard}
		<span>
			{#if card.type === "wild"}
				🥦
			{:else if card.type === "wild-draw-four"}
				+4
			{:else if card.type === "draw-two"}
				+2
			{:else if card.type === "skip"}
				🚫
			{:else if card.type === "reverse"}
				🔃
			{:else}
				{numberMap[card.type]}
			{/if}

			{#if card.color === "wild-draw-four" || card.color === "wild"}
				<span class="fancy">🥦</span>
			{/if}
		</span>
	{/if}
</button>

<style>
	.card {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 1rem;
		border-radius: 10px;
		font-weight: 600;
		outline: none;
		border: none;
		color: white;
		font-family: inherit;
		transition: all 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
		height: 150px;
		width: 80px;
		font-size: 50px;
		position: relative;
		overflow: hidden;
	}

	.card:hover {
		cursor: grab;
		box-shadow: 0 4px 10px #000000;
		transform: translateY(-5px);
	}

	.card:focus {
		box-shadow: 0 4px 10px #000000;
		transform: translateY(-5px);
	}

	.card:active {
		box-shadow: 0 2px 5px #000000;
		transform: translateY(-2px) scale(0.989);
	}

	.card.red {
		background-color: #d1180a;
	}

	.card.blue {
		background-color: rgb(13, 77, 216);
	}

	.card.green {
		background-color: #099e20;
	}

	.card.yellow {
		background-color: #dad603;
		color: #000;
	}

	.card.wild {
		background-color: #000;
		color: #fff;
	}

	/**Shimmer effect*/
	.card.wild::after {
		content: "";
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 200px;
		background: linear-gradient(
			to right,
			rgba(255, 255, 255, 0) 0%,
			rgba(204, 204, 204, 0.2) 20%,
			rgba(255, 255, 255, 0.5) 40%,
			rgba(255, 255, 255, 0) 100%
		);
		animation: shimmer 6s infinite cubic-bezier(0.23, 1, 0.32, 1);
	}

	@keyframes shimmer {
		0% {
			transform: translateX(-200%) rotate(40deg) scale(2);
		}
		100% {
			transform: translateX(200%) scale(2);
		}
	}

	.fancy {
		position: absolute;
		right: -4px;
		bottom: -19px;
		transform: scale(1.8) rotate(-80deg);
		background: transparent !important;
		opacity: 0.6;
		transition: all 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
	}

	.card:hover .fancy {
		transform: scale(1.8) rotate(-80deg) translateX(5px);
	}

	.card span {
		background: rgba(255, 255, 255, 0.26);
		border-radius: 100px;
		--size: 80px;
		min-width: var(--size);
		max-width: var(--size);
		min-height: var(--size);
		max-height: var(--size);
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 1px;
		z-index: 1;
	}

	.card.black {
		background-color: #ffffff6b;
		color: #fff;
		pointer-events: none;
	}
</style>
