<!-- 
TODO
Convert useGenerateThread hook to vue ✅
Convert GeneratedThread functional component to vue ✅
  Maybe just scrap the component and write it into App.vue?
Sass style => Bulma ✅
Send generated text and prompt to PostgreSQL database (if there's time)
Remake tweet-gen-api with django (if there's time)
-->
<template>
  <h1 class="title has-text-primary">Tweeter Man</h1>
  <h1 class="subtitle has-text-primary">Vue edition</h1>
  <div
    class="box"
    style="width: 480px; maxWidth: 100%; minHeight: 80%; maxHeight: 90%; overflow: auto;"
  >
    <h1 class="subtitle has-text-primary">Tweet:</h1>
    <div class="block">
      <textarea
        class="textarea is-primary"
        type="text"
        style="resize: none;"
        v-model="tweet"
        placeholder="Initial Tweet"
      />
    </div>
    <div class="block buttons has-addons" style="alignitems: flex-end">
      <button class="button is-danger is-light" @click="clearTweet()">Clear</button>
      <button :class="[loading == true ? 'button is-primary is-loading' : 'button is-primary is-light']" @click="fetchGenText(tweet, numThreads)">
        Submit
      </button>
    </div>
    <h1 class="subtitle has-text-primary">Number of Threads:</h1>
    <div class="block buttons">
      <button
        :class="[isActive(1) ? 'button is-primary' : 'button is-primary is-light']"
        @click="updateNum(1)"
      >
        1
      </button>
      <button
        :class="[isActive(2) ? 'button is-primary' : 'button is-primary is-light']"
        @click="updateNum(2)"
      >
        2
      </button>
      <button
        :class="[isActive(3) ? 'button is-primary' : 'button is-primary is-light']"
        @click="updateNum(3)"
      >
        3
      </button>
      <button
        :class="[isActive(4) ? 'button is-primary' : 'button is-primary is-light']"
        @click="updateNum(4)"
      >
        4
      </button>
    </div>
    <div
      class="block box has-background-primary-light"
      v-for="genThread in displayedThread"
      v-bind:key="genThread.id"
    >
        {{ genThread.text }}
    </div>
  </div>
</template>

<script>
import "./assets/main.scss";

export default {
  name: "App",
  data() {
    return {
      tweet: "",
      numThreads: 1,
      displayedThread: "",
      loading: false,
    };
  },
  methods: {
    removeGenItem(genItemId) {
      this.displayedThread = this.displayedThread.filter(
        (genItem) => genItem.id !== genItemId
      );
    },
    addGenText() {
      if (this.tweet.length === 0) return;
      else {
        this.genText.push({
          id: Math.random(),
          text: this.tweet,
        });
      }
    },
    clearTweet() {
      this.tweet = "";
      this.displayedThread = []
    },
    updateNum(number) {
      this.numThreads = number;
    },
    isActive(num) {
      if (num == this.numThreads) return true;
      else false;
    },

    async fetchGenText(tweet, numThreads, thread = []) {
      
      if (thread.length >= numThreads) {
        this.loading = false;
        return (this.displayedThread = thread);
      }
      this.loading = true
      const prompt =
        "Continue " +
        [
          tweet,
          ...Array.from(Object.values(thread), (threads) => threads.text),
        ].join(" ") +
        "\n";

      const result = await fetch("https://tweeterman.herokuapp.com/tweet/", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ sentPrompt: prompt }),
      }).then((response) => response.json());

      const newThread = {
        id: Math.random(),
        text: result.choices[0].text.trim().substring(0, 280),
      };

      return this.fetchGenText(tweet, numThreads, [...thread, newThread]);
    },
  },
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-content: center;
  justify-content: center;
  height: 100vh;
  align-items: center;
  overflow: hidden;
  background-color: hsl(171, 100%, 96%)
}
</style>
