<template>
  <div>
    <div class="page-bak">
      <div class="title-txt">一起猜成语</div>
      <div class="game-field">
        <div class="gate-txt">第{{num2ChStr(curLevel)}}关</div>
        <!--游戏矩阵-->
        <div class="box-con">
          <div v-for="(it,idx) in idiomMap" class="box-item" :class="getMapClass(it, idx)"
               :key="idx" @click="onMapItCLick(it,idx)">
            {{getMapChar(it)}}
          </div>
        </div>
        <!--按钮行-->
        <div class="btn-line">
          <div/>
          <div class="line-right">
            <div class="promot-it" @click="showTipPop">
              <van-icon name="info" size="27"/>
              <div>提示</div>
            </div>
          </div>
        </div>
        <!--答案矩阵-->
        <div class="options-con">
          <div v-for="(it,idx) in answerMap" class="option-item" :class="getAnsClass(it)"
               @click="onAnsItClick(it)" :key="idx">{{getAnswerChar(it)}}
          </div>
        </div>
        <!--按钮-->
        <div class="jump-con">
          <van-button class="jump-btn" type="info" @click="jumpOverLevel">跳过本关
          </van-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {Button, Dialog, Icon, NavBar} from 'vant';

  const IDIOM_MAP = require('./idiom_map.json');

  export default {
    name: "app",
    components: {
      [Button.name]: Button,
      [Icon.name]: Icon,
      [NavBar.name]: NavBar
    },
    computed: {
      getMapClass() {
        return (it, idx) => {
          if (it.type == 2) {
            return 'box-it-nor'
          } else if (it.type == 3) {
            if (it.ans && idx == it.ans.i) {
              return 'box-it-filled'
            }
            return 'box-it-error'
          } else if (it.type == 4) {
            return 'box-it-ok'
          } else if (it.type == 1) {
            if (idx == this.cursorIndex) {
              return 'box-it-focus'
            }
          }
          return 'box-it-txt'
        }
      },
      getAnsClass() {
        return it => {
          if (it.used) {
            return ['option-none']
          } else {
            return ['option-char']
          }
        };
      },
      getAnswerChar() {
        return (it) => {
          if (!it.used) {
            return it.c
          } else {
            return ''
          }
        }
      },
      getMapChar() {
        return it => {
          let t = it.type;
          if (t == 0 || t == 3 || t == 4) {
            return it.txt;
          }
          return '';
        }
      },
    },
    data() {
      return {
        curLevel: 1,
        cursorIndex: 0,
        idiomMap: [], //type : 0正常字符，1未填空，2空格, 3已填空, 4填写正确，5填写错误
        answerMap: [], //used: 0未被使用，1已被使用
        playedLevelIdx: [], //当前玩过的关卡索引
      };
    },
    methods: {
      jumpOverLevel() {
        Dialog.confirm({
          message: '确定要跳过本关？',
        }).then(() => {
          this.curLevel++;
          this.reloadMap();
        }).catch(() => {
        });
      },
      onLevelFinish() {
        Dialog({message: '恭喜过关，继续下一关！'}).then(() => {
          this.curLevel++;
          this.reloadMap();
        });
      },
      doShowAnswer() {
        for (let i in this.answerMap) {
          let at = this.answerMap[i];
          if (at.i == this.cursorIndex) {
            this.onAnsItClick(at);
            break;
          }
        }
      },
      showTipPop() {
        Dialog.confirm({
          message: '查看答案？',
        }).then(() => {
          this.doShowAnswer();
        }).catch(() => {
        });
      },
      // 游戏关卡结束
      checkLevelFinish() {
        let up = 1;
        for (let i in this.idiomMap) {
          let t = this.idiomMap[i].type;
          if (t == 1 || t == 3) {
            up = 0;
            break;
          }
        }
        if (up) {
          this.onLevelFinish();
        }
      },
      onAnsItClick(it) {
        if (!it.used) {
          let ii = this.cursorIndex;
          if (ii < 0) {
            return
          }
          it.used = 1;
          let idmap = this.idiomMap[ii];
          if (idmap.ans) {
            idmap.ans.used = 0;
          }
          idmap.txt = it.c;
          idmap.ans = it;
          idmap.type = 3;
          this.checkLineFinish(ii);
          this.refreshFirstBlank();
          this.checkLevelFinish();
        }
      },
      checkLineFinish(pos) {
        let sp = this.findWordStart(pos);
        let xx = [], yy = [];
        for (let i = 0; i < 4; i++) {
          let idx = sp.x + i;
          if (idx >= this.idiomMap.length) {
            break;
          }
          let it = this.idiomMap[idx];
          if (it.type == 0 || it.type == 4 || (it.type == 3 && it.ans.i == idx)) {
            xx.push(idx);
          }
        }
        for (let i = 0; i < 4; i++) {
          let idx = sp.y + i * 9;
          if (idx >= this.idiomMap.length) {
            break;
          }
          let it = this.idiomMap[idx];
          if (it.type == 0 || it.type == 4 || (it.type == 3 && it.ans.i == idx)) {
            yy.push(idx);
          }
        }
        if (xx.length == 4) {
          for (let j in xx) {
            let it = this.idiomMap[xx[j]]
            it.type = 4;
          }
        }
        if (yy.length == 4) {
          for (let j in yy) {
            let it = this.idiomMap[yy[j]]
            it.type = 4;
          }
        }
      },
      isMapFilled(idx) {
        let t = this.idiomMap[idx].type;
        return t == 0 || t == 3 || t == 4;
      },
      findWordStart(pos) {
        let xx = pos, yy = pos;
        while (1) {
          if (this.isMapFilled(xx)) {
            if (xx % 9 == 0) {
              break;
            } else {
              if (this.isMapFilled(xx - 1)) {
                xx--;
              } else {
                break;
              }
            }
          }
        }
        while (1) {
          if (this.isMapFilled(yy)) {
            if (yy - 9 < 0) {
              break;
            } else {
              if (this.isMapFilled(yy - 9)) {
                yy -= 9;
              } else {
                break;
              }
            }
          }
        }
        return {x: xx, y: yy};
      },
      onMapItCLick(it, idx) {
        if (it.type == 1 || it.type == 3) {
          this.cursorIndex = idx;
          if (it.ans) { //如果选中，取消选中
            it.ans.used = 0;
            it.ans = null;
            it.type = 1;
          }
        }
      },
      // 游标动作
      refreshFirstBlank() {
        for (let i = 0; i < this.idiomMap.length; i++) {
          if (this.idiomMap[i].type == 1) {
            this.cursorIndex = i;
            return
          }
          this.cursorIndex = -1;
        }
      },
      calcMapIndex() {
        let range = IDIOM_MAP.length;
        let retry = Math.floor(range / 2);
        let idx = Math.floor(Math.random() * range);
        while (idx in this.playedLevelIdx && (retry > 0)) {
          idx = Math.floor(Math.random() * range);
          retry--;
        }
        this.playedLevelIdx.push(idx);
        return idx;
      },
      reloadMap() {
        this.cursorIndex = 0;
        let idx = this.calcMapIndex();
        let data = IDIOM_MAP[idx];
        this.idiomMap = [];
        this.answerMap = [];
        for (let ii in data.s) {
          let m = data.s[ii];
          let box = {txt: m, type: 0};
          if (m == '&') {
            box.type = 1
          } else if (m == '@') {
            box.type = 2
          }
          this.idiomMap.push(box)
        }
        for (let ii in data.a) {
          let m = data.a[ii];
          let ans = {...m, used: 0, pos: ii};
          this.answerMap.push(ans);
        }
        this.refreshFirstBlank();
      },
      num2ChStr(num) {
        let chnNumChar = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九'];
        let chnUnitChar = ['', '十', '百', '千', '万', '十', '百', '千', '亿', '十', '百', '千'];
        let chnStr = '';
        let str = num.toString();
        while (str.length > 0) {
          let tmpNum = chnNumChar[parseInt(str.substr(0, 1))];
          let tmpChar = chnUnitChar[str.length - 1];
          chnStr += (chnStr.substr(-1, 1) == '零' && tmpNum == '零') ? '' : tmpNum;
          if (tmpNum != '零') {
            chnStr += tmpChar;
          }
          if (chnStr == '一十') {
            chnStr = '十';
          }
          str = str.substr(1);
          if (parseInt(str) === 0) {
            if (str.length >= 8) {
              if (chnStr.substr(-1, 1) != '亿') {
                chnStr += '亿';
              }
            } else if (str.length >= 5) {
              if (chnStr.substr(-1, 1) != '万') {
                chnStr += '万';
              }
            }
            str = '';
          }
        }
        return chnStr;
      },
    },
    mounted() {
      this.reloadMap();
    },
  }
