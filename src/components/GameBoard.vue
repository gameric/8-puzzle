<template>
  <div class="game-container">
    <h1 class="title">8-Puzzle Game</h1>
    <table class="board-table" :class="{ solved: isSolved }">
      <tbody>
        <tr v-for="(row, rowIndex) in board" :key="rowIndex">
          <Tile
            v-for="(tile, index) in row"
            :tileNumber="tile"
            :key="tile"
            :rowNumber="rowIndex"
            :columnNumber="index"
            v-on:tile-clicked="onTileClicked"
          />
        </tr>
      </tbody>
    </table>
    <div class="state-container">
      <span>{{ timer }} Second{{ timer !== 1 ? "s" : "" }} </span>
      <span> {{ moves }} Move{{ moves !== 1 ? "s" : "" }} </span>
    </div>
    <div v-if="isSolved">
      <button @click="$emit('new-game')" class="new-game">New Game?</button>
    </div>
    <a href="https://github.com/gameric/8-puzzle" v-if="timer > 30 && !isSolved"
      >Don't know how to solve?</a
    >
  </div>
</template>

<script>
import Tile from "@/components/BoardTile.vue";
import {
  getBoard,
  findBlank,
  isSamePoint,
  addPoints,
  swapPoints,
} from "@/assets/board";

const correctBoard = getBoard();

export default {
  data() {
    return {
      board: getBoard(true),
      gameStarts: false,
      moves: 0,
      timer: 0,
    };
  },
  created() {
    this.blank = findBlank(this.board);
  },
  computed: {
    isSolved() {
      for (let i = 0; i < 3; i++)
        for (let j = 0; j < 3; j++)
          if (this.board[i][j] !== correctBoard[i][j]) return false;

      return true;
    },
  },
  watch: {
    gameStarts() {
      this.timer++;
    },
    timer() {
      setTimeout(() => {
        if (!this.isSolved && this.moves) this.timer++;
      }, 1000);
    },
  },
  methods: {
    onTileClicked(clickedTile) {
      if (this.isSolved || isSamePoint(clickedTile, this.blank)) return;
      const moves = [
        { x: 1, y: 0 },
        { x: -1, y: 0 },
        { x: 0, y: -1 },
        { x: 0, y: 1 },
      ];
      for (const move of moves) {
        // check if moving one step in horizontal or vertical direction from the clicked tile
        // leads to the blank tile, then this is a movable tile
        const point = addPoints(clickedTile, move);
        if (isSamePoint(point, this.blank)) {
          swapPoints(this.board, clickedTile, this.blank);
          this.blank = clickedTile;
          this.moves++;
          this.gameStarts = true;
          break;
        }
      }
    },
  },
  components: {
    Tile,
  },
};
</script>

<style scoped>
.game-container {
  user-select: none;
  color: var(--text);
  background-color: var(--bg-dark);
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

a {
  color: var(--text);
}

.state-container {
  min-width: 250px;
  justify-content: space-around;
  display: flex;
  margin: 10px;
  border-radius: 5px;
  background-color: var(--bg-light);
  padding: 10px;
  font-size: 20px;
  box-shadow: 0.5px 0.5px 0.5px 0.5px var(--dark);
}

.board-table {
  border: 1px solid var(--dark);
  border-radius: 5px;
  margin: 5px;
}

.new-game {
  border-radius: 5px;
  background-color: var(--bg-light);
  color: var(--text);
  padding: 10px;
  font-size: 15px;
  box-shadow: 1px 1px 1.5px 1.5px var(--dark);
  border-width: 0.6px;
}

.solved {
  background-color: var(--solved);
}
</style>
