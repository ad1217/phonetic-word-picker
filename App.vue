<template>
  <div>
    <input v-model="input" pattern="[ 0-9]+" />
    <div class="matches">
      <div class="match-list" v-for="num in input_numbers">
        <div
          class="match"
          v-for="match in matches(num)"
          @click="bad.unshift(match)"
        >
          {{ match }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import pronouncing from "pronouncing";

import sounds from "./sounds.yaml";
let regex_sounds = {};
Object.keys(sounds).forEach(k => (regex_sounds[k] = sounds[k]));
// parse to regex
regex_sounds.VOWELS = regex_sounds.VOWELS.flatMap(s => [
  s + "0",
  s + "1",
  s + "2"
]);
regex_sounds.JUNK = [...regex_sounds.VOWELS, ...regex_sounds.NOP, " "];
Object.keys(regex_sounds).forEach(
  k => (regex_sounds[k] = "(" + regex_sounds[k].join("|") + ")")
);

export default {
  name: "App",
  data() {
    return {
      input: "",
      bad: []
    };
  },

  mounted() {
    if (localStorage["wordpicker-bad"]) {
      this.bad = JSON.parse(localStorage["wordpicker-bad"]);
    }
  },

  watch: {
    bad() {
      localStorage["wordpicker-bad"] = JSON.stringify(this.bad);
    }
  },

  computed: {
    input_numbers() {
      return this.input
        .split(" ")
        .map(num => num.split("").map(digit => regex_sounds[digit]));
    }
  },

  methods: {
    matches(symbols) {
      const junk = regex_sounds.JUNK + "*";
      let regex = `^${junk}${symbols.join(junk)}${junk}$`;
      return Array.from(new Set(pronouncing.search(regex))).filter(
        x => !this.bad.includes(x)
      );
    }
  }
};
</script>

<style>
.matches {
  display: flex;
  flex-direction: row;
}

.match-list {
  margin: 1em;
}

.match {
  background-color: #aaa;
  border-radius: 0.1em;
  margin: 0.25em;
  padding: 0.3em;
  cursor: pointer;
}
</style>
