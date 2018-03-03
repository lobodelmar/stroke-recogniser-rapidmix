<template>
  <div id="app">
    <h1>{{ msgs[this.gameState] }}</h1>
    <canvas id="drawing" width="600" height="400" @mousemove="addPoint"
                                                  @mousedown="drawingStatus=true"
                                                  @mouseup="finishedDrawing">
    </canvas>

   <div>
      <h2>{{this.msgs[3]}} {{this.answer}}</h2>
   </div>
   <div>
     <button id="resetbutton" @click="resetModel">reset model</button>
   </div>
  </div>
</template>

<script>
var rapidlib = window.RapidLib();
var myDTW = new rapidlib.SeriesClassification();

export default {
  name: 'app',
  data () {
    return {
      msgs: [
        "draw training shape one",
        "draw training shape two",
        "draw test shape",
        "your closest match is "
      ],
      cv: "",
      drawingStatus: false,
      ctx: "",
      gameState: 0,
      //these arrays will hold two training examples and one to test against.
      mlData: [{ input: [], label: "shape 1" }, { input: [], label: "shape 2" }, { input: [], label: "test shape" }],
      answer: "N/A"
    }
  },
  methods: {
     addPoint(e){
      if (this.drawingStatus) {
          this.mlData[this.gameState].input.push([e.offsetX, e.offsetX])
          this.ctx.lineTo(e.offsetX, e.offsetY);
          if (this.gameState<2) this.ctx.strokeStyle = 'black';
          else this.ctx.strokeStyle = 'red';
          this.ctx.lineWidth = 2;
          this.ctx.stroke();
      }
    },
    finishedDrawing(){
      this.clearCanvas()
      this.drawingStatus=false;
      if (this.gameState==2){
        //clear out the model
        myDTW.reset();
        //train it on an array of training examples
        var seriesSet = [this.mlData[0], this.mlData[1]];
        myDTW.train(seriesSet);
        //run on a test series
        var match = myDTW.run(this.mlData[2].input);
        // //get the costs of all matches
        var costs = myDTW.getCosts();
        this.mlData[2].input = [];
        this.answer = match
        console.log("testSet ", seriesSet);
        console.log("match ", match);
        console.log("costs ", costs);
      }
      else this.gameState++;
    },
    clearCanvas()
    {
      this.ctx.clearRect(0,0,this.cv.width,this.cv.height);
      this.ctx.beginPath();
    },
    resetModel(){
      this.gameState=0;
      this.mlData[0].input = [];
      this.mlData[1].input = [];
      this.mlData[2].input = [];
      this.drawingStatus = false;
      this.answer = "N/A";
    }
  },
  mounted(){
    this.cv=document.getElementById("drawing");
    this.ctx=this.cv.getContext("2d");
    this.clearCanvas();
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#resetbutton {
  background-color: #4CAF50; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
}
canvas {
  border:3px solid #4CAF50;
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
canvas {
  margin:0px;
  padding:0px;
}
</style>
