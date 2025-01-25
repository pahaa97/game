<template>
  <div id="app">
    <h1>Guess the Number</h1>
    <p>Guess a number between 1 and 100</p>
    <input type="number" v-model.number="guess" placeholder="Enter your guess" />
    <button @click="sendGuess" :disabled="gameOver">Guess</button>
    <button @click="resetGame">Reset</button>

    <p v-if="message" class="message">{{ message }}</p>

    <h2>Guess History</h2>
    <ul>
      <li
        v-for="(item, index) in sortedGuesses"
        :key="index"
        :style="{ backgroundColor: getColor(item.difference) }"
        class="guess-item"
      >
        {{ item.number }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data () {
    return {
      ws: null, // WebSocket-соединение
      guess: null, // Текущая попытка игрока
      message: '', // Сообщения для игрока
      gameOver: false, // Флаг окончания игры
      guesses: [] // История всех попыток
    }
  },
  computed: {
    // Сортировка попыток от ближайшей к дальней
    sortedGuesses () {
      return this.guesses.slice().sort((a, b) => a.difference - b.difference)
    }
  },
  methods: {
    // Подключение к WebSocket
    connectWebSocket () {
      this.ws = new WebSocket('wss://3214-195-189-33-9.ngrok-free.app')

      this.ws.onopen = () => {
        this.message = 'Connected to the game!'
      }

      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data)

        if (data.type === 'winner') {
          this.message = `Player guessed the number ${data.guess}! Game Over.`
          this.gameOver = true
        } else if (data.type === 'update-history') {
          this.guesses = data.guesses // Обновляем историю
        } else if (data.type === 'reset') {
          this.message = data.message
          this.guesses = [] // Очищаем историю
          this.gameOver = false
        }
      }

      this.ws.onclose = () => {
        this.message = 'Disconnected from the server.'
      }
    },
    // Отправка попытки игрока
    sendGuess () {
      if (this.ws && this.guess !== null && !this.gameOver) {
        this.ws.send(JSON.stringify({ type: 'guess', guess: this.guess }))
        this.guess = null
      }
    },
    // Генерация цвета на основе разницы
    getColor (difference) {
      if (difference === 0) {
        return 'green' // Угадано
      } else if (difference <= 10) {
        return 'lightgreen' // Очень близко
      } else if (difference <= 30) {
        return 'yellow' // Средняя дистанция
      } else {
        return 'red' // Очень далеко
      }
    },
    // Сброс игры
    resetGame () {
      if (this.ws) {
        this.ws.send(JSON.stringify({ type: 'reset' })) // Отправляем сброс на сервер
      }
    }
  },
  mounted () {
    this.connectWebSocket()
  }
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 40px;
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
  padding: 20px;
}

input {
  padding: 10px;
  font-size: 16px;
  width: 150px;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  margin: 5px;
}

.message {
  margin-top: 20px;
  font-size: 18px;
}

ul {
  list-style-type: none;
  padding: 0;
}

.guess-item {
  font-size: 16px;
  margin: 5px 0;
  padding: 10px;
  border-radius: 10px;
  color: black; /* Цвет текста */
  text-align: center;
  width: 100%; /* Ширина списка */
}
</style>
