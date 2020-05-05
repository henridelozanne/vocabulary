<template>
  <div id="app">
    <div class="choose-topic" v-if="!chosenTopic">
      <h1>Choisir un thème</h1>
      <ul class="topic-list">
        <li v-for="topic in topics" :key="topic"
            @click="chosenTopic = topic">{{ topic }}</li>
      </ul>
    </div>
    <div v-else-if="onListActions">
      <h1>{{ chosenTopic }}</h1>
      <div class="spans-ctn">
        <span class="action" @click="launchTest">Lancer le test</span>
        <span class="action" @click="checkList">Voir liste</span>
        <span class="action red" @click="chosenTopic = undefined">Revenir</span>
      </div>
    </div>
    <div v-else class="test-actions">
      <div v-if="testIsLaunched" class="test-vue">
        <h1>Test: {{ chosenTopic }}</h1>
        <h3 @click="escapeTest" class="escape-test action red">Revenir</h3>
        <div class="question-ctn">
          <div>{{ currentQuestion.index }}) {{ currentQuestion.es }}</div>
          <input autofocus type="text" v-model="answer">
          <button @click="checkAnswer">Envoyer</button>
        </div>
        <div class="score">Score: {{score}}/{{currentQuestion.index - 1}}</div>
        <div class="answer-feedback" :class="feedbackClass">{{ answerFeedback }}</div>
      </div>
      <div v-else-if="listIsVisible">
        <h1>Liste: {{ chosenTopic }}</h1>
        <h3 @click="listIsVisible = false" class="escape-test action red">Revenir</h3>
        <ul class="words-list">
          <li v-for="word in currentWords" :key="word.es">
            {{ word.es }} : {{ word.fr }}
          </li>
        </ul>
      </div>
      <div v-if="recapIsVisible" class="modal">
        <div class="modal-content">
          <h1>Résumé</h1>
          <div>Score: {{ score }} / {{ currentQuestion.index }}</div>
          <div @click="closeRecap" class="quit-recap-btn action red">Quitter</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import spaceWords from './assets/spaceWords.js';

export default {
  name: 'App',
  created() {
    const that = this;
    document.addEventListener('keypress', function (e) {
      if (e.key === 'Enter') {
        if (that.testIsLaunched) {
          that.checkAnswer();
        }
      }
    });
  },
  computed: {
    topics() {
      const topics = [];
      this.vocLists.forEach((list) => {
        topics.push(list.listName);
      });
      return topics;
    },
    onListActions() {
      return this.chosenTopic && !this.testIsLaunched && !this.listIsVisible;
    },
    currentWords() {
      return this.vocLists.filter((list) => list.listName === this.chosenTopic)[0].words;
    },
  },
  methods: {
    checkList() {
      this.listIsVisible = true;
    },
    checkAnswer() {
      if (this.currentQuestion.fr === this.answer || (this.currentQuestion.fr2 && this.currentQuestion.fr2 === this.answer)) {
        this.answerFeedback = 'Correct';
        this.feedbackClass = 'green';
        setTimeout(() => {
          this.answerFeedback = undefined;
        }, 1400);
        this.score += 1;
      } else {
        this.answerFeedback = 'Incorrect';
        this.feedbackClass = 'red';
        setTimeout(() => {
          this.answerFeedback = undefined;
        }, 1400);
      }
      this.answer = undefined;
      if (this.currentQuestion.index + 1 <= this.shuffledList.length) {
        this.newTurn();
      } else {
        this.showRecap();
      }
    },
    escapeTest() {
      this.testIsLaunched = false;
      this.reinitTest();
    },
    showRecap() {
      this.recapIsVisible = true;
    },
    reinitTest() {
      this.testIsLaunched = false;
      this.score = 0;
      this.currentQuestion = {
        index: undefined,
        es: undefined,
        fr: undefined,
      };
    },
    closeRecap() {
      this.recapIsVisible = false;
      this.chosenTopic = undefined;
      this.reinitTest();
    },
    launchTest() {
      this.testIsLaunched = true;
      this.shuffledList = this.shuffle(this.vocLists.filter((list) => list.listName === this.chosenTopic)[0].words);
      this.newTurn();
    },
    newTurn() {
      const index = !this.currentQuestion.index ? 1 : this.currentQuestion.index + 1;
      this.currentQuestion = {
        index,
        es: this.shuffledList[index - 1].es,
        fr: this.shuffledList[index - 1].fr,
        fr2: this.shuffledList[index - 1].fr2 || undefined,
      }
    },
    shuffle(array) {
      let currentIndex = array.length;
      let temporaryValue;
      let randomIndex;
      while (0 !== currentIndex) {
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;
        temporaryValue = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temporaryValue;
      }
      return array;
    },
  },
  data() {
    return {
      answer: undefined,
      answerFeedback: undefined,
      score: 0,
      chosenTopic: undefined,
      shuffledList: undefined,
      testIsLaunched: false,
      recapIsVisible: false,
      feedbackClass: undefined,
      currentQuestion: {
        index: undefined,
        es: undefined,
        fr: undefined,
        fr2: undefined,
      },
      listIsVisible: false,
      vocLists: [
        {
          listName: 'Corps humain',
          words: [
            {
              fr: 'jambe',
              es: 'pierna',
            },
            {
              fr: 'bras',
              es: 'brazo',
            },
            {
              fr: 'pied',
              es: 'pie',
            },
          ],
        },
        {
          listName: 'Espace',
          words: spaceWords,
        },
        {
          listName: 'Vêtements',
          words: [
            {
              fr: 'chemise',
              es: 'camisa',
            },
            {
              fr: 'chaussures',
              es: 'zapatos',
            },
          ],
        },
      ],
    };
  },
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
}

