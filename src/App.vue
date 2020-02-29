<template>
  <div id="app">
    <div class="button">
      <button @click="apply">Apply ({{list.length - currentIndex}} left)</button>
    </div>

    <div class="controls">
      <label>
        <span>Опыт в месяцах</span>
        <input type="number" v-model="currentParams[0]" />
      </label>
      <label>
        <span>Тематическое образование, %</span>
        <input type="number" v-model="currentParams[1]" />
      </label>
      <label>
        <span>Косвенный опыт, мес</span>
        <input type="number" v-model="currentParams[2]" />
      </label>
      <label>
        <span>Пол, М</span>
        <input type="number" v-model="currentParams[3]" />
      </label>
      <label>
        <span>Город: не Минск</span>
        <input type="number" v-model="currentParams[4]" />
      </label>
      <label>
        <span>Лояльность к профессии, %</span>
        <input type="number" v-model="currentParams[5]" />
      </label>
      <label>
        <span>Личное впечатление, %</span>
        <input type="number" v-model="currentParams[6]" />
      </label>
      <label>
        <span>Не нуждающийся возраст (23-32) %</span>
        <input type="number" v-model="currentParams[7]" />
      </label>
    </div>

    <div class="iframe">
      <iframe :src="currentSrc" sandbox="allow-same-origin allow-scripts"></iframe>
    </div>

    <div class="result">
      <table>
        <tbody>
          <tr v-for="(row, i) in visibleRows" :key="i">
            <td v-for="(item, j) in row" :key="j" style="min-width: 100px">
              <a :href="item" v-if="j === 1" target="_blank">{{item}}</a>
              <span v-else>  {{item}}</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import "./assets/style.scss";
const storedRows = JSON.parse(localStorage.getItem("draft")) || [];
console.log(storedRows);
const list = require("../list.json");
console.log(list);

export default {
  name: "App",
  data() {
    return {
      list: list.filter(url => !storedRows.some(row => row[0] === url)),
      currentIndex: 0,
      currentParams: new Array(8).fill(0),
      rows: storedRows
    };
  },
  computed: {
    currentSrc() {
      return this.list[this.currentIndex];
    },

    visibleRows() {
      const getPoint = (arr, i) => {

        const experience = arr[1];
        const education = arr[2];
        const coExperience = arr[3];
        const isMale = arr[4];
        const isNotMinsk = arr[5];
        const jobLoyality = arr[6];
        const appearance = arr[7];
        const salaryRequests = arr[8];

        const experienceMark = Math.min(1, experience / 6);
        const educationMark = education / 100;
        const coExperiencemark = Math.min(1, coExperience ** (1/2) / 10);
        const isMaleMark = isMale;
        const isNotMinskMark = isNotMinsk;
        const jobLoyalityMark = jobLoyality / 100;
        const appearanceMark = appearance / 100;
        const salaryRequestsMark = salaryRequests / 100;

        const result = 
          3 * experienceMark +
          2 * educationMark +
          1 * coExperiencemark +
          1 * isMaleMark + 
          0.5 * isNotMinskMark + 
          5 * jobLoyalityMark +
          6 * appearanceMark + 
          5 * salaryRequestsMark;

        return Math.round(result * 100 * (i < 125 ? 1.05 : 1));  
      };
      return this.rows.map((row, i) => [i, ...row, getPoint(row, i)]).sort((a, b) => b[10] - a[10]);
    }
  },
  methods: {
    apply() {
      this.rows.push([this.currentSrc, ...this.currentParams]);
      this.currentParams = new Array(8).fill(0);
      if(this.currentIndex < this.list.length-1) {
        this.currentIndex++;
      } else {
        alert('Done!');
      }
      localStorage.setItem("draft", JSON.stringify(this.rows));
    }
  }
};

</script>