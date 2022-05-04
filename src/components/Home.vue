<template>
  <div class="reels" v-if="playedTriplets.length">
    <p v-for="index in playedTriplets.length" :key="index" :id="index">
      <span class="triplet" v-for="(triplet, i) in playedTriplets" :key="i">
        {{ triplet[index - 1] }}
      </span>
    </p>
  </div>
  <div v-else>
    <img src="@/assets/777pic.jpg" alt="777pic" height="150" />
  </div>
  <div>
    <button @click="start">Start</button>
    <button @click="autoStart">Autostart</button>
    <button @click="this.autoStop = true">Stop</button>
  </div>

  <div>
    <button @click="bet >= 10 ? (bet -= 5) : bet">-</button>
    <span class="title">Bet {{ bet }}</span>
    <button @click="bet <= credits - 5 ? (bet += 5) : bet">+</button>
    <span class="title">Credits {{ credits }}</span>

    <form v-on:submit.prevent="onSubmit">
      <button @click="credits += addCredits">Add credits</button>
      <label for="add"></label>
      <input id="add" type="number" v-model="addCredits" />
    </form>
  </div>
  <div v-if="prize">
    <p>{{ message }}</p>
    <p>{{ prize }}</p>
  </div>
  <div v-else-if="credits < bet">
    <p>{{ message }}</p>
  </div>
  <div class="statistics">
    <button @click="statistics = !statistics">Statistics</button>
    <div v-if="statistics">
      <p>
        Return To Player(RTP): <span class="number">{{ returnToPlayer }}</span>
      </p>
      <p>
        Total games played: <span class="number">{{ games }}</span>
      </p>
      <p>
        All player's bets sum:<span class="number">{{ playerBet }}</span>
      </p>
      <p>
        Player won:<span class="number">{{ wins }}</span>
      </p>
      <p>
        Payment amount:<span class="number">{{ winningsSum }}</span>
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: "HomeView",
  data: () => ({
    //изначально заданные ленты
    base_reels: [
      [1, 3, 3, 3, 3, 2, 7, 4, 4, 4, 4, 7, 7, 5, 5, 5, 5, 2, 2, 0, 6, 6, 6, 6],
      [1, 7, 5, 7, 5, 5, 5, 2, 6, 2, 6, 6, 6, 6, 3, 0, 3, 3, 3, 7, 4, 7, 4, 4],
      [1, 3, 3, 3, 3, 7, 7, 0, 4, 4, 4, 4, 7, 7, 5, 5, 5, 5, 2, 2, 6, 6, 6, 6],
    ],
    triplets: [], //все возможные "тройки" для каждой ленты
    playedTriplets: [], //"выигравшие" тройки
    prizes: [750, 200, 60, 40, 40, 40, 40, 5], //стоимости выигрышей
    prize: 0,
    message: "",
    bet: 5,
    credits: 100,
    addCredits: 0,
    autoStop: false,
    games: 0, //всего сыграно игр
    playerBet: 0, //сумма всех ставок игрока
    wins: 0, //выиграно игр
    winningsSum: 0, //сумма всех выигрышей
    returnToPlayer: 0, //сумма всех выигрышей делится на сумму ставок
    statistics: false,
  }),
  methods: {
    //получение всех возможных "троек"; вычисляется до начала игры
    getTriplets() {
      this.base_reels.forEach((reel, index) => {
        this.triplets[index] = Array.from(reel, (item, pos) => {
          if (!pos) {
            return [reel[reel.length - 1], item, reel[1]];
          } else if (pos === reel.length - 1)
            return [reel[pos - 1], item, reel[0]];
          else return [reel[pos - 1], item, reel[pos + 1]];
        });
      });
    },
    //вычисление случайной стоповой позиции в зависимости от длины ленты
    getRandom(length) {
      return Math.floor(Math.random() * length);
    },
    start() {
      this.games++;
      this.playerBet = this.playerBet + this.bet;
      this.prize = 0;
      this.message = "";
      if (this.credits >= this.bet) {
        this.playedTriplets = [];
        for (let reel of this.triplets) {
          let stopPosition = this.getRandom(reel.length);
          this.playedTriplets.push(reel[stopPosition]);
        }
        this.credits -= this.bet;
        this.isWin();
        this.returnToPlayer = this.winningsSum / this.playerBet;
      } else {
        this.message = "CREDITS: 0 - ADD COINS";
      }
    },
    autoStart() {
      this.autoStop = false;
      const auto = setInterval(() => {
        this.start();
        if (this.credits < this.bet || this.autoStop === true) {
          clearInterval(auto);
          this.message = "CREDITS: 0 - ADD COINS";
        }
      }, 600);
    },
    isWin() {
      if (
        this.playedTriplets[0][1] === this.playedTriplets[1][1] &&
        this.playedTriplets[1][1] === this.playedTriplets[2][1]
      ) {
        let i = this.playedTriplets[0][1];
        this.prize = (this.bet / 5) * this.prizes[i];
        this.credits += this.prize;
        this.message = "YOU WIN!!";
        this.wins++;
        this.winningsSum = this.winningsSum + this.prizes[i];
      }
    },
  },
  created() {
    this.getTriplets();
  },
};
</script>

<style scoped>
.reels {
  height: 150px;
  margin-bottom: 1rem;
}
button {
  font-size: 1.5rem;
  margin: 1rem;
}
.triplet {
  margin: 0 1rem;
  font-weight: bold;
  font-size: 2.25rem;
}
.title,
input {
  font-size: 1.25rem;
}
.statistics {
  font-size: 1.25rem;
  text-align: left;
  margin-left: 10vw;
}
.number {
  margin-left: 2rem;
}
</style>