html {
  height: 100vh;
}

body {
  height: 100%;
  margin: 60px 0 0;
}

.test-actions {
  height: 100%;

  .test-vue {
    height: 100%;
    display: flex;
    flex-direction: column;

    >div {
      height: 100%;
    }
  }
}

ul {
  padding: 0;
}

.topic-list {
  display: flex;
  justify-content: space-around;
  list-style: none;
  margin-top: 100px;

  li {
    font-size: 20px;
    cursor: pointer;
    text-transform: capitalize;
  }
}

.score {
  position: absolute;
  top: 85px;
  right: 195px;
  font-size: 20px;
}

.spans-ctn {
  display: flex;
  justify-content: space-around;
  margin-top: 60px;

  span {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    background: white;
    font-size: 20px;
    cursor: pointer;
  }
}

.escape-test {
  cursor: pointer;
}

.words-list {
  list-style: none;
}

.green {
  color: rgb(46, 182, 46);
}

.red {
  color: rgb(227, 30, 30) !important;
}

.question-ctn {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 auto 10px;
  font-size: 30px;
  flex-grow: 1;
  
  input {
    margin: 0 20px;
    font-size: 30px;
    width: 200px;
    padding: 0 10px;
    border-top: none;
    border-left: none;
    border-right: none;
    font-family: Avenir, Helvetica, Arial, sans-serif;
    color: #2c3e50;
    height: 40px;

    &:focus {
      outline-width: 0;
    }
  }

  button {
    padding: 10px 15px;
    font-size: 15px;
    font-family: Avenir, Helvetica, Arial, sans-serif;
    color: #2c3e50;
    height: 40px;
    cursor: pointer;
  }
}

.answer-feedback {
  margin-top: -150px;
  font-size: 20px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: center;
  align-items: center;

  .modal-content {
    width: 500px;
    padding: 100px;
    background: white;
    margin: auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
}

.quit-recap-btn {
  margin-top: 50px;
  cursor: pointer;
}

.action {
  color: rgb(0, 136, 255);
  font-weight: 700;
  font-size: 1.2em;
}
</style>
