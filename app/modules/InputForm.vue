<template lang="html">
  <div id="InputForm">
    <div>
      <div v-if="result">
        <h2 class="result-text">{{result}}</h2>
      </div>
        <div class="grid" :style="gridInputs" v-if="this.decisions.length>0">
          <div class="slide-input" v-for="decision in decisions">
            <input class="inpt" v-model="decision.value" :placeholder="placeholderText(decision)"/>
            <div class="input-border"></div>
          </div>
        </div>
        <div class="errors" v-if="errors.length!==0">
          <p v-for="error in errors">{{error.error}}</p>
        </div>
        <div class="button-group">
          <div class="button" @click="addDecision()">
            Add Option
          </div>
          <div :class="buttonClass" @click="makeDecision()">
            Make Decision
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";
export default {
  name: "InputForm",
  data: () => ({
    decisions: [],
    errors: [],
    result: ""
  }),
  computed: {
    gridInputs() {
      if (this.decisions.length > 0) {
        return `grid-template-columns:repeat(${this.decisions.length},1fr)`;
      }
      return;
    },
    buttonClass() {
      return this.decisions.length > 1 && !this.isEmpty()
        ? "button"
        : "button button-disabled";
    }
  },
  methods: {
    placeholderText(decision) {
      return `Option ${decision.id + 1}`;
    },
    isEmpty() {
      const errorTexts = this.decisions.filter(decision => !decision.value);
      let indexOfError = 0;
      if (errorTexts.length > 0) {
        const newError = "Empty option";
        const errors = this.errors.filter(error => error.error !== newError);
        errors.push({ id: "emptyError", error: newError });
        this.errors = [...errors];
        return true;
      } else {
        this.errors.forEach((error, index) => {
          if (error.id === "emptyError") {
            indexOfError = index;
          }
        });
        this.errors.splice(indexOfError,1);
        return false;
      }
    },
    addDecision() {
      const len = Object.keys(this.decisions).length;
      this.decisions.push({ value: "", id: this.decisions.length });
    },
    getNumberIndexes() {
      const len = this.decisions.length;
      const base = 100;
      const partitions = Math.round(base / len * 100) / 100;
      let curr = 0;
      const decisions = this.decisions.map(decision => {
        const endRange = curr + partitions;
        const a = Object.assign({}, decision);
        a.from = curr;
        a.to = endRange;
        curr += partitions;
        return a;
      });
      this.decisions = [...decisions];
    },
    makeDecision() {
      const empty = this.isEmpty();
      if (empty || this.decisions.length < 2) {
        return;
      }
      this.getNumberIndexes();
      const result = Math.round(_.random(0, 100, true) * 100) / 100;
      const finalObject = this.decisions.filter(
        decision => decision.from <= result && decision.to >= result
      );
      this.result = finalObject[0].value;
    }
  }
};
</script>

<style lang="css">
#InputForm{
    height:100vh;
    display:flex;
    justify-content: center;
    align-items: center;
}

.errors{
    margin:10px auto;
    text-align: center;
}

.errors p{
  font-family: "Lato";
  font-size:14px;
  color:rgba(255,0,0,0.5);
}

.button-group{
  margin-top:50px;
  display: flex;
  justify-content:center;
  align-items: center;
}

.grid{
    display: grid;
    grid-column-gap: 10px;
}

.result-text{
  font-size:100px;
  text-align:center;
  color:#333;
  font-family:"Lato","Arial",sans-serif;
}

.slide-input{
  position:relative;
}

.input-border{
   position: relative;
}

.input-border:after{
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 4px;
  transform: scaleX(0);
  background:#11CBD7;
  transition: transform 250ms;
}

.inpt:focus + .input-border:after{
  transform: scaleX(1);
}

.inpt{
    width:100%;
    box-sizing: border-box;
    padding:8px 10px;
    padding-bottom:2px;
    outline:none;
    background:white;
    color:#555;
    border:0px;
    border-bottom:4px solid #555;
    font-size:20px;
}

.button{
  margin-left:10px;
  user-select: none;
  display: inline-block;
  padding:16px;
  background:#11CBD7;
  color:rgba(255,255,255,.8);
  font-weight: bold;
  font-family: "Lato";
  border-radius: 2px;
  box-shadow: inset 0px -4px rgba(0,0,0,0.2);
  transition: .2s ease;
}
.button:hover{
  box-shadow: inset 0px 0px rgba(0,0,0,0.2);
}
.button.button-disabled{
  background:grey;
  cursor: not-allowed;
}
.button.button-disabled:hover{
  box-shadow: inset 0px -4px rgba(0,0,0,0.2);
}
</style>
