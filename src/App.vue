<template>
  <body>
    <span style="float: left;font-size: 20px">{{ rightCount+' / '+count }}</span>
    <input type="checkbox"
    style="float: right;scale: 150%;margin-top: 6px;"
    v-model="onlyMis">
    <span style="float: right;font-size: 20px">Только ошибки:&nbsp;</span>
    <br>
    <transition-group
    name="list" tag="div">
      <div
      v-for="(w,widx) in getWords()"
      :key="w.id"
      :data-index="widx">
        <div
        v-for="(s,idx) in slogs(w)"
        :key="idx"
        style="display:inline">
          <p style="display:inline"
          :style="{'font-size': widx==0 ? '42px' : '32px'}">
            {{s[0]}}
          </p>
          <button v-if="s[1]"
          @click="widx==0 && selectLetter(idx)"
          :style="{'font-size': widx==0 ? '42px' : '32px'}"
          :class="{ 'right':idx===w.accent && w.answer!=='',
                    'wrong':(idx!==w.accent) && idx===w.answer,
                    'button':w.answer===''}">
            {{s[1]}}
          </button>
        </div>
      </div>
    </transition-group>
  </body>
</template>

<script>
export default {
    data() {
        return {
            words: [],
            listWords: [],
            wrong: [],
            rightCount: 0,
            count: 0,
            onlyMis: false
        };
    },
    methods: {
        getWords() {
            return this.words.filter((w,idx) => !this.onlyMis || idx==0 || w.accent!==w.answer);
        },
        addWord() {
            if (Math.random() < this.listWords.length / (this.listWords.length + 5 * this.wrong.length)) {
                this.words.unshift({ ...this.listWords[Math.floor(Math.random() * this.listWords.length)] });
            }
            else {
                this.words.unshift({ ...this.listWords[this.wrong[Math.floor(Math.random() * this.wrong.length)]] });
            }
        },
        selectLetter(v) {
            this.words[0].answer = v;
            this.count++;
            if (v !== this.words[0].accent) {
                this.wrong.push(this.words[0].id);
            }
            else {
                this.wrong = this.wrong.filter((t) => t != this.words[0].id);
                this.rightCount++;
            }
            setTimeout(() => {
                this.addWord();
                if (this.words.length > 100) {
                    this.words.pop();
                }
            }, 300);
        },
        slogs(w) {
            let slgs = [];
            for (let i = 0; i < w.vowels.length; i++) {
                slgs.push([w.word.slice(i > 0 ? w.vowels[i - 1] + 1 : 0, w.vowels[i]), w.word[w.vowels[i]]]);
            }
            return slgs;
        }
    },
    async created() {
        let response = await fetch("./accents.txt");
        let text = (await response.text()).split("\n");
        let vows, acc;
        const VOWELS = "уеыаоэяиюё";
        for (let i = 0; i < text.length; i++) {
            vows = [];
            for (let j = 0; j < (text[i].indexOf("(") == -1 ? text[i].length : text[i].indexOf("(")); j++) {
                if (VOWELS.indexOf(text[i][j].toLowerCase()) != -1) {
                    vows.push(j);
                    if (text[i][j] !== text[i][j].toLowerCase()) {
                        acc = j;
                    }
                }
            }
            vows.push(text[i].length);
            this.listWords.push({ id: i, word: text[i].toLowerCase(), vowels: vows, accent: vows.indexOf(acc), answer: "" });
        }
        this.addWord();
    }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
  font-size: 32pt;
}
.list-move,
.list-enter-active{
  transition: all 0.5s ease;
}
.list-leave-active {
  transition: all 0.5s ease;
  position:absolute;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: scale(0.5);
}
.list-leave-from,
.list-enter-to {
  opacity: 1;
  transform: scale(1);
}
button {
  border: none;
  border-radius: 8px;
  color: #2c3e50;
  padding: 0px 2px;
  line-height: 100%;
  display: inline;
  box-shadow: 0 3px 2px 0 rgba(0,0,0,0.24);
}
.button {
  transition-duration: 0.4s;
}
.button:hover {
  box-shadow: 0 3px 10px 0 rgba(0,0,0,0.24);
}
.right {
  color: white;
  background-color: limegreen;
}
.wrong {
  color: white;
  background-color: crimson;
}
</style>
