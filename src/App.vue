<template>
  <div id="app">
    <img width="40%" alt="Wumpus logo" src="./assets/logo.png">
    <h2>Score: {{ returnValue.score }}</h2>
    <div v-if="returnValue.states">
      <!-- <h2>Action: {{ currentAction }}</h2> -->
      <div class="container">
        <button @click="previousPage" class="button">
          <p>&lt;</p>
        </button>
        <div class="map">
          <div v-for="(row, rowIndex) in currentCave" :key="rowIndex" class="row">
            <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="cell">
              <template v-if="cell.includes('U')">
                <div class="image-stack">
                  <img
                    src="./assets/NotVisited.png"
                    class="overlay-image"
                    alt="Human Image"
                    :style="rotationStyle">
                </div>
              </template>
              <template v-else>
                <!-- 조건에 따라 겹쳐서 이미지 표시 -->
                <div class="image-stack">
                  <!-- Human 이미지가 표시될 조건 -->
                  <img v-if="rowIndex === agentPosition.x && cellIndex === agentPosition.y"
                    src="./assets/Human.png"
                    class="overlay-image"
                    alt="Human Image"
                    :style="rotationStyle">

                  <!-- 각 셀의 상태에 따른 이미지 표시 -->
                  <img v-if="cell.includes('G')"
                    src="./assets/Gold.png"
                    class="overlay-image"
                    alt="Gold Image">
                  <img v-else-if="cell.includes('P')"
                    src="./assets/Pit.png"
                    class="overlay-image"
                    alt="Pit Image">
                  <img v-else-if="cell.includes('W')"
                    src="./assets/Wumpus.png"
                    class="overlay-image"
                    alt="Wumpus Image">

                  <img v-if="cell.includes('B')"
                    src="./assets/Breeze.png"
                    class="overlay-image"
                    alt="Breeze Image">
                  <img v-if="cell.includes('S')"
                    src="./assets/Stench.png"
                    class="overlay-image"
                    alt="Stench Image">
                </div>
              </template>
            </div>
          </div>
        </div>
        <button @click="nextPage" class="button">
          <p>&gt;</p>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "App",
  data() {
    return {
      page: 0,
      returnValue: {},
    };
  },
  mounted() {
    this.getStates();
  },
  computed: {
    currentCave() {
      return this.returnValue.states[this.page % this.returnValue.states.length].cave;
    },
    currentAction() {
      return this.returnValue.states[this.page % this.returnValue.states.length].action;
    },
    agentPosition() {
      const state = this.returnValue.states[this.page % this.returnValue.states.length];
      return { x: state.agentX, y: state.agentY };
    },
    currentDirection() {
      return this.returnValue.states[this.page % this.returnValue.states.length].agentDirection;
    },
    rotationStyle() {
      let degrees = 0;
      switch (this.currentDirection) {
        case 'RIGHT':
          degrees = 0;
          break;
        case 'UP':
          degrees = 90;
          break;
        case 'LEFT':
          degrees = 180;
          break;
        case 'DOWN':
          degrees = 270;
          break;
      }
      return {
        transform: `rotate(${degrees}deg)`
      };
    }
  },
  methods: {
    getStates() {
      axios.get("http://localhost:8080/api")
        .then(res => {
          console.log("성공", res);
          this.returnValue = res.data;
        })
        .catch(err => {
          console.error("실패", err);
          this.returnValue = { result: "error", states: [] };
        });
    },
    previousPage() {
      this.page = (this.page - 1 + this.returnValue.states.length) % this.returnValue.states.length;
    },
    nextPage() {
      this.page = (this.page + 1) % this.returnValue.states.length;
    },
    getImageSrc(cell, rowIndex, cellIndex) {
      const content = this.determineCellContent(cell, rowIndex, cellIndex, this.agentPosition.x, this.agentPosition.y, this.currentDirection);
      return `./assets//${content}.png`;
    },
    determineCellContent(cell, rowIndex, cellIndex, agentPosX, agentPosY, currentDirection) {
      let result = "";
      if (rowIndex === agentPosX && cellIndex === agentPosY) {
        result += this.getCellType(cell) + this.getEnvironmentConditions(cell);
        result += "Human" + currentDirection;
      } else {
        if (!cell.includes('U')) {
          result = this.getCellType(cell) + this.getEnvironmentConditions(cell);
        } else {
          result = "NotVisited";
        }
      }
      return result;
    },
    getCellType(cell) {
      if (cell.includes('G')) return "Gold";
      if (cell.includes('P')) return "Pit";
      if (cell.includes('W')) return "Wumpus";
      return "";
    },
    getEnvironmentConditions(cell) {
      let conditions = "";
      if (cell.includes('B')) conditions += "Breeze";
      if (cell.includes('S')) conditions += "Stench";
      return conditions;
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 900px;
}

.button {
  height: 540px;
  width: 10%;
}

.map {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  background: #000;
  padding: 3px;
}

.cell {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-left: 0px;
  margin-right: 0px;
  margin-top: 5px;
  margin-bottom: 5px;
}

.image-stack {
  position: relative;
  width: 126px; /* 적절한 크기 지정 */
  height: 126px; /* 적절한 크기 지정 */
  background: white;
}

.overlay-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 90%;
  height: 90%;
  object-fit: cover; /* 이미지가 셀에 꽉 차게 표시 */
}
</style>
