<template>
    <div>
      <div id="game-setup" v-if="!gameStarted">
        <div>
          <input v-model="player1" placeholder="Player 1 Name" />
          <button @click="createGame">Create Game</button>
        </div>
        <div>
          <input v-model="gameId" placeholder="Enter Game ID" />
          <input v-model="player2" placeholder="Player 2 Name" />
          <button @click="joinGame">Join Game</button>
        </div>
      </div>
  
      <div id="game-board" v-if="gameStarted">
        <div class="grid">
          <div
            class="square"
            v-for="(value, index) in board"
            :key="index"
            @click="makeMove(index)"
          >
            {{ value }}
          </div>
        </div>
        <div id="status">{{ statusMessage }}</div>
        <button v-if="isGameOver" @click="startNewGame">Start New Game</button>
      </div>
    </div>
  </template>
  
<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';
import axios from 'axios';

const player1 = ref('');
const player2 = ref('');
const gameId = ref('');
const board = ref<string[]>(Array(9).fill(''));
const currentPlayer = ref('X');
const playerTurn = ref('X');
const gameStarted = ref(false);
const isGameOver = ref(false);
const apiUrl = 'http://127.0.0.1:8000/api/games';

let checkStatusInterval: number | null = null;

// Lifecycle hooks to manage status checks
onMounted(() => {
  if (gameStarted.value) {
    startStatusCheck();
  }
});

onBeforeUnmount(() => {
  stopStatusCheck();
});

// Game status message
const statusMessage = computed(() => {
  if (isGameOver.value) {
    return `${playerTurn.value} wins!`;
  }
  return `Current Player: ${playerTurn.value}`;
});

const createGame = async () => {
  if (!player1.value) {
    return alert("Player 1 name is required.");
  }

  try {
    // Send API request to create a new game
    const response = await axios.post(apiUrl, {
      player1: player1.value,
    });

    // Set game id
    gameId.value = response.data.id;

    // Initialize the game board, reset the game board and start a status check
    gameStarted.value = true;
    startStatusCheck();
  } catch (error) {
    alert('Failed to create game.');
    console.error(error);
  }
};

// Function to join a game
const joinGame = async () => {
  if (!player2.value || !gameId.value) {
    return alert("Player 2 name and Game ID are required.");
  }

  try {
    // Send a request to the api to join a game
    await axios.post(`${apiUrl}/${gameId.value}/join`, {
      player2: player2.value,
    });

    // Initialize the game board, reset the game board and start a status check
    gameStarted.value = true;
    currentPlayer.value = 'O'; // Set currentPlayer to 'O' for Player 2
    resetBoard();
    startStatusCheck();
  } catch (error) {
    alert('Failed to join game.');
    console.error(error);
  }
};

// Function to make a move
const makeMove = async (index: number) => {
  if (board.value[index] || isGameOver.value) {
    return; // Ignore if already occupied or game over
  }

  try {
    // Send a request to the api to make a move
    const response = await axios.post(`${apiUrl}/${gameId.value}/move`, {
      position: index,
      player: currentPlayer.value,
    });
    board.value = response.data.board;

    // Check response to see if the game winning condition has been met
    if (response.data.status === 'finished') {
      isGameOver.value = true;
      stopStatusCheck();
      return;
    }

    // Switch player turn
    playerTurn.value = playerTurn.value === 'X' ? 'O' : 'X';
    
  } catch (error) {
    alert('Invalid move.');
    console.error(error);
  }
};

// Function to start a new game
const startNewGame = () => {
  resetBoard();
  isGameOver.value = false;
  playerTurn.value = 'X';
};

// Function to reset the board
const resetBoard = () => {
  board.value = Array(9).fill('');
};

// Function to fetch the current game state
const fetchGameState = async () => {
  if (!gameId.value) return;

  try {
    const response = await axios.get(`${apiUrl}/${gameId.value}`);
    board.value = response.data.board;

    // Set player turn
    playerTurn.value = response.data.currentPlayer ?? playerTurn.value;

    if (response.data.status === 'finished') {
      isGameOver.value = true;
      stopStatusCheck();
    }
  } catch (error) {
    console.error('Failed to fetch game state.', error);
  }
};

// Function to check status of the game
const startStatusCheck = () => {
  stopStatusCheck();
  checkStatusInterval = window.setInterval(fetchGameState, 3000); // Check game status every 3 seconds
};

// Function to stop status check
const stopStatusCheck = () => {
  if (checkStatusInterval !== null) {
    clearInterval(checkStatusInterval);
    checkStatusInterval = null;
  }
};

</script>

  
  <style scoped>
  .grid {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    gap: 5px;
    margin: 20px auto;
  }
  
  .square {
    width: 100px;
    height: 100px;
    background-color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2em;
    border: 2px solid #333;
    cursor: pointer;
  }
  
  .square:hover {
    background-color: #f0f0f0;
  }
  
  #status {
    margin-top: 20px;
    font-size: 1.2em;
  }
  </style>
