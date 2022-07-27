<!-- 
TODO
Convert useGenerateThread hook to vue ✅
Convert GeneratedThread functional component to vue ✅
  Maybe just scrap the component and write it into App.vue?
Sass style => Bulma
Send generated text and prompt to PostgreSQL database (if there's time)
Remake tweet-gen-api with django (if there's time)
-->

<template>
  <div
    class="block box"
    style="width: 480px; minHeight: 80%; maxHeight: 90%; overflow: scroll"
  >
    <h1 class="subtitle">Tweet:</h1>
    <div class="block">
      <textarea
        class="textarea is-primary"
        type="text"
        v-model="tweet"
        placeholder="Initial Tweet"
      />
    </div>
    <div class="block buttons has-addons" style="alignitems: flex-end">
      <button class="button" @click="clearTweet()">Clear</button>
      <button class="button" @click="fetchGenText(tweet, numThreads)">
        Submit
      </button>
    </div>
    <h1 class="subtitle">Number of Threads:</h1>
    <div class="block buttons">
      <button
        :class="[isActive(1) ? 'button is-black' : 'button']"
        @click="updateNum(1)"
      >
        1
      </button>
      <button
        :class="[isActive(2) ? 'button is-black' : 'button']"
        @click="updateNum(2)"
      >
        2
      </button>
      <button
        :class="[isActive(3) ? 'button is-black' : 'button']"
        @click="updateNum(3)"
      >
        3
      </button>
      <button
        :class="[isActive(4) ? 'button is-black' : 'button']"
        @click="updateNum(4)"
      >
        4
      </button>
    </div>
    <div
      class="block"
      v-for="genThread in displayedThread"
      v-bind:key="genThread.id"
    >
      <div class="block box">
        {{ genThread.text }}
      </div>
      <button class="button" @click="removeGenItem(genThread.id)">❌</button>
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
      numThreads: 2,
      displayedThread: "",
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
        console.log(thread);
        return (this.displayedThread = thread);
      }

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
  align-content: center;
  justify-content: center;
  height: 100vh;
  align-items: center;
}
</style>
