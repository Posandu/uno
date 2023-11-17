<script lang="ts">
	import { crossfade, fade, fly, scale } from "svelte/transition";
	import Card from "./Card.svelte";

	type UnoCard = {
		type: UnoType;
		color: "red" | "blue" | "green" | "yellow" | "wild";
	};

	type UnoType =
		| "one"
		| "two"
		| "three"
		| "four"
		| "five"
		| "six"
		| "seven"
		| "eight"
		| "nine"
		| "skip"
		| "reverse"
		| "draw-two"
		| "draw-four"
		| "wild"
		| "wild-draw-four";

	type UnoPlayer = {
		name: string;
		cards: UnoCard[];
	};

	type UnoGame = {
		players: UnoPlayer[];
		deck: UnoCard[];
		debtCards: UnoCard[];
		direction: "clockwise" | "counter-clockwise";
		turn: number;
		color: "red" | "blue" | "green" | "yellow";
	};

	let game: UnoGame = {
		players: [],
		debtCards: [],
		direction: "clockwise",
		turn: 0,
		color: "red",
		deck: [],
	};

	let chooseColorOpen = false;
	$: document.body.style.overflow = chooseColorOpen ? "hidden" : "auto";

	const [sendCard, receiveCard] = crossfade({
		duration: 400,
	});
	function shuffle(array: any[], seed: number) {
		let m = array.length;
		let t: any;
		let i: number;

		// While there remain elements to shuffle…
		while (m) {
			// Pick a remaining element…
			i = Math.floor(random(seed) * m--); // <-- MODIFIED LINE

			// And swap it with the current element.
			t = array[m];
			array[m] = array[i];
			array[i] = t;
			++seed; // <-- ADDED LINE
		}

		return array;
	}

	function random(seed: number) {
		const x = Math.sin(seed++) * 10000;
		return x - Math.floor(x);
	}

	const seed = 0.77844477;

	const generateDeck = (): UnoCard[] => {
		let deck: UnoCard[] = [];

		const colors: ("red" | "blue" | "green" | "yellow")[] = [
			"red",
			"blue",
			"green",
			"yellow",
		];
		const types: UnoType[] = [
			"one",
			"two",
			"three",
			"four",
			"five",
			"six",
			"seven",
			"eight",
			"nine",
			"skip",
			"reverse",
			"draw-two",
		];

		for (let color of colors) {
			for (let type of types) {
				deck.push({ type, color });
				deck.push({ type, color });
			}
		}

		for (let i = 0; i < 4; i++) {
			deck.push({ type: "wild", color: "wild" });
			deck.push({ type: "wild-draw-four", color: "wild" });
		}

		deck = shuffle(deck, seed);

		while (deck[0].type === "wild" || deck[0].type === "wild-draw-four") {
			deck = shuffle(deck, seed);
		}

		return deck;
	};

	const generatePlayers = (playerNames: string[]): UnoPlayer[] => {
		let players: UnoPlayer[] = [];

		for (let name of playerNames) {
			players.push({ name, cards: [] });
		}

		return players;
	};

	game.deck = generateDeck();
	game.color = game.deck[0].color as any;
	game.players = generatePlayers(["Bob", "Alice", "Posandu", "David"]);

	for (let i = 0; i < 7; i++) {
		for (let player of game.players) {
			player.cards.push(game.deck.pop()!);
		}
	}

	const nextTurn = () => {
		if (game.direction === "clockwise") {
			game.turn++;
		} else {
			game.turn--;
		}

		if (game.turn === game.players.length) {
			game.turn = 0;
		} else if (game.turn === -1) {
			game.turn = game.players.length - 1;
		}
	};

	const playCard = (player: String, card: UnoCard) => {
		if (!canPlayCard(card)) return;
		if (game.players[game.turn].name !== player) return;

		game.deck.unshift(card);
		game.players[game.turn].cards.splice(
			game.players[game.turn].cards.indexOf(card),
			1
		);

		if (card.type === "wild" || card.type === "wild-draw-four") {
			chooseColorOpen = true;
		} else {
			//@ts-ignore
			game.color = card.color;
		}

		if (card.type === "draw-two") {
			game.debtCards.push(game.deck.pop()!);
			game.debtCards.push(game.deck.pop()!);
		}

		if (card.type === "draw-four" || card.type === "wild-draw-four") {
			game.debtCards.push(game.deck.pop()!);
			game.debtCards.push(game.deck.pop()!);
			game.debtCards.push(game.deck.pop()!);
			game.debtCards.push(game.deck.pop()!);
		}

		if (card.type === "skip") {
			nextTurn();
		}

		if (card.type === "reverse") {
			game.direction =
				game.direction === "clockwise" ? "counter-clockwise" : "clockwise";
		}

		nextTurn();
	};

	const drawDebtCards = () => {
		for (let i = 0; i < game.debtCards.length; i++) {
			game.players[game.turn].cards.push(game.debtCards.pop()!);
		}

		nextTurn();
	};

	const drawCard = () => {
		game.players[game.turn].cards.push(game.deck.pop()!);

		if (game.debtCards.length > 0) {
			drawDebtCards();
		}

		nextTurn();
	};

	const canPlayCard = (card: UnoCard) => {
		const topCard = game.deck[0];

		if (card.color === "wild") {
			return true;
		}

		if (card.color === game.color) {
			return true;
		}

		if (card.type === topCard.type) {
			return true;
		}

		return false;
	};
