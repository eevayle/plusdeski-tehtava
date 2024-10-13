<template>
  <div class="app">
    <h3>Plus-deskin ennakkotehtävä</h3>
    <h3>Eeva Sarlin</h3>
    <h2>Tutki, miten paljon kuntasi varhaiskasvatuksessa on vieraskielisiä lapsia</h2>
    <label for="kunta">Valitse kunta: </label>
    <select id="kunta" v-model="selectedKunta" @change="calculateSelectedKuntaPercentage">
      <option v-for="kunta in kuntas" :key="kunta.code" :value="kunta.code">
        {{ kunta.name }}
      </option>
    </select>
    <div class="results" v-if="percentage !== null">
      <p>
        <b>{{ selectedKuntaName }}: </b>
        <span v-if="percentage === 0">Ei vieraskielisiä lapsia varhaiskasvatuksessa.</span>
        <span v-else>{{ percentage.toFixed(2) }}%</span>
      </p>
      <div class="progress-bar">
        <div :style="{ width: `${percentage}%`, backgroundColor: '#7C59FA' }" class="progress"></div>
      </div>
      <p>
        <b>Kansallinen keskiarvo: </b> 
        <span>{{ nationalAverage.toFixed(2) }}%</span>
      </p>
      <div class="progress-bar">
        <div :style="{ width: `${nationalAverage}%`, backgroundColor: '#009CB5' }" class="progress"></div>
      </div>
      <p style="font-weight: 800;">Vertailun vuoksi muita relevantteja kuntia:</p>
      <div v-for="city in additionalCities" :key="city.name" class="progress-container">
        <div class="progress-bar">
          <div :style="{ width: `${city.percentage}%`, backgroundColor: city.color }" class="progress">
            <span class="progress-label other-kuntas">{{ city.name }}: {{ city.percentage.toFixed(2) }}%</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import jsonData from './data.json';

export default {
  data() {
    const alueLabels = jsonData.dataset.dimension.Alue.category.label;
    const values = jsonData.dataset.value;

    // Process kunta data
    const kuntas = Object.entries(alueLabels)
      .map(([code, name], index) => ({
        code,
        name,
        total: parseInt(values[index * 2], 10),
        foreign: parseInt(values[index * 2 + 1], 10)
      }))
      .slice(1); // Skip the national "SSS" entry

    // Additional cities for comparison
    const additionalCityCodes = ["KU092", "KU049", "KU091", "KU853"];
    const additionalCities = additionalCityCodes.map(code => {
      const city = kuntas.find(k => k.code === code);
      return {
        name: city.name,
        percentage: city.total > 0 ? (city.foreign / city.total) * 100 : 0,
        color: '#949697'
      };
    });

    return {
      kuntas,
      selectedKunta: null,
      percentage: null,
      nationalAverage: this.calculateNationalAverage(values),
      additionalCities
    };
  },
  computed: {
    selectedKuntaName() {
      const kunta = this.kuntas.find(k => k.code === this.selectedKunta);
      return kunta ? kunta.name : '';
    },
  },
  methods: {
    calculateSelectedKuntaPercentage() {
      const kuntaData = this.kuntas.find(k => k.code === this.selectedKunta);
      if (kuntaData) {
        this.percentage = kuntaData.total > 0 
          ? (kuntaData.foreign / kuntaData.total) * 100 
          : 0;
      }
    },
    calculateNationalAverage(values) {
      const total = parseInt(values[0], 10);
      const foreign = parseInt(values[1], 10);
      return total > 0 ? (foreign / total) * 100 : 0;
    }
  }
};
</script>

<style>
h3 {
  color: #694BD7;
}

select { 
  font-size: 15px;
  border: black 2px solid;
  border-radius: 4px;
  padding: 4px;
}

label {
  font-weight: 600;
} 

.app {
  padding: 20px;
}

.app p {
  margin-block-end: 0.5em;
}

.results {
  text-align: left;
}

.progress-container {
  margin-top: 10px;
}

.progress-bar {
  width: 100%;
  background-color: #e0e0e0;
  border-radius: 4px;
  position: relative;
}

.progress {
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  border-radius: 4px;
}

.progress-label {
  position: relative;
  padding-left: 0.2em;
  width: 100%;
  text-align: left;
  color: white;
  font-size: smaller;
}
</style>
