<template>
  <div>
    <label>
      Number (space to separate words):
      <input v-model="input" pattern="[ 0-9]+" />
    </label>
    <div>
      Show words of type:
      <label class="match" :class="cat" v-for="cat in categories">
        <input type="checkbox" :value="cat" v-model="enabled_categories" />
        {{ cat }}
      </label>
    </div>
    <div class="matches">
      <div class="match-list" v-for="num in input_numbers">
        <div
          class="match"
          v-for="match in matches(num)"
          @click="bad.unshift(match)"
          :class="category(match)"
          v-if="enabled_categories.includes(category(match))"
        >
          {{ match }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import pronouncing from "pronouncing";
import categorized_words from "categorized-words";

import sounds from "./sounds.yaml";
let regex_sounds = {};
Object.keys(sounds).forEach(k => (regex_sounds[k] = sounds[k]));
// parse to regex
regex_sounds.VOWELS = regex_sounds.VOWELS.flatMap(
  s => [...new Array(3).keys()].map(k => s + k) // append 0, 1, 2
);
regex_sounds.JUNK = [...regex_sounds.VOWELS, ...regex_sounds.NOP, " "];
Object.keys(regex_sounds).forEach(
  k => (regex_sounds[k] = "(" + regex_sounds[k].join("|") + ")")
);

export default {
  name: "App",
  data() {
    return {
      input: "",
      bad: [],
      categories: ["noun", "verb", "adjective", "other"],
      enabled_categories: ["noun", "verb", "adjective"]
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
    },

    category(word) {
      const wordlist = categorized_words;
      return wordlist.N.includes(word)
        ? "noun"
        : wordlist.V.includes(word)
        ? "verb"
        : wordlist.A.includes(word)
        ? "adjective"
        : "other";
    }
  }
};
</script>

<style lang="scss">
.matches {
  display: flex;
  flex-direction: row;
}

.match-list {
  margin: 0.5em;
}

.match {
  background-color: #aaa;
  border-radius: 0.1em;
  margin: 0.25em;
  padding: 0.3em;
  cursor: pointer;

  &.noun {
    background-color: #b95f5f;
  }

  &.verb {
    background-color: #5f5fb9;
  }

  &.adjective {
    background-color: #5fb95f;
  }
}
</style>
