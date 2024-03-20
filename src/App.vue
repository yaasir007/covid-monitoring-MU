<script setup>
import Chart from 'primevue/chart';
import { ref, onMounted } from "vue";

const countryName = ref("");
const getCountryName = ref(false);
const totalCases = ref("");
const totalDeaths = ref("");
const totalRecovered = ref("");

const getCovidData = async () => {
  const url = `https://covid-19-tracking.p.rapidapi.com/v1/Mauritius`;
  const options = {
    method: 'GET',
    headers: {
      'X-RapidAPI-Key': '1da92de020msh88608772bd7f708p16ff04jsnc92b20273f45',
      'X-RapidAPI-Host': 'covid-19-tracking.p.rapidapi.com'
    }
  };

  try {
    const response = await fetch(url, options);
    const result = await response.json();
    countryName.value = result.Country_text;
    getCountryName.value = true;

    for (const [key, value] of Object.entries(result)) {
      if (key === "Total Cases_text") {
        totalCases.value = value;
      } else if (key === "Total Deaths_text") {
        totalDeaths.value = value;
      } else if (key === "Total Recovered_text") {
        totalRecovered.value = value;
      }
    }
  } catch (error) {
    console.error(error);
  }
}

onMounted(async () => {
    await getCovidData();
    chartData.value = setChartData();
    chartOptions.value = setChartOptions();
});

const chartData = ref();
const chartOptions = ref(null);

const setChartData = () => {
    const documentStyle = getComputedStyle(document.body);

    return {
        labels: ['Total Cases', 'Total Deaths', 'Total Recovered'],
        datasets: [
            {
                data: [totalCases.value, totalDeaths.value, totalRecovered.value],
                backgroundColor: [documentStyle.getPropertyValue('--cyan-500'), documentStyle.getPropertyValue('--orange-500'), documentStyle.getPropertyValue('--gray-500')],
                hoverBackgroundColor: [documentStyle.getPropertyValue('--cyan-400'), documentStyle.getPropertyValue('--orange-400'), documentStyle.getPropertyValue('--gray-400')]
            }
        ]
    };
};

const setChartOptions = () => {
    const documentStyle = getComputedStyle(document.documentElement);
    const textColor = documentStyle.getPropertyValue('--text-color');

    return {
        plugins: {
            legend: {
                labels: {
                    cutout: '60%',
                    color: textColor
                }
            }
        }
    };
};
</script>

<template>
  <div class="chart-container">
      <div v-if="getCountryName" class="country">Covid Cases in {{ countryName }}</div>
      <Chart type="doughnut" :data="chartData" :options="chartOptions" class="chart-content" />
  </div>
</template>

<style scoped>
.chart-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 3rem;
}

.country {
  font-size: 2rem;
  font-family: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif;
  font-weight: 600;
}

.chart-content {
  width: 35%;
}
</style>
