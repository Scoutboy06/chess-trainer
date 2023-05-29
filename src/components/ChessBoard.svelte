<script lang="ts">
  import { Chess, SQUARES, type PieceSymbol, type Square, type Color } from "chess.js";
	const chess = new Chess();

	let pieces: { square: Square; type: PieceSymbol; color: Color }[] = getPieces();

	let board: HTMLElement;
	let boardSize = 800;
	let isFlipped = false;

	let selectedPieceSquare: Square | undefined;
	let shouldDeselectPiece = false;
	let draggedPieceSquare: Square | undefined;
	let draggedPiecePos: { x: number, y: number } | undefined;

	function mouseDownHandler(e: MouseEvent) {
		const target = e.target as HTMLElement;

		if(!target.classList.contains('piece')) return;

		const { x: boardX, y: boardY } = board.getBoundingClientRect();

		let rowIndex = Math.floor((e.clientY - boardY) / boardSize * 8);
		let colIndex = Math.floor((e.clientX - boardX) / boardSize * 8);

		draggedPieceSquare = square(rowIndex, colIndex);

		if(!selectedPieceSquare) selectedPieceSquare = draggedPieceSquare;

		window.addEventListener('mousemove', pieceDrag);
		window.addEventListener('mouseup', pieceRelease);
		pieceDrag(e);
	}

	function pieceDrag(e: MouseEvent) {
		const { x, y } = board.getBoundingClientRect();
		draggedPiecePos = {
			x: e.clientX - x - (boardSize / 16),
			y: e.clientY - y - (boardSize / 16),
		}
	}

	function pieceRelease(e: MouseEvent) {
		const { x, y } = board.getBoundingClientRect();
		const rowIndex = Math.floor((e.clientY - y) / boardSize * 8);
		const colIndex = Math.floor((e.clientX - x) / boardSize * 8);

		const from = draggedPieceSquare;
		const to = square(rowIndex, colIndex);

		if(from === to) {
			if(shouldDeselectPiece) selectedPieceSquare = undefined;
			shouldDeselectPiece = !shouldDeselectPiece;
		}
		
		else if(rowIndex >= 0 && rowIndex <= 7 && colIndex >= 0 && colIndex <= 7) {
			selectedPieceSquare = undefined;
			shouldDeselectPiece = false;

			try {
				chess.move(`${from}-${to}`);
				pieces = getPieces()
			} catch(err) {
				console.error(err)
			}
		}


		// Cleanup
		window.removeEventListener('mousemove', pieceDrag);
		window.removeEventListener('mouseup', pieceRelease);
		draggedPiecePos = undefined;
		draggedPieceSquare = undefined;
	}

	function colToX(col: number | string): number {
		if(typeof col === 'number') {
			return isFlipped
				? (7 - col) * boardSize / 8
				: col * boardSize / 8
		} else {
			return isFlipped
				? ('hgfedcba'.indexOf(col)) * boardSize / 8
				: 'abcdefgh'.indexOf(col) * boardSize / 8
		}
	}

	function rowToY(row: number | string): number {
		return isFlipped
			? (Number(row) - 1) * boardSize / 8
			: (8 - Number(row)) * boardSize / 8
	}

	function square(row: number, col: number): Square {
		if(isFlipped) return SQUARES[8 * (7 - row) + (7 - col)];
		else return SQUARES[8 * row + col];
	}

	function getPieces(): typeof pieces  {
		let pArr: ReturnType<typeof getPieces> = [];

		for(const row of chess.board()) {
			for(const piece of row) {
				if(piece) pArr.push(piece);
			}
		}

		return pArr;
	}
</script>

<div
	class="board"
	style:width="{boardSize}px"
	style:height="{boardSize}px"
	bind:this={board}
	on:mousedown={mouseDownHandler}
>
	{#each pieces as piece}
		<div
			class="piece {piece.color === 'b' ? 'black' : 'white'} {piece.type} {piece.square === draggedPieceSquare ? 'dragged' : ''}"
			style:--x="{
				piece.square === draggedPieceSquare
					? draggedPiecePos?.x
					: colToX(piece.square[0])
			}px"
			style:--y="{
				piece.square === draggedPieceSquare
					? draggedPiecePos?.y
					: rowToY(piece.square[1])
			}px"
		></div>
	{/each}

	{#if selectedPieceSquare}
		<div
			class="square selected"
			style:--x="{colToX(selectedPieceSquare[0])}px"
			style:--y="{rowToY(selectedPieceSquare[1])}px"
		></div>
	{/if}
</div>


<style>
.board {
	background-color: #000;
	background-image: url('boards/lichess.svg');
	background-size: 100%;
	position: relative;
	cursor: pointer;
	user-select: none;
}

.square {
	width: 12.5%;
	height: 12.5%;
	position: absolute;
	z-index: 1;
}

.square.selected {
	background-color: #5e43b9bd;
	transform: translate(var(--x), var(--y));
}

.square.hover {
	border: .5rem solid violet;
}

.piece {
	width: 12.5%;
	height: 12.5%;
	position: absolute;
	transform: translate(var(--x), var(--y));
	display: grid;
	place-items: center;
	background-size: 100%;
	transition: background-color .08s;
	z-index: 2;
}

.piece.dragged {
	z-index: 3;
}

.piece.white.r {
	background-image: url('/pieces/wr.svg');
}
.piece.white.n {
	background-image: url('/pieces/wn.svg');
}
.piece.white.b {
	background-image: url('/pieces/wb.svg');
}
.piece.white.q {
	background-image: url('/pieces/wq.svg');
}
.piece.white.k {
	background-image: url('/pieces/wk.svg');
}
.piece.white.p {
	background-image: url('/pieces/wp.svg');
}
.piece.black.r {
	background-image: url('/pieces/br.svg');
}
.piece.black.n {
	background-image: url('/pieces/bn.svg');
}
.piece.black.b {
	background-image: url('/pieces/bb.svg');
}
.piece.black.q {
	background-image: url('/pieces/bq.svg');
}
.piece.black.k {
	background-image: url('/pieces/bk.svg');
}
.piece.black.p {
	background-image: url('/pieces/bp.svg');
}

.flipBoard {
	position: absolute;
	right: -5px;
	translate: 100% 0;
	top: 5px;
}

</style>