<template>
  <div>
    <div v-if="loading" class="centered">
      <p>Loading...</p>
    </div>
    <div v-else>
      <p class="joke">{{ joke }}</p>

      <div class="button-container">
        <button @click="likeJoke" :disabled="likeButtonDisabled" class="btn"><icon name="thumbs-up"></icon></button>
        <button @click="logJokes" class="btn"><icon name="list"></icon></button>
        <button @click="clearStorage" class="btn"><icon name="trash"></icon></button>
      </div>

      <ul v-show="displayJokeList" class="list">
        <li v-for="joke in favoriteJokeList">
          {{ joke }}
        </li>
      </ul>
    </div>
  </div>
</template>
c
<script>
import axios from 'axios';

export default {
  data () {
    return {
      loading: true,
      joke: "",
      likeButtonDisabled: false,
      favoriteJokeList: [],
      displayJokeList: false
    }
  },
  methods: {
    likeJoke(){
      chrome.storage.local.get("jokes", (res) => {
        if(!res.jokes) res.jokes = [];
        res.jokes.push(this.joke);
        chrome.storage.local.set(res);
        this.likeButtonDisabled = true;
        this.searchFavoriteJokeList();
      });
    },
    logJokes(){
      chrome.storage.local.get("jokes", (res) => {
        console.log(`chrome storage local: \n${JSON.stringify(res, null, 2)}`);
        if (res.jokes) {
          res.jokes.map(joke => {
            console.log(joke);
          });
        }
      });
      this.displayJokeList = !this.displayJokeList;
    },
    clearStorage(){
      chrome.storage.local.clear();
      this.searchFavoriteJokeList();
    },
    searchFavoriteJokeList() {
      chrome.storage.local.get("jokes", (res) => {
        this.favoriteJokeList = res.jokes;
        console.log(`favoriteJokeList: ${this.favoriteJokeList}`);
      });
    }
  },
  mounted() {
    axios.get(
            "https://icanhazdadjoke.com/",
            { 'headers': { 'Accept': 'application/json' } }
    )
            .then(res => {
              console.log(`axios request response: \n${JSON.stringify(res, null, 2)}`);
              this.joke = res.data.joke;
              this.loading = false;
            });
    this.searchFavoriteJokeList();
  }
}
</script>

<style>
body {
  height: 98vh;
  text-align: left;
  color: #353638;
  font-size: 22px;
  line-height: 30px;
  font-family: Merriweather,Georgia,serif;
  background: url("https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/12/1544189726troll-dad.png") no-repeat 1% 1%;
  background-size: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}

/*.list {*/
/*  display: flex;*/
/*  flex-direction: column;*/
/*  */
/*}*/

.joke {
  text-align: center;
  max-width: 800px;
}

.button-container {
  position: absolute;
  right: 0px;
  top: calc(50% - 74px);
}


.btn {
  background-color: #D8D8D8;
  border: none;
  color: white;
  padding: 12px 16px;
  font-size: 16px;
  cursor: pointer;
  display: block;
  margin-bottom: 5px;
  width: 50px;
}

.btn:hover {
  background-color: #C8C8C8;
}

.btn:disabled {
  background-color: #909090;
}
</style>
