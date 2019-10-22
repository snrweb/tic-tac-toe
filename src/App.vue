<template>
  <div id="app-container">
    <div class="game-alert">{{gameAlert}}</div>
    <div class="symbol-option" v-if="!gameStarted">
      <span>Choose:</span>
      <img v-bind:src="circle" alt="A circle image" @click="selectSymbol($event, 'circle')" />
      <img v-bind:src="cross" alt="A cross image" @click="selectSymbol($event, 'cross')" />
    </div>
    <div class="app-body">
      <div class="cell-1 cell" id="cell-1" @click="userIsPlaying"></div>
      <div class="cell-2 cell" id="cell-2" @click="userIsPlaying"></div>
      <div class="cell-3 cell" id="cell-3" @click="userIsPlaying"></div>

      <div class="cell-4 cell" id="cell-4" @click="userIsPlaying"></div>
      <div class="cell-5 cell" id="cell-5" @click="userIsPlaying"></div>
      <div class="cell-6 cell" id="cell-6" @click="userIsPlaying"></div>

      <div class="cell-7 cell" id="cell-7" @click="userIsPlaying"></div>
      <div class="cell-8 cell" id="cell-8" @click="userIsPlaying"></div>
      <div class="cell-9 cell" id="cell-9" @click="userIsPlaying"></div>
    </div>

    <div class="game-score">
      <p class="player-wins">
        <img v-if="player == 'circle'" v-bind:src="circle" alt="A circle image" />
        <img v-if="player == 'cross'" v-bind:src="cross" alt="A cross image" />
        You: {{playerScore}}
      </p>
      <p class="player-losses">
        <img v-if="computer == 'circle'" v-bind:src="circle" alt="A circle image" />
        <img v-if="computer == 'cross'" v-bind:src="cross" alt="A cross image" />
        Computer: {{computerScore}}
      </p>
    </div>

    <div class="game-control-btn">
      <button class="new-game-btn" @click="newGame" v-if="!gameStarted">New Game</button>
      <button class="cancel-game-btn" v-if="gameStarted" @click="giveUp">Give Up</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      root: "https://localhost/naijadate/aj",

      playerScore: 0,
      computerScore: 0,
      gameAlert: "",
      gameStarted: false,
      gameOver: true,
      computerHasPlayed: false,

      firstPlayer: "computer",
      tacticChoice: [],
      tacticSelected: false,

      player: "",
      computer: "",

      cross: require("./assets/cross.png"),
      circle: require("./assets/circle.png"),
      greenCross: require("./assets/cross_green.png"),
      greenCircle: require("./assets/circle_green.png"),
      redCross: require("./assets/cross_red.png"),
      redCircle: require("./assets/circle_red.png"),

      allBoxes: [],
      pTrack: [],
      unChangedPossibleWins: [],
      elemAry: [],
      compTrackForWin: [],

      tacticAry: []
    };
  },

  methods: {
    /**
     * starts new game.
     * The computer starts playing if the a new game has started and
     * was selected to start first.
     */
    startGame() {
      if (this.gameStarted && this.firstPlayer == "computer") {
        this.computerIsPlaying();
      }
    },

    giveUp() {
      this.alertWinner("computer");
      this.computerScore++;
      this.newGame();
    },

    userIsPlaying(event) {
      /**
       * Get the id attribute of the clicked div element/box,
       * split and get the box number.
       *
       * The box number is used to check if the box has been played
       * or still empty
       *
       * If still empty, an image symbol is inserted else a notification
       * is generated.
       */
      if (this.gameOver) return;

      let attr = event.target.getAttribute("id");
      if (attr == null) {
        //checks if the target elem is the parent elem
        attr = event.target.parentElement.getAttribute("id");
      }

      let userLastPlay = parseInt(attr.split("-")[1]); //the clicked box number

      if (this.allBoxes.indexOf(userLastPlay) >= 0) {
        if (this.player == "cross") {
          event.target.innerHTML = "<img src=" + this.cross + " />";
        } else {
          event.target.innerHTML = "<img src=" + this.circle + " />";
        }

        this.allBoxes.splice(this.allBoxes.indexOf(userLastPlay), 1);
        this.removePlayedBoxByComp(userLastPlay, "player");
        this.pTrack.splice(userLastPlay - 1, 1, "player");
        this.computerHasPlayed = false;
        if (!this.gameOver) this.checkWinner();
        this.removePlayedBoxByUser(userLastPlay);

        this.computerIsPlaying();
        if (!this.gameOver) this.checkWinner();
      } else {
        alert("Select another box");
      }

      /**
       * prompt for a new game is shown if all the boxes
       * has been selected and there are no winners
       **/
      if (this.allBoxes.length === 0 && !this.gameOver) {
        if (confirm("There was no winner, new Game?")) this.newGame();
      }
    },

    computerIsPlaying() {
      if (this.gameOver) return;

      this.selectTactic(); //A tactic is selected for the computer
      let elemLen = this.elemAry.length;

      this.checkForPossibleCompWin();

      /**
       * searches for patterns where the player is one step away from wining.
       * If there's one, the box is played by the computer i.e the computer prevents the player from wining.
       * and the array containing the
       * box is removed from the elemAry array.
       */
      if (this.gameOver) return;

      for (let i = 0; i < elemLen; i++) {
        if (this.elemAry[i].length == 1) {
          if (this.allBoxes.indexOf(this.elemAry[i][0]) < 0) {
            this.elemAry.splice(i, 1); //if the box is not empty
            this.computerIsPlaying();
          } else {
            //if the box is empty
            let selectedBoxNum = this.elemAry[i][0];
            this.pTrack.splice(selectedBoxNum - 1, 1, "computer");
            let elem = document.querySelector(".cell-" + selectedBoxNum);
            this.checkAndInsertComputerSymbol(elem);
            this.removePlayedBoxByComp(selectedBoxNum, "computer");
            this.allBoxes.splice(this.allBoxes.indexOf(selectedBoxNum), 1);
            this.computerHasPlayed = true;
          }
          break;
        }
      }

      if (!this.computerHasPlayed) {
        this.tactic();
        this.computerHasPlayed = true;
      }
    },

    checkForPossibleCompWin() {
      let len = this.compTrackForWin.length;

      for (let i = 0; i < len; i++) {
        if (
          this.compTrackForWin[i].length == 1 &&
          this.compTrackForWin[i][0] != "player"
        ) {
          let elem = document.querySelector(
            ".cell-" + this.compTrackForWin[i][0]
          );
          this.checkAndInsertComputerSymbol(elem);
          this.computerHasPlayed = true;
          this.alertWinner("computer", this.unChangedPossibleWins[i]);
          break;
        }
      }
    },

    tactic() {
      if (this.tacticChoice !== undefined) {
        if (this.tacticAry.length !== 0 && this.tacticChoice.length !== 0) {
          let boxNumRand = Math.floor(Math.random() * this.tacticChoice.length);
          let selectedBoxNum = this.tacticChoice[boxNumRand];

          let elem = document.querySelector(".cell-" + selectedBoxNum);

          if (elem.innerHTML == "") {
            this.pTrack.splice(selectedBoxNum - 1, 1, "computer");
            this.checkAndInsertComputerSymbol(elem);
            this.removePlayedBoxByComp(selectedBoxNum, "computer");
            this.allBoxes.splice(this.allBoxes.indexOf(selectedBoxNum), 1);
            this.tacticChoice.splice(boxNumRand, 1);
          } else {
            this.tacticSelected = false;
            for (let i = 0; i < this.tacticAry.length; i++) {
              if (this.tacticAry[i].indexOf(selectedBoxNum) >= 0) {
                this.tacticAry.splice(i, 1);
              }
            }

            this.selectTactic();
            this.tactic();
          }
        } else {
          if (this.allBoxes.length !== 0) {
            let elem = document.querySelector(".cell-" + this.allBoxes[0]);
            this.pTrack.splice(this.allBoxes[0] - 1, 1, "computer");
            this.checkAndInsertComputerSymbol(elem);
            this.removePlayedBoxByComp(this.allBoxes[0], "computer");
            this.allBoxes.splice(this.allBoxes.indexOf(this.allBoxes[0]), 1);
          }
        }
      } else {
        if (this.allBoxes.length !== 0) {
          let elem = document.querySelector(".cell-" + this.allBoxes[0]);
          this.pTrack.splice(this.allBoxes[0] - 1, 1, "computer");
          this.checkAndInsertComputerSymbol(elem);
          this.removePlayedBoxByComp(this.allBoxes[0], "computer");
          this.allBoxes.splice(this.allBoxes.indexOf(this.allBoxes[0]), 1);
        }
      }
    },

    /**
     * Randomly select a tactic for the computer,
     * registers/stores the tactic and
     * delete the tactics from the tacticAry variable
     */
    selectTactic() {
      if (this.tacticSelected) return;

      let randNum = Math.floor(Math.random() * this.tacticAry.length);

      this.tacticChoice = this.tacticAry[randNum];
      this.tacticSelected = true;
      this.tacticAry.splice(randNum, 1);
    },

    //inserts the image symbol into selected/chosen div/box
    checkAndInsertComputerSymbol(elem) {
      if (this.computer == "circle") {
        elem.innerHTML = "<img src=" + this.circle + " />";
      } else {
        elem.innerHTML = "<img src=" + this.cross + " />";
      }
    },

    /**
     * it removes every array that contains the box number
     * played by the player
     */
    removePlayedBoxByUser(userLastPlay) {
      let firstLayerAry = this.elemAry.map(fa => {
        let secondLayerAry = fa.map((sa, i) => {
          if (sa == userLastPlay) {
            fa.splice(i, 1);
          }
        });
        return secondLayerAry;
      });
    },

    /**
     * it removes every array that contains the box number
     * played by the computer
     */
    removePlayedBoxByComp(userLastPlay, player) {
      let firstLayerAry = this.compTrackForWin.map(fa => {
        let secondLayerAry = fa.map((sa, i) => {
          if (sa == userLastPlay && player == "player") {
            fa.splice(i, 1, "player");
          } else if (sa == userLastPlay && player == "computer") {
            fa.splice(i, 1);
          }
        });
        return secondLayerAry;
      });
    },

    //sets which player plays first
    setFirstPlayer() {
      this.firstPlayer == "computer"
        ? (this.firstPlayer = "player")
        : (this.firstPlayer = "computer");
    },

    /**
     * the player chooses a symbol and the other symbol is
     * selected fo the computer
     */
    selectSymbol(e, type) {
      if (type == "circle") {
        this.player = "circle";
        this.computer = "cross";
      } else {
        this.player = "cross";
        this.computer = "circle";
      }
    },

    //removes all symbols or images from all divs/boxes
    clearSymbolBox() {
      const elemLength = document.getElementsByClassName("cell").length;
      for (let i = 0; i < elemLength; i++) {
        document.querySelectorAll(".cell")[i].innerHTML = "";
      }
    },

    checkWinner() {
      if (this.allBoxes.length > 6) return;

      //check for column one
      if (
        this.pTrack[0] != "" &&
        this.pTrack[0] == this.pTrack[3] &&
        this.pTrack[0] == this.pTrack[6]
      ) {
        this.alertWinner(this.pTrack[0], [1, 4, 7]);
      }

      //check for column two
      if (
        this.pTrack[1] != "" &&
        this.pTrack[1] == this.pTrack[4] &&
        this.pTrack[1] == this.pTrack[7]
      ) {
        this.alertWinner(this.pTrack[1], [2, 5, 8]);
      }

      //check for column three
      if (
        this.pTrack[2] != "" &&
        this.pTrack[2] == this.pTrack[5] &&
        this.pTrack[2] == this.pTrack[8]
      ) {
        this.alertWinner(this.pTrack[0], [3, 6, 9]);
      }

      //check for row one
      if (
        this.pTrack[0] != "" &&
        this.pTrack[0] == this.pTrack[1] &&
        this.pTrack[0] == this.pTrack[2]
      ) {
        this.alertWinner(this.pTrack[0], [1, 2, 3]);
      }

      //check for row two
      if (
        this.pTrack[3] != "" &&
        this.pTrack[3] == this.pTrack[4] &&
        this.pTrack[3] == this.pTrack[5]
      ) {
        this.alertWinner(this.pTrack[3], [4, 5, 6]);
      }

      //check for row three
      if (
        this.pTrack[6] != "" &&
        this.pTrack[6] == this.pTrack[7] &&
        this.pTrack[6] == this.pTrack[8]
      ) {
        this.alertWinner(this.pTrack[6], [7, 8, 9]);
      }

      //check for diagonal one
      if (
        this.pTrack[0] != "" &&
        this.pTrack[0] == this.pTrack[4] &&
        this.pTrack[0] == this.pTrack[8]
      ) {
        this.alertWinner(this.pTrack[0], [1, 5, 9]);
      }

      //check for diagonal two
      if (
        this.pTrack[2] != "" &&
        this.pTrack[2] == this.pTrack[4] &&
        this.pTrack[2] == this.pTrack[6]
      ) {
        this.alertWinner(this.pTrack[2], [3, 5, 7]);
      }
    },

    alertWinner(player, winnableTrack = []) {
      this.gameOver = true;
      this.gameStarted = false;
      let elem = document.querySelector(".game-alert");
      if (player == "player") {
        if (this.player == "circle") {
          this.showPatternWonByPlayer(winnableTrack, this.greenCircle);
        } else {
          this.showPatternWonByPlayer(winnableTrack, this.greenCross);
        }

        elem.style.color = "#055305";
        this.gameAlert = "Wow...You won!";
        this.playerScore++;
        this.storeResultInDatabase("player", this.playerScore);
      } else {
        if (this.player == "circle") {
          this.showPatternWonByPlayer(winnableTrack, this.redCircle);
        } else {
          this.showPatternWonByPlayer(winnableTrack, this.redCross);
        }
        elem.style.color = "#af0909";
        this.gameAlert = "Oops...You lost!";
        this.computerScore++;
        this.storeResultInDatabase("computer", this.computerScore);
      }
    },

    fetchScore() {
      fetch(this.root + "/game/getScore")
        .then(res => res.json())
        .then(data => {
          if (data.status) {
            this.playerScore = data.player;
            this.computerScore = data.computer;
          }
        })
        .catch(err => {
          console.log(err);
        });
    },

    storeResultInDatabase(winner, score) {
      let formdata = new FormData();
      formdata.append("winner", winner);
      formdata.append("score", score);
      fetch(this.root + "game/saveScore", {
        method: "post",
        body: formdata
      }).catch(err => {
        console.log(err);
      });
    },

    showPatternWonByPlayer(winnableTrack, imgSrc) {
      winnableTrack.forEach(num => {
        document.querySelector("#cell-" + num).innerHTML =
          "<img src=" + imgSrc + " />";
      });
    },

    alertPlayerTurn() {
      if (this.firstPlayer == "player") {
        this.gameAlert = "Your turn to play first";
      } else {
        this.gameAlert = "Computer's turn to play first";
      }
    },

    clearGameAlert() {
      this.gameAlert = "";
    },

    newGame() {
      let date = new Date();
      let day = date.getDay();

      if (day != 0 && day != 6) {
        this.gameAlert = "This game is only available on weekends";
        return;
      }

      if (this.player == "") {
        this.gameAlert = "Select any symbol below";
        return;
      }

      this.clearSymbolBox();

      this.gameStarted = true;
      this.gameOver = false;
      this.gameAlert = "";
      this.computerHasPlayed = false;
      this.tacticChoice = [];
      this.tacticSelected = false;

      this.allBoxes = [1, 2, 3, 4, 5, 6, 7, 8, 9];
      this.pTrack = ["", "", "", "", "", "", "", "", ""];

      this.unChangedPossibleWins = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],

        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9],

        [1, 5, 9],
        [3, 5, 7]
      ];

      this.compTrackForWin = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],

        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9],

        [1, 5, 9],
        [3, 5, 7]
      ];

      this.elemAry = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],

        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9],

        [1, 5, 9],
        [3, 5, 7]
      ];

      this.tacticAry = [
        [1, 3, 7],
        [1, 3, 9],
        [1, 7, 9],
        [3, 9, 7],

        [1, 5, 3],
        [1, 5, 7],

        [3, 5, 9],

        [9, 5, 7]
      ];

      this.setFirstPlayer();
      this.alertPlayerTurn();
      setTimeout(this.clearGameAlert, 2000);
      this.startGame();
    }
  },

  mounted() {
    this.fetchScore();
  }
};
</script>