</script>

<style scoped lang="less">
  .btn-line {
    .my-power {
      height: 26px;
      background-size: 100% 100%;
      display: flex;
      justify-content: flex-end;
      align-items: center;
      padding-right: 10px;
    }
    .line-right {
      display: flex;
      .promot-it {
        padding: 0 8px;
      }
    }
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 4vw 3vw;
    font-size: 13px;
    color: #A84F29;
  }

  .jump-con {
    .jump-btn {
      background-size: 100% 100%;
      width: 60%;
      border: none;
      background: linear-gradient(#FE901B 0%, #FDB33D 100%);
      border-radius: 16px;
    }
    margin-top: 9vw;
    margin-bottom: 6vw;
    text-align: center;
  }

  .options-con {
    .option-char {
      opacity: 1;
    }
    .option-none {
      opacity: 0;
    }
    .option-char:active {
      background-color: #fce9c9;
    }
    .option-item {
      color: #333333;
      border-radius: 4px;
      width: 10vw;
      height: 10vw;
      display: flex;
      font-size: 7vw;
      font-weight: bold;
      border: 1px solid #333333;
      box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.3);
      justify-content: center;
      align-items: center;
    }
    justify-content: space-between;
    margin: 4vw 3vw;
    flex-wrap: wrap;
    display: flex;
  }

  .box-it-filled {
    border: 1px solid #FB220A;
    color: #333333;
    background-color: yellow;
  }

  .box-it-error {
    border: 1px solid #FB220A;
    color: #FB3803;
    background-color: yellow;
    animation: shake-red 0.2s;
  }

  @keyframes shake-red {
    0%, 20%, 40%, 60%, 80%, 100% {
      transform: translateX(-2px)
    }
    10%, 30%, 50%, 70%, 90% {
      transform: translateX(2px)
    }
  }

  .box-it-ok {
    color: white;
    background-color: #5EC001;
    border: 1px solid #333333;
    animation: blink-green 0.5s;
  }

  @keyframes blink-green {
    0% {
      background-color: #7Ee021;
    }
    50% {
      transform: scale(1.1);
    }
    100% {
      background-color: #5EC001;
    }
  }

  .box-it-nor {
    font-weight: 500;
    color: #333333;
    border: 1px solid #333333;
    opacity: 0.05;
  }

  .box-it-focus {
    border: 1px solid #FB220A;
  }

  .box-it-txt {
    border: 1px solid #333333;
  }

  .box-it-wrong {
    border: 1px solid #FB220A;
    color: #FB1C03;
  }

  .box-con {
    .box-item {
      width: 9vw;
      height: 9vw;
      border-radius: 4px;
      text-align: center;
      margin-bottom: 0.8vw;
      display: flex;
      font-size: 6vw;
      font-weight: bold;
      box-shadow: 0 3px 3px 0 rgba(0, 0, 0, 0.3);
      justify-content: center;
      align-items: center;
    }
    justify-content: space-between;
    flex-wrap: wrap;
    margin-top: 2vw;
    display: flex;
  }

  .game-field {
    .gate-txt {
      margin: 1vw 0;
      font-size: 25px;
      font-weight: bold;
      color: #2A2A2A;
    }
    background-color: #FFFBF5;
    margin: 1.5vw;
    padding: 1.5vw;
    border-radius: 4px;
    text-align: center;
  }

  .page-bak {
    .title-txt {
      color: #664022;
      margin-bottom: 6vw;
      font-size: 20px;
      font-weight: bold;
      position: fixed;
      width: 100%;
      top: 0;
      padding: 40px 0 20px 0;
      z-index: 1000;
      background-color: #FCE6C5;
    }
    width: 100vw;
    margin-top: 80px;
    padding-top: 1vw;
    min-height: 100vh;
    text-align: center;
    background-size: 100% auto;
    background: #FCE6C5;
  }

  #app {
    min-height: 100vh;
    background-color: #FCE6C5;
  }
</style>
