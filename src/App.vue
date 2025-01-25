// Frontend (Vue.js)
<template>
  <div id="app">
    <h1>Tic Tac Toe</h1>
    <div class="board">
      <div
        v-for="(cell, index) in board"
        :key="index"
        class="cell"
        :class="{ 'cell-x': cell === 'X', 'cell-o': cell === 'O' }"
        @click="makeMove(index)"
      >
        {{ cell }}
      </div>
    </div>
    <p v-if="message" class="message">{{ message }}</p>
    <button @click="resetGame">Reset</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      ws: null, // WebSocket connection
      board: Array(9).fill(null), // Game board
      playerSymbol: '', // Player's symbol ('X' or 'O')
      message: '', // Game message
    };
  },
  methods: {
    connectWebSocket() {
      this.ws = new WebSocket('wss://fbbb-195-189-33-9.ngrok-free.app ');

      this.ws.onopen = () => {
        this.message = 'Connected to the game!';
      };

      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data);

        if (data.type === 'assign-symbol') {
          this.playerSymbol = data.symbol;
          this.message = `You are player ${data.symbol}`;
        } else if (data.type === 'update-board') {
          this.board = data.board;
          this.message = data.message;
        } else if (data.type === 'reset') {
          this.board = Array(9).fill(null);
          this.message = data.message;
        }
      };

      this.ws.onclose = () => {
        this.message = 'Disconnected from the server.';
      };
    },
    makeMove(index) {
      if (this.ws && this.board[index] === null && this.playerSymbol) {
        this.ws.send(JSON.stringify({ type: 'make-move', index, symbol: this.playerSymbol }));
      }
    },
    resetGame() {
      if (this.ws) {
        this.ws.send(JSON.stringify({ type: 'reset' }));
      }
    },
  },
  mounted() {
    this.connectWebSocket();
  },
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 40px;
}
.board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-gap: 5px;
  margin: 20px auto;
}
.cell {
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid black;
  font-size: 24px;
  cursor: pointer;
}
.cell-x {
  color: red;
}
.cell-o {
  color: blue;
}
.message {
  font-size: 18px;
  margin-top: 20px;
}
</style>
