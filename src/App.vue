<template>
  <div id='app'>
    <div class='top left button green'
         :class='{"green active": green.LED}'
         @mousedown='input("green")'>
    </div>
    <div class='top right button red'
         :class='{"red active": red.LED}'
         @mousedown='input("red")'>
    </div>
    <div class='bottom left button yellow'
         :class='{"yellow active": yellow.LED}'
         @mousedown='input("yellow")'>
    </div>
    <div class='bottom right button blue'
         :class='{"blue active": blue.LED}'
         @mousedown='input("blue")'>
    </div>
    <div class='inner'>
      <div class='title'>Simon</div>
      <div class='controls'>
        <div id='screen_wrap'>
          <div id='screen'
               :class='{"screen active": on}'>
            <transition enter-active-class='animated fadeIn'
                        leave-active-class='animated fadeOut'>
              <div v-if='on && !started'>simon</div>
              <div id='count'
                   v-if='on && started'>{{count}}</div>
            </transition>
          </div>
          COUNT
        </div>
        <div id='start_wrap'>
          <div id='start' class='btn'
               @click='start'></div>
          START
        </div>
        <div id='strict_wrap'>
          <div id='strict_LED'
               :class='{"strict_LED active": strictMode}'></div>
          <div id='strict' class='btn'
               @click='changeMode()'></div>
          STRICT
        </div>
        <div id='power_wrap'>
          <label class='lbl'>OFF</label>
          <label class="switch">
            <input type="checkbox"
                   v-model='on'
                   @change='power'>
            <div class="slider"></div>
          </label>
          <label class='lbl'>ON</label>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      on: false,
      started: false,
      strictMode: false,
      unclickable: false,

      count: 1,
      index: 0,

      sequence: [],
      keys: ['red', 'blue', 'yellow', 'green'],

      green: {
        LED: false,
        tone: new Audio('https://s3.amazonaws.com/freecodecamp/simonSound4.mp3')
      },

      red: {
        LED: false,
        tone: new Audio('https://s3.amazonaws.com/freecodecamp/simonSound3.mp3')
      },

      blue: {
        LED: false,
        tone: new Audio('https://s3.amazonaws.com/freecodecamp/simonSound1.mp3')
      },

      yellow: {
        LED: false,
        tone: new Audio('https://s3.amazonaws.com/freecodecamp/simonSound2.mp3')
      },

      buzz: new Audio('https://www.dropbox.com/s/uvvwxzgifila5uo/lose.wav?dl=1'),
    }
  },
  methods: {
    power(){
      if(!this.on){
        this.reset();
        this.strictMode = false;
      }
    },

    start(){
      if(this.on && !this.started){
        this.init();
        this.started = true;
        this.pattern();
      }
    },

    init(){
      let count = 0;
      const sequence = [];

      while(count < 20){
        const ind = Math.floor(Math.random() * 4);
        const color = this.keys[ind];
        sequence.push(color);
        count++;
      }
      this.sequence = sequence;
    },

    reset(){
      clearInterval(this.interval);
      this.unclickable = false;
      this.started = false;
      this.count = 1;
      this.index = 0;
    },

    pattern(){
      this.unclickable = true;
      this.interval = setInterval(() => {
        const color = this.sequence[this.index];
        this.output(color);
        this.index++;

        if(this.index == this.count){
          this.index = 0;
          clearInterval(this.interval);
          this.unclickable = false;
        }
      }, 600)
    },

    input(color){
      if(!this.unclickable && this.started){
        this.output(color);
        this.check(color);
      }
    },

    check(color){
      const curr = this.sequence[this.index];

      if(color == curr)
        this.index++;

      if(color != curr && !this.strictMode){
        this.unclickable = true;
        setTimeout(() => {
          this.buzz.play();
        }, 600)
        setTimeout(() => {
          this.index = 0;
          this.pattern();
        }, 1500)
      }

      if(color != curr && this.strictMode){
        this.unclickable = true;
        setTimeout(() => {
          this.buzz.play();
        }, 600)
        setTimeout(() => {
          this.reset();
        }, 1500)
      }

      if(this.index == this.count){
        this.index = 0;
        setTimeout(() => {
          this.count++;
          this.pattern();
        }, 500)
      }
    },

    output(color){
      const tone = this[color].tone;
      const len = Math.round(tone.duration * 1000);

      this[color].LED = true;
      tone.play();
      setTimeout(() => {
        this[color].LED = false;
      }, len)
    },

    changeMode() {
      if (this.on && !this.started) {
        this.strictMode = !this.strictMode;
      }
    }
  }//METHODS
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Alfa+Slab+One|Oswald|Lato:300');
$size: 250px;
$inside: 10px solid #333;
$outside: 20px solid #333;

