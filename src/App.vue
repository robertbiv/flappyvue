<!--Robert Bennethum CMPSC 421-->
<!--Couldn't figure out how to make picture link to 3d array, so tried a different way-->
<!--/*Don't publish online, could be problematic because of personal info*/-->

<!--npm run serve-->
<!--installed firebase from npm-->
<!--installed vue from npm-->
<!--https://firebase.google.com/docs/firestore/quickstart-->


<template>
  <div id="app" @click="jump" :style="{ height: windowHeight + 'px' }">
    <figure> <!--Score-->
      <img id="playScore0" src="../flappy_assets/sprites/0.png" style="float: left; margin-left:40%; margin-right: 5%; margin-bottom: 5%;"><img id="playScore" src="../flappy_assets/sprites/0.png" style="float: left; margin-right: 5%; margin-bottom: 5%;">
    </figure>

<!--    <div class="playScore" :style="{ top: playScorey + 'px' }"></div>-->
    <div class="bird" :style="{ top: birdy + 'px' }"></div>
    <div class="pipe" v-for="(pipe, index) in pipes" :key="index" :style="{ left: pipe.left + 'px' }">
      <div class="pipe top" :style="{ height: pipe.topHeight + 'px' }"></div>
      <div class="space" :style="{ height: '150px' }"></div>
      <div class="pipe bottom" :style="{ height: pipe.bottomHeight + 'px' }"></div>
    </div>
  </div>
  <div id="text">
    <h1>Flappy Vue!</h1>
    <h3>Score Board</h3>
    <ul class="collection" id="score_ul"></ul>
  </div>
</template>
<script type="text/javascript">
// Import the functions you need from the SDKs you need

//import { initializeApp } from "firebase/app";
//import firebase from "firebase/compat/app";
//import firebase from 'firebase/app';
import firebase from 'firebase/compat/app';
import 'firebase/compat/firestore';
/*Not all worked but these did */
//import {waitFor} from "@babel/core/lib/gensync-utils/async";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = { /*Don't publish online, could be problematic*/
  apiKey: "AIzaSyACAPoGV__CZog6MfD2WgTYvgAjTHE9vgE",
  authDomain: "cmpsc421-flappyvue.firebaseapp.com",
  projectId: "cmpsc421-flappyvue",
  storageBucket: "cmpsc421-flappyvue.appspot.com",
  messagingSenderId: "194566899644",
  appId: "1:194566899644:web:ab0ec33c4b386ca9738db9"
};

// Initialize Firebase
//const app = initializeApp(firebaseConfig);
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
db.settings({})
window.addEventListener("load", async function () { //wait for dom to load then load score board
  // do things after the DOM loads fully
  console.log("Everything is loaded");
  const ul = document.getElementById("score_ul");
  console.log(ul);
  const citiesRef = db.collection('highScores');
  const snapshot = await citiesRef.orderBy('score','desc').get();
  snapshot.forEach(doc => {
    console.log(doc.id, '=>', doc.data());
    console.log(doc.data().score);
    let li = document.createElement('li');
    li.innerText = doc.data().playerName + ": " + doc.data().score;
    ul.appendChild(li);
  });
});

/*db.collection('highScores') //attempt 1 at calling db, not working, keep for reference from car example
    .orderBy('score', 'desc')
    .limit(10) // Get top 10 scores
    .get()
    .then(querySnapshot => {
      querySnapshot.forEach(doc => {
        console.log(doc.data()); // Handle each high score
        let li = document.createElement('li');
        li.className = "collection-item"
        let span = document.createElement('span');
        span.className = 'title';
        span.textContent = doc.data().playerName;

        // <p className="grey-text">2023</p>
        let p = document.createElement('p');
        p.className = 'grey-text';
        // p.textContent = '2023'
        p.textContent = doc.data().score;
        li.append(span)
        li.append(p)
        ul.append(li);
      });
    });*/

export default {
  data() {
    return { //make variables default
      birdy: 220,
      windowHeight: 450,
      pipes: [],
      timer: null,
      pipeTimer: null,
      score: 0,
    };
  },
  methods: {
    jump() {
      this.birdy -= 90;
    },
    checkLose() {
      for (let pipe of this.pipes) {
        if ( //check if inbetween pipes
            (this.birdy <= pipe.topHeight || this.birdy >= this.windowHeight - pipe.bottomHeight) &&
            pipe.left <= 80 && pipe.left >= 60
        ) {
          this.stop();
        }
        else if (pipe.left === 60){ //increase score if passed pipe
          this.score+=1;
          console.log(this.score);
          const playScore = document.getElementById("playScore"); //update score at top
          if (this.score <= 9){ /*calling a file wasn't working, so pull from GitHub*/
            playScore.src="https://github.com/samuelcust/flappy-bird-assets/blob/master/sprites/"+this.score+".png?raw=true";
          }
          else {
            const playScore0 = document.getElementById("playScore0"); /*calling a file wasn't working, so pull from GitHub*/
            playScore0.src="https://github.com/samuelcust/flappy-bird-assets/blob/master/sprites/"+String(this.score)[0]+".png?raw=true";
            playScore.src="https://github.com/samuelcust/flappy-bird-assets/blob/master/sprites/"+String(this.score)[1]+".png?raw=true";
          }


        }
      }
    },
    run() { /*start clocks*/
      if (this.timer) return;
      this.timer = setInterval(() => {
        this.birdy += 2;
        this.pipes.forEach((pipe) => { //shift pipe
          pipe.left -= 2;
        });
        this.checkLose();
      }, 20);
      this.pipeTimer = setInterval(this.newPipe, 2500); /*Spawn Rate*/
    },
    randomtopHeight() {
      const howRandom = 222;
      return Math.floor(Math.random() * howRandom);
    },
    newPipe() {
      const gapSize = 150;
      const topHeight = this.randomtopHeight();
      const bottomHeight = this.windowHeight - gapSize - topHeight;
      const pipe = { topHeight, bottomHeight, left: 450 }; /*start at 450 px when screen width is 400*/
      this.pipes.push(pipe);
    },
    stop() {
      /*Stop run by stop clocks*/
      clearInterval(this.timer);
      clearInterval(this.pipeTimer);
      db.collection('highScores').add({ //add score to db
        playerName: 'Anon Player', /*maybe in the future add name, placeholder for now*/
        score: this.score,
      });
    },
  },
  mounted() { /*starter*/
    this.run();
  },
};
</script>

<!--Make pretty and make pipes work-->
<style>
#text { /*make text readable*/
  background: azure;
}
#app {
  width: 400px;
  position: relative;
  background-image: url("../flappy_assets/sprites/background-day.png");
  overflow: hidden;  /*hide offscreen stuff*/
}
.bird {
  width: 34px;
  height: 24px;
  background-image: url("../flappy_assets/sprites/redbird-midflap.png");
  position: absolute;
  left: 80px; /*Shift Right from left side*/
}
/*.playScore {
  position: absolute;
  left: 450px;
}*/
.pipe {
  width: 50px;
  position: absolute;
}
.pipe.top {
  position: relative;
  background-image: url("../flappy_assets/sprites/pipe-green.png");
  transform: rotate(180deg);
}
.space { /*Dont know why this isn't showing in its color, but if it ain't broke dont fix it*/
  position: relative;
  color: aliceblue;
}
.pipe.bottom {
  position: absolute;
  background-image: url("../flappy_assets/sprites/pipe-green.png");
}
</style>