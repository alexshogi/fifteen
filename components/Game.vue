<template>
  <section class="game">
    <link href="https://fonts.googleapis.com/css?family=Inconsolata&display=swap" rel="stylesheet">
    <div class="header">
      <div class="steps">
        <h2>Ходов</h2>
        <p>{{ count }}</p>
      </div>
      <div class="restart">
        <svg @click="restart" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px" viewBox="0 0 489.711 489.711" style="enable-background:new 0 0 489.711 489.711;" xml:space="preserve">
          <g>
            <g>
              <path d="M112.156,97.111c72.3-65.4,180.5-66.4,253.8-6.7l-58.1,2.2c-7.5,0.3-13.3,6.5-13,14c0.3,7.3,6.3,13,13.5,13
                c0.2,0,0.3,0,0.5,0l89.2-3.3c7.3-0.3,13-6.2,13-13.5v-1c0-0.2,0-0.3,0-0.5v-0.1l0,0l-3.3-88.2c-0.3-7.5-6.6-13.3-14-13
                c-7.5,0.3-13.3,6.5-13,14l2.1,55.3c-36.3-29.7-81-46.9-128.8-49.3c-59.2-3-116.1,17.3-160,57.1c-60.4,54.7-86,137.9-66.8,217.1
                c1.5,6.2,7,10.3,13.1,10.3c1.1,0,2.1-0.1,3.2-0.4c7.2-1.8,11.7-9.1,9.9-16.3C36.656,218.211,59.056,145.111,112.156,97.111z"/>
              <path d="M462.456,195.511c-1.8-7.2-9.1-11.7-16.3-9.9c-7.2,1.8-11.7,9.1-9.9,16.3c16.9,69.6-5.6,142.7-58.7,190.7
                c-37.3,33.7-84.1,50.3-130.7,50.3c-44.5,0-88.9-15.1-124.7-44.9l58.8-5.3c7.4-0.7,12.9-7.2,12.2-14.7s-7.2-12.9-14.7-12.2l-88.9,8
                c-7.4,0.7-12.9,7.2-12.2,14.7l8,88.9c0.6,7,6.5,12.3,13.4,12.3c0.4,0,0.8,0,1.2-0.1c7.4-0.7,12.9-7.2,12.2-14.7l-4.8-54.1
                c36.3,29.4,80.8,46.5,128.3,48.9c3.8,0.2,7.6,0.3,11.3,0.3c55.1,0,107.5-20.2,148.7-57.4
                C456.056,357.911,481.656,274.811,462.456,195.511z"/>
            </g>
          </g>
        </svg>
      </div>
      <div class="time">
        <h2>Время</h2>
        <p>{{ time }}</p>
      </div>
    </div>
    <div class="field noselect">
      <div class="preloader-container" v-if="loading">
        <div class="preloader"></div>
      </div>
      <template v-else>
        <div  class="cell"
              v-for="(cell, cellIndex) in gameField"
              :key="cellIndex"
              @click="swap(cellIndex);"
              :class="{
                active: cellIndex === active,
                right: cellIndex + 1 === emptyIndex,
                left: cellIndex - 1 === emptyIndex,
                up: emptyIndex - cellIndex > 1,
                down: cellIndex > emptyIndex + 1,
                empty: cell === 0,
                awailable: closeToEmpty.includes(cellIndex),
              }">
          <span v-if="cell > 0">
            {{ cell }}
          </span>
        </div>
      </template>
    </div>
    <div class="modal" v-if="isDone">
      <div class="modal-content">
        <h2>Победа!</h2>
        <button @click.prevent="restart" class="button">Еще раз</button>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      count: 0,
      time: '00:00:00',
      started: null,
      update: null,
      field: [],
      active: null,
      loading: true,
      playing: false,
    }
  },
  computed: {
    isDone() {
      if (this.field.length) {
        let current = this.field.filter(elem => elem > 0);
        let right = this.field.filter(elem => elem > 0).sort((a, b) => a - b );
        return current === right;
      }
      return false;
    },
    gameField() {
      return this.field;
    },
    emptyIndex() {
      if (this.field.length) {
        const arr = this.field;
        const index = arr.findIndex(elem => elem === 0);
        return index;
      }
      return false;
    },
    closeToEmpty() {
      const arr = this.field;
      const empty = this.emptyIndex;

      let res = [];
      let max = 4 * 4 - 1;

      if (empty >= 0) {
        if (empty + 1 <= max && (empty + 1) !== 4 && (empty + 1) !== 8 && (empty + 1) !== 12) {
          res.push(empty + 1);
        }
        if (empty + 4 <= max) {
          res.push(empty + 4);
        }
        if (empty - 4 >= 0) {
          res.push(empty - 4);
        }
        if (empty - 1 >= 0 && (empty - 1) !== 3 && (empty - 1) !== 7 && (empty - 1) !== 11) {
          res.push(empty - 1);
        }
      }
      return res;
    },
  },
  methods: {
    initField() {
      const field = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 0];
      this.field = field.sort(() => Math.random() - 0.5);
    },
    countTime() {
      this.started = new Date;
      this.update = ms => this.time = new Date(ms).toISOString().split(/T|\./)[1]
      setInterval(() => this.update(new Date - this.started), 500);
    },
    swap(index) {
      this.active = this.emptyIndex;
      if (!this.playing) {
        this.playing = true;
        this.countTime();
      }
      setTimeout(() => {
        this.active = null;
      }, 200);
      const value = this.field[index];
      const emptyInd = this.emptyIndex;
      const tmpArr = this.field;
      tmpArr[emptyInd] = value;
      tmpArr[index] = 0;
      this.field = [...tmpArr];
      this.count += 1;
    },
    restart() {
      this.playing = false;
      this.count = 0;
      this.time = '00:00:00';
      this.started = null;
      this.update = null;
      this.field = [];
      this.active = null;
      this.loading = true;
      this.playing = false;
      this.initField();
      setTimeout(() => {
        this.loading = false;
      }, 700);
    },
  },
  mounted() {
    this.initField();
    setTimeout(() => {
      this.loading = false;
    }, 700);
  },
}
</script>