$green: #1E824C;
$red: #CF000F;
$yellow: #F9BF3B;
$blue: #3A539B;

$title: 'Alfa Slab One', sans-serif;
$label: 'Oswald', sans-serif;
* {
  box-sizing: border-box;
}

body {
  background: rgba(51,51,51,.5);
}

.center {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%,-50%);
}

#app {
  @extend .center;
  width: $size * 2;
  height: $size * 2;
  border-radius: 50%;
  box-shadow: 0px 0px 12px #222;
  user-select: none;
}

.button {
  position: absolute;
  width: $size;
  height: $size;
  cursor: pointer;
}

.top {
  top: 0;
  border-top: $outside;
  border-bottom: $inside;
&.left {
   border-top-left-radius: 100%;
 }
&.right {
   border-top-right-radius: 100%;
 }
}

.bottom {
  top: $size;
  border-bottom: $outside;
  border-top: $inside;
&.left {
   border-bottom-left-radius: 100%;
 }
&.right {
   border-bottom-right-radius: 100%;
 }
}

.left {
  left: 0;
  border-left: $outside;
  border-right: $inside;
}

.right {
  left: $size;
  border-right: $outside;
  border-left: $inside;
}

.green {
  background: rgba($green,.6);
&.active {
   background: rgba($green,1);
 }
}

.red {
  background: rgba($red,.6);
&.active {
   background: rgba($red,1);
 }
}

.yellow {
  background: rgba($yellow,.6);
&.active {
   background: rgba($yellow,1);
 }
}

.blue {
  background: rgba($blue,.6);
&.active {
   background: rgba($blue,1);
 }
}

.inner {
  @extend .center;
  width: $size;
  height: $size;
  background: #333;
  border-radius: 50%;
.title {
  position: relative;
  top: 40px;
  font-family: $title;
  color: white;
  font-size: 60px;
  text-align: center;
}
.controls {
  position: relative;
  top: 40px;
  width: 80%;
  margin-left: 10%;
  height: 100px;
  font-family: $label;
  color: white;
  text-align: center;
  font-size: 12px;
}
}

#screen_wrap {
  width: 90px;
  height: 60px;
}

#screen {
  width: 100%;
  height: 40px;
  background: rgba(#6C7A89,.7);
  border-radius: 10px;
  box-shadow: inset 0 0 5px #333;
  line-height: 40px;
  font-size: 20px;
  color: black;
  text-transform: uppercase;
  font-family: 'Lato';
  letter-spacing: 2px;
#count {
  font-size: 25px;
}
&.active {
   background: rgba(#6C7A89,1);
 }
}

#start_wrap {
  position: absolute;
  top: 8px;
  left: 100px;
  width: 50px;
  height: 40px;
}

#start {
  width: 60%;
  height: 10px;
  margin-top: -3px;
  background: $red;
}

#strict_wrap {
  position: absolute;
  top: 0;
  left: 160px;
  width: 50px;
  height: 40px;
}

#strict {
  width: 60%;
  height: 10px;
  margin-top: -3px;
  background: $yellow;
}

#strict_LED {
  position: relative;
  width: 10px;
  height: 8px;
  border-radius: 50%;
  margin-left: auto;
  margin-right: auto;
  background: rgba($red,.4);
&.active {
   background: rgba($red,1);
 }
}

#power_wrap {
  height: 24px;
  position: relative;
  top: 20px;
}

.lbl {
  position: relative;
  bottom: 12px;
  margin: 4px;
}

.switch {
  position: relative;
  display: inline-block;
  width: 42px;
  height:22px;
input {
  display: none;
}
}

.slider {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #474747;
  transition: .2s;
  border-radius: 3px;
  cursor: pointer;
&::before {
   position: absolute;
   height: 20px;
   width: 20px;
   left: 1px;
   bottom: 1px;
   background: $red;
   transition: .2s;
   border-radius: 3px;
   content: '';
 }
}

input:checked + .slider:before {
  background: $green;
  transform: translateX(20px);
}
</style>