</script>

<div
	class="chooseColor"
	style="
	transition: all 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
	{chooseColorOpen
		? ``
		: `
		pointer-events: none;
		opacity: 0;
	`}
"
>
	{#if chooseColorOpen}
		<div
			class="chooseColorContent"
			transition:scale={{
				delay: 100,
				duration: 200,
				start: 0.8,
			}}
		>
			<h1 class="chooseColorHeading">Choose a color</h1>

			<p>Choose wisely, this will be the color of the next card.</p>

			{#each ["red", "blue", "green", "yellow"] as color}
				<button
					on:click={() => {
						//@ts-ignore
						game.color = color;
						chooseColorOpen = false;
					}}
					class="chooseColorButton {color}"
				>
					{color}
				</button>
			{/each}
		</div>
	{/if}
</div>

<div class="grid">
	<div class="gridleft">
		<div class="cardsContainer">
			{#each game.players as player, i}
				<div class={["top", "right", "bottom", "left"][i]}>
					<h3 class="cardPlayerName">{player.name}</h3>

					<div class="cardsParent">
						{#each player.cards as card (card)}
							<div
								out:sendCard={{
									key: "card",
								}}
								in:receiveCard={{
									key: "card",
								}}
							>
								<Card
									{card}
									on:click={() => {
										if (canPlayCard(card)) {
											playCard(player.name, card);
										}
									}}
									showCard={player.name === game.players[game.turn].name}
								/>
							</div>
						{/each}
					</div>
				</div>
			{/each}
		</div>
	</div>

	<div class="gridright">
		{#if game.debtCards.length > 0}
			<br /><br />

			<b>
				Debt cards - {game.debtCards.length}
			</b>
		{/if}

		<br /><br />

		{#key game.deck[0]}
			<div
				in:receiveCard={{
					key: "card",
				}}
			>
				<Card card={game.deck[0]} on:click={drawCard} showCard />
			</div>
		{/key}
	</div>
</div>

<style>
	:global(body, html) {
		font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS",
			sans-serif;
		background: url(https://d33wubrfki0l68.cloudfront.net/4629bd964e9c60ed914da61ad48150a2cbd75533/cloth.jpg);
		color: #fff;
		color-scheme: dark;
		/*Take scrollbar into account*/
		scrollbar-gutter: stable;
		user-select: none;
	}

	.chooseColor {
		position: fixed;
		inset: 0;
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 9999;
		background: #0000008a;
		backdrop-filter: blur(17px);
	}

	.chooseColorContent {
		background: #00000066;
		padding: 15px 29px;
		border-radius: 27px;
		box-shadow: 0px 10px 20px black;
	}

	.chooseColorHeading {
		margin: 0;
		margin-bottom: 20px;
		font-size: 30px;
		font-weight: 400;
	}

	.chooseColorButton,
	.btn {
		display: block;
		width: 100%;
		border: none;
		margin-bottom: 10px;
		padding: 14px;
		border-radius: 19px;
		font-size: 20px;
		text-align: center;
		font-weight: 600;
		cursor: pointer;
		text-transform: uppercase;
		transition: all 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
	}

	.chooseColorButton:hover,
	.chooseColorButton:focus,
	.btn:hover,
	.btn:focus {
		transform: scale(0.98);
		opacity: 0.8;
	}

	.chooseColorButton:active,
	.btn:active {
		transform: scale(0.95);
		opacity: 0.8;
	}

	.chooseColorButton.red {
		background: #d1180a;
		color: #fff;
	}

	.chooseColorButton.blue {
		background: rgb(13, 77, 216);
		color: #fff;
	}

	.chooseColorButton.green {
		background: #099e20;
		color: #fff;
	}

	.chooseColorButton.yellow {
		background: #dad603;
		color: #000;
	}

	/**
	
	*/
	.gridright {
		position: fixed;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		padding: 10px;
		border-radius: 20px;
	}

	.cardsParent {
		display: flex;
		gap: 10px;
		border-radius: 13px;
		padding: 11px;
		flex-wrap: wrap;
		align-content: center;
		justify-content: center;
		overflow: auto;
	}

	.cardPlayerName {
		position: absolute;
		z-index: 99999;
		margin-left: 10px;
		background: black;
	}
	.top {
		top: 2vh;
		position: fixed;
		width: 96vw;
		right: 2vw;
		transform: rotate(180deg);
	}
	.right {
		position: fixed;
		width: 96vh;
		top: 2vh;
		right: 0;
		margin-right: 12vw;
		transform: rotate(-90deg);
		transform-origin: top right;
	}
	.left {
		position: fixed;
		width: 96vh;
		top: 2vh;
		left: 0;
		margin-left: 12vw;
		transform: rotate(90deg);
		transform-origin: top left;
	}
	.bottom {
		bottom: 5vh;
		position: fixed;
		width: 96vw;
		right: 2vw;
		transform-origin: bottom center;
	}
</style>
