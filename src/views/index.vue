<template>
  <section>
    <img
      src="@/assets/Tictactoe.svg"
      alt="Tictactoe"
      title="Game Tictactoe"
      class="mb-3"
    />
    <span>If you can beat my algorithm, you're damn good.</span>
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
    <footer class="mt-auto">
      made with <span style="color: #e25555">&hearts;</span> by
      <a href="https://www.linkedin.com/in/tegarumarabdillah/" target="_blank"
        >Tegar</a
      >
    </footer>
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
      botLevel: "hard", // level 'easy', 'medium', atau 'hard'
    };
  },
  methods: {
    makeMove(index) {
      if (!this.board[index] && !this.gameOver) {
        if (this.currentPlayer === "O") {
          return; // Nunggu bot
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
      this.botLevel = "hard";
      this.gameOver = false;
    },
    makeBotMove() {
      if (this.botLevel === "easy") {
        this.makeEasyMove();
      } else if (this.botLevel === "medium") {
        this.makeMediumMove();
      } else if (this.botLevel === "hard") {
        this.makeHardMove();
      }
    },
    makeEasyMove() {
      let emptyCells = this.getEmptyCells();
      const randomIndex = Math.floor(Math.random() * emptyCells.length);
      const botMove = emptyCells[randomIndex];
      this.makeMoveForBot(botMove);
    },
    makeMediumMove() {
      let emptyCells = this.getEmptyCells();
      // Cek apakah pemain akan menang di langkah selanjutnya
      for (let i = 0; i < emptyCells.length; i++) {
        const testBoard = [...this.board];
        testBoard[emptyCells[i]] = "X"; // Simulasikan langkah pemain
        if (this.checkWin("X")) {
          this.makeMoveForBot(emptyCells[i]); // Jika pemain akan menang, cegah ajik paur
          return;
        }
      }
      // Jika tidak ada yang harus dicegah, pilih langkah secara acak
      this.makeEasyMove();
    },
    makeHardMove() {
      const bestMove = this.minimax(this.board, "O").index;
      this.makeMoveForBot(bestMove);
    },
    getEmptyCells() {
      let emptyCells = [];
      for (let i = 0; i < this.board.length; i++) {
        if (this.board[i] === "") {
          emptyCells.push(i);
        }
      }
      return emptyCells;
    },
    makeMoveForBot(index) {
      this.board.splice(index, 1, this.currentPlayer);
      if (this.checkWin(this.currentPlayer)) {
        this.alertSuccess("Sorry, Bot win!");
        this.gameOver = true;
      } else if (this.checkDraw()) {
        this.alertSuccess("It's a draw!");
        this.gameOver = true;
      } else {
        this.currentPlayer = this.currentPlayer === "X" ? "O" : "X";
      }
    },
    minimax(board, player) {
      const emptyCells = this.getEmptyCells();

      if (this.checkWin("X")) {
        return { score: -10 };
      } else if (this.checkWin("O")) {
        return { score: 10 };
      } else if (emptyCells.length === 0) {
        return { score: 0 };
      }

      let moves = [];

      for (let i = 0; i < emptyCells.length; i++) {
        let move = {};
        move.index = emptyCells[i];
        board[emptyCells[i]] = player;

        if (player === "O") {
          let result = this.minimax(board, "X");
          move.score = result.score;
        } else {
          let result = this.minimax(board, "O");
          move.score = result.score;
        }

        board[emptyCells[i]] = "";
        moves.push(move);
      }

      let bestMove;
      if (player === "O") {
        let bestScore = -10000;
        for (let i = 0; i < moves.length; i++) {
          if (moves[i].score > bestScore) {
            bestScore = moves[i].score;
            bestMove = i;
          }
        }
      } else {
        let bestScore = 10000;
        for (let i = 0; i < moves.length; i++) {
          if (moves[i].score < bestScore) {
            bestScore = moves[i].score;
            bestMove = i;
          }
        }
      }
      return moves[bestMove];
    },

    secretKeyToEasy(e) {
      if (e.key == "/") {
        this.botLevel = "easy";
      }
    },
  },
  mounted() {
    window.addEventListener("keydown", this.secretKeyToEasy);
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.secretKeyToEasy);
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
  position: relative;
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

footer {
  position: absolute;
  bottom: 0;
  width: 100%;
  text-align: center;
  padding: 10px 0;
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