<style lang="scss">
  .game {

    display: flex;
    justify-content: center;
    align-items: center;
    box-sizing: border-box;
    padding: 0;
    margin: 0;
    color: #272727;
    flex-flow: column;

    .header {
      width: 426px;
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: space-around;
      color: #e9e9e9;
      border: 3px solid #e9e9e9;
      border-radius: 5px;
      box-shadow: 0px 2px 2px 0px #4e321a;
    }

    .steps,
    .restart,
    .time {
      height: 100%;
      flex: 1;
      display: flex;
      flex-direction: column;
      justify-content: space-around;
      align-items: center;
      padding: 10px 0;
    }

    .steps h2,
    .time h2 {
      text-transform: uppercase;
    }

    .restart svg {
      height: 40px;
      width: 40px;
      fill: #e9e9e9;
      cursor: pointer;
      transition: color 0.14s;
      &:hover {
        fill: #ffc600;
      }
    }

    .modal {
      position: fixed;
      z-index: 1;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: rgba(0,0,0,0.84);

      .modal-content {
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: space-around;
        color: #f0f0f0;
        background-color: rgba(255,255,255,0.1);;
        margin: 15% auto;
        padding: 3em;
        width: 80%;
        max-width: 400px;
        min-height: 30%;
        box-shadow: 0 2px 3px rgba(0,0,0,0.7);
        border: 3px solid #e9e9e9;
        border-radius: 5px;
      }

      .button {
        border: none;
        border-radius: 6px;
        padding: 8px 24px;
        font-size: 16px;
        background: #fa6c9f;
        background: linear-gradient(135deg, #fa6c9f 0%, #ffe140 80%, #ffe140 100%);
        color: #272727;
      }
    }

    .field {
      width: 426px;
      height: 426px;
      display: flex;
      margin-top: 30px;
      border: 3px solid #e9e9e9;
      border-radius: 5px;
      flex-wrap: wrap;
      padding: 2px;
      box-shadow: 0px 2px 2px 0px #4e321a;
    }

    .cell {
      height: 100px;
      width: 100px;
      background-color: #ffd4a2;
      border-radius: 2px;
      margin: 2px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 25px;
      font-weight: bold;
      pointer-events: none;
      box-shadow: 0px 1px 2px 0px #4e321a;

      &.empty {
        background-color: transparent;
        border-color: transparent;
        box-shadow: none;
      }
      &.awailable {
        cursor: pointer;
        pointer-events: all;
      }
    }

    .noselect {
      -webkit-touch-callout: none;
      -webkit-user-select: none;
      -khtml-user-select: none;
      -moz-user-select: none;
      -ms-user-select: none;
      user-select: none;
    }

    .active {
      &.down {
        animation: moveDown 0.2s ease-in-out;
      }

      &.up {
        animation: moveUp 0.2s ease-in-out;
      }

      &.left {
        animation: moveLeft 0.2s ease-in-out;
      }

      &.right {
        animation: moveRight 0.2s ease-in-out;
      }
    }

    .preloader-container {
      display: flex;
      flex: 1;
      justify-content: center;
      align-items: center;
    }

    .preloader {
      height: 70px;
      width: 70px;
      border: 5px solid #e6810f;
      border-top: 5px solid #ffc600;
      border-radius: 50%;
      animation: rotate 1s infinite linear;
    }

    @keyframes moveUp {
      from {
        transform: translate(0, 104px);
      }
      90% {
        transform: translate(0, -4px);
      }
      to {
        transform: translate(0, 0);
      }
    }

    @keyframes moveDown {
      from {
        transform: translate(0, -104px);
      }
      90% {
        transform: translate(0, 4px);
      }
      to {
        transform: translate(0, 0);
      }
    }

    @keyframes moveLeft {
      from {
        transform: translate(-104px, 0);
      }
      90% {
        transform: translate(4px, 0);
      }
      to {
        transform: translate(0, 0);
      }
    }

    @keyframes moveRight {
      from {
        transform: translate(104px, 0);
      }
      90% {
        transform: translate(-4px, 0);
      }
      to {
        transform: translate(0, 0);
      }
    }

    @keyframes rotate {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }

  }
</style>
