<script setup>
import { onMounted, ref } from "vue";
import { Chess } from "chess.js";
import {
  Chessboard,
  COLOR,
  BORDER_TYPE,
  MARKER_TYPE,
  INPUT_EVENT_TYPE,
} from "cm-chessboard";
import sprite from "cm-chessboard/assets/images/chessboard-sprite.svg";
import "cm-chessboard/assets/styles/cm-chessboard.css";

const props = defineProps({
  fen: String,
  auto: Boolean,
});

const chess = new Chess();

const chessRef = ref(null)

let chessboard;

const fen = props.fen
    ? chess.validate_fen(props.fen).valid 
        ? props.fen 
        : "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1"
    : "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1"


chess.load(fen);

const auto = props.auto || false;

const color =  fen.includes(" w ") ? COLOR.white : COLOR.black

const defaultProps = {
  position: fen || "start", // set as fen, "start" or "empty"
  orientation: color, // white on bottom
  responsive: true, // resize the board automatically to the size of the context element
  animationDuration: 300, // pieces animation duration in milliseconds. Disable all animation with `0`.
  language: navigator.language.substring(0, 2).toLowerCase(), // supports "de" and "en" for now, used for pieces naming
  style: {
    cssClass: "default", // set the css theme of the board, try "green", "blue" or "chess-club"
    showCoordinates: true, // show ranks and files
    borderType: BORDER_TYPE.none, // "thin" thin border, "frame" wide border with coordinates in it, "none" no border
    aspectRatio: 1, // height/width of the board
    moveFromMarker: MARKER_TYPE.frame, // the marker used to mark the start square
    moveToMarker: MARKER_TYPE.frame,
  },
  sprite: {
    url: sprite, //"https://cdn.jsdelivr.net/npm/cm-chessboard@4.1.13/assets/images/chessboard-sprite.svg", // pieces and markers are stored in a sprite file
    size: 40, // the sprite tiles size, defaults to 40x40px
    cache: true, // cache the sprite
  },
  accessibility: {
    movePieceForm: true, // display a form to move a piece (from, to, move)
    boardAsTable: true, // display the board additionally as HTML table
    piecesAsList: true, // display the pieces additionally as List
  },
  extensions: [
    /* {class: ExtensionClass, props: { ... }} */
  ], // add extensions here
};

const randomMove = (event) => {
  const chessMove = { from: event.squareFrom, to: event.squareTo };
  const result = chess.move(chessMove);
  if (result) {
    event.chessboard.setPosition(chess.fen(), true);
    event.chessboard.disableMoveInput();
    const boardMoves = chess.moves({ verbose: true });
    if (boardMoves.length) {
      const randomIndex = Math.floor(Math.random() * boardMoves.length);
      const boardMove = boardMoves[randomIndex];
      // smoother with 500ms delay
        setTimeout(() => {
          console.log(chess.fen())
          chess.move({ from: boardMove.from, to: boardMove.to });
          event.chessboard.enableMoveInput(handleInput, color);
          event.chessboard.setPosition(chess.fen(), true);
        }, 500);
    } else {
      alert("Game Over!");
    }
  } else {
    alert("Invalid move");
  }
};

const makeRandomMove = (callback) => {
  const possibleMoves = chess.moves()
  if (possibleMoves.length === 0) {
      alert("Game Over!")
      return
  }
  const randomIndex = Math.floor(Math.random() * possibleMoves.length)
  chess.move(possibleMoves[randomIndex])
  chessboard.setPosition(chess.fen(), true)
  if (callback) callback()
}


const handleInput = (event) => {
  event.chessboard.removeMarkers(undefined, MARKER_TYPE.dot);
  event.chessboard.removeMarkers(undefined, MARKER_TYPE.square);
  if (event.type === INPUT_EVENT_TYPE.moveStart) {
    const moves = chess.moves({ square: event.square, verbose: true });
    event.chessboard.addMarker(event.square, MARKER_TYPE.square);
    for (const move of moves) {
      // draw dots on possible moves
      event.chessboard.addMarker(move.to, MARKER_TYPE.dot);
    }
    return moves.length > 0;
  } else if (event.type === INPUT_EVENT_TYPE.moveDone) {
    randomMove(event)
  }
};


const undo = () => {
  chess.undo()
  chess.undo()
  chessboard.setPosition(chess.fen(), true);
}

onMounted(() => {
  console.log(chessRef.value)
  const board = chessRef.value;
  chessboard = new Chessboard(board, defaultProps);
  if (auto) {
      setInterval(() => {
        makeRandomMove()
      }, 1000)
  }
  else chessboard.enableMoveInput(handleInput, color);
  
});

</script>

<template>
  <div class="fen">Fen: {{ fen }}</div>
  <div id="board" ref="chessRef"></div>
  <button @click="undo" class="btn">Undo</button>
</template>

<style scoped>
.btn {
  border: none;
  outline: none;
  padding: 12px 36px;
  border-radius: 8px;
  color: #fff;
  background-color: #17abfc;
  margin-top: 10px;
}
.fen {
  margin: 10px 0;
  font-family: Times New Roman;
  font-weight: 400;
}
</style>
