<template>
  <body>
    <dl>
      <div
      v-for="(w,widx) in words"
      :key="widx">
        <div
        v-for="(s,idx) in slogs(w)"
        :key="idx"
        style="display:inline">
          <p style="display:inline"
          :style="{'font-size': widx==0 ? '42px' : '32px'}">
            {{s[0]}}
          </p>
          <button v-if="s[1]"
          @click.stop="widx==0 && selectLetter(widx,idx)"
          :style="{'font-size': widx==0 ? '42px' : '32px'}"
          :class="{ 'right':idx===w.accent && w.answer!=='',
                    'wrong':(idx!==w.accent) && idx===w.answer,
                    'button':w.answer===''}">
            {{s[1]}}
          </button>
        </div>
      </div>
    </dl>
  </body>
</template>

<script>
export default {
  data(){
    return{
      words:[],
      listWords:[]
    }
  },
  methods:{
    addWord(){
      this.words.unshift({...this.listWords[Math.floor(Math.random()*this.listWords.length)]})
    },
    selectLetter(w,v){
      this.words[w].answer=v
      setTimeout(() => {
        this.addWord()
        if (this.words.length>100){
          this.words.pop()
        }
      }, 400);
    },
    slogs(w){
      let slgs=[]
      for(let i=0;i<w.vowels.length;i++){
        slgs.push([w.word.slice(i>0?w.vowels[i-1]+1:0,w.vowels[i]),w.word[w.vowels[i]]])
      }
      return slgs
    }
  },
  async created(){
    let response = await fetch('./accents.txt')
    let text = (await response.text()).split('\r\n')
    let vows,acc
    const VOWELS='уеыаоэяиюё'
    for(let i=0;i<text.length;i++) {
      vows=[]
      for(let j=0;j<(text[i].indexOf('(')==-1?text[i].length:text[i].indexOf('('));j++){
        if(VOWELS.indexOf(text[i][j].toLowerCase())!=-1){
          vows.push(j)
          if(text[i][j]!==text[i][j].toLowerCase()){
            acc=j
          }
        }
      }
      vows.push(text[i].length)
      this.listWords.push({word:text[i].toLowerCase(),vowels:vows,accent:vows.indexOf(acc),answer:''})
    }
    this.addWord()
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
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  font-size: 32pt;
}
button {
  border: none;
  border-radius: 8px;
  color: #2c3e50;
  padding: 0px 2px;
  line-height: 100%;
  display: inline;
}
.button {
  transition-duration: 0.4s;
}
.button:hover {
  box-shadow: 0 12px 16px 0 rgba(0,0,0,0.24), 0 17px 50px 0 rgba(0,0,0,0.19);
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