<style lang="scss">
#app-container {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;

  .game-alert {
    text-align: center;
    font-weight: 600;
    margin: 4rem auto 1rem;
  }

  .symbol-option {
    display: flex;
    justify-content: center;
    align-items: center;

    span {
      font-size: 1.5em;
    }

    img {
      margin-left: 10px;
      height: 25px;
      width: 25px;
      box-shadow: 0px 2px 4px 0 #ccc;
      cursor: pointer;
    }
  }

  .app-body {
    height: 25rem;
    width: 25rem;
    margin: 1rem auto 1rem;
    box-shadow: 0 2px 4px 0 #ccc;
    background: white;

    .cell {
      float: left;
      width: 33%;
      border: 1px solid #ccc;
      height: 33%;
      display: flex;
      justify-content: center;
      align-items: center;
      cursor: pointer;
    }

    .cell-1,
    .cell-2,
    .cell-3 {
      border-top: none;
    }

    .cell-7,
    .cell-8,
    .cell-9 {
      border-bottom: none;
    }

    .cell-1,
    .cell-4,
    .cell-7 {
      border-left: none;
    }

    .cell-3,
    .cell-6,
    .cell-9 {
      border-right: none;
    }
  }

  .game-score {
    width: 25rem;
    margin: 1rem auto 1rem;
    display: flex;
    justify-content: space-around;

    p {
      font-weight: 600;
      color: #444343;
      display: flex;
      justify-content: center;
      align-items: center;

      img {
        height: 20px;
        width: 20px;
        margin-right: 10px;
      }
    }
  }

  .game-control-btn {
    width: 30rem;
    margin: 1rem auto 1rem;

    button {
      box-shadow: 0 2px 4px 0 #ccc;
      width: 30%;
      padding: 5px 1%;
      margin: 0 1%;
      border: none;
      border-radius: 15px;
      font-size: 1em;
      color: #fff;
      cursor: pointer;
    }

    .new-game-btn {
      background: green;
    }

    .cancel-game-btn {
      background: red;
    }

    .new-game-btn:hover {
      background: #055305;
    }

    .cancel-game-btn:hover {
      background: #af0909;
    }
  }
}
</style>
