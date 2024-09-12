<template>
  <section>
    <img src="@/assets/Tictactoe.svg" alt="Tictactoe" title="Game Tictactoe" />
    <div class="board-container">
      <div class="board">
        <div
          class="cell"
          v-for="(cell, index) in board"
          :key="index"
          @click="makeMove(index)"
        >
          {{ cell }}
        </div>
      </div>
      <div v-if="currentPlayer === 'O' && !gameOver">
        Wait for the bot to choose....
      </div>
      <button
        class="btn btn-cartoon"
        :class="currentPlayer === 'O' && !gameOver ? 'mt-2' : 'mt-0'"
        @click="resetBoard"
      >
        Reset
      </button>
    </div>
  </section>
</template>

<script>
import { AlertUtils } from "@/mixins/AlertUtils";

export default {
  mixins: [AlertUtils],
  name: "TictactoeGames",
  data() {
    return {
      board: Array(9).fill(""),
      currentPlayer: "X",
      gameOver: false,
    };
  },
  methods: {
    makeMove(index) {
      if (!this.board[index] && !this.gameOver) {
        if (this.currentPlayer === "O") {
          return; // Menunggu bot mengisi
        }
        this.board.splice(index, 1, this.currentPlayer);
        if (this.checkWin(this.currentPlayer)) {
          let player = this.currentPlayer === "O" ? "Bot" : "You";
          this.alertSuccess("Yeayy, " + player + " win!");
          this.gameOver = true;
        } else if (this.checkDraw()) {
          this.alertSuccess("It's Draw!");
          this.gameOver = true;
        } else {
          this.currentPlayer = this.currentPlayer === "X" ? "O" : "X";
          if (this.currentPlayer === "O") {
            this.currentPlayer = "O";
            setTimeout(() => {
              this.makeBotMove();
            }, 1000);
          }
        }
      }
    },
    checkWin(player) {
      const winConditions = [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8], // rows
        [0, 3, 6],
        [1, 4, 7],
        [2, 5, 8], // columns
        [0, 4, 8],
        [2, 4, 6], // diagonals
      ];
      return winConditions.some(
        (condition) =>
          this.board[condition[0]] === player &&
          this.board[condition[1]] === player &&
          this.board[condition[2]] === player
      );
    },
    checkDraw() {
      return this.board.every((cell) => cell !== "");
    },
    resetBoard() {
      this.board = Array(9).fill("");
      this.currentPlayer = "X";
      this.gameOver = false;
    },
    makeBotMove() {
      // Logic untuk menentukan langkah bot
      let emptyCells = [];
      for (let i = 0; i < this.board.length; i++) {
        if (this.board[i] === "") {
          emptyCells.push(i);
        }
      }
      const randomIndex = Math.floor(Math.random() * emptyCells.length);
      const botMove = emptyCells[randomIndex];
      this.board.splice(botMove, 1, this.currentPlayer);

      if (this.checkWin(this.currentPlayer)) {
        this.alertSuccess("Sorry, Bot win!");
        this.gameOver = true;
      } else if (this.checkDraw()) {
        alert("It's a draw!");
        this.gameOver = true;
      } else {
        this.currentPlayer = this.currentPlayer === "X" ? "O" : "X";
      }
    },
  },
};
</script>

<style scoped>
section {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.board-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 15px;
  max-width: 400px;
  width: 100%;
  margin: 20px 0;
}

.cell {
  border: 2px solid #333;
  width: 100%;
  height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36px;
  font-weight: bold;
  cursor: pointer;
  background-color: #fff;
  transition: background-color 0.3s ease;
  box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
  -webkit-box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
  -moz-box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
}

.cell:hover {
  background-color: #eee;
}

.btn-cartoon {
  font-size: 16px;
  font-weight: 700;
  border: 2px solid black;
  border-radius: 0;
  box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
  -webkit-box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
  -moz-box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 1);
}

@media (max-width: 480px) {
  .board {
    max-width: 300px;
  }

  .cell {
    font-size: 24px;
    height: 90px;
  }
}
</style>
