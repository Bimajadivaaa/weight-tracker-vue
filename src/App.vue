<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import { Chart } from "chart.js/auto";

const weightChartElement = ref(null);

const weightsKey = "weights";

const savedWeights = JSON.parse(localStorage.getItem(weightsKey)) || [];

const weights = ref(savedWeights);

const weightChart = shallowRef(null);

const weightInput = ref();

const currentWeight = computed(() => {
  return (
    weights.value.sort(
      (firstWeight, nextWeight) => nextWeight.date - firstWeight.date
    )[0] || { weight: 0 }
  );
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
  savedWeightsTolocalStorage();
};
function savedWeightsTolocalStorage() {
  localStorage.setItem(weightsKey, JSON.stringify(weights.value));
}

watch(
  weights,
  (newWeights) => {
    updateChart(newWeights);
  },
  { deep: true }
);

onMounted(() => {
  if (weights.value.length > 0) {
    updateChart(weights.value);
  }
});
function updateChart(newWeights) {
  const ws = [...newWeights];

  if (weightChart.value) {
    weightChart.value.data.labels = ws
      .sort((firstWeight, nextWeight) => firstWeight.date - nextWeight.date)
      .map((w) => new Date(w.date).toLocaleDateString())
      .slice(-7);

    weightChart.value.data.datasets[0].data = ws
      .sort((firstWeight, nextWeight) => firstWeight.date - nextWeight.date)
      .map((w) => w.weight);

    weightChart.value.update();

    return;
  }

  nextTick(() => {
    weightChart.value = new Chart(weightChartElement.value.getContext("2d"), {
      type: "line",
      data: {
        labels: ws
          .sort((firstWeight, nextWeight) => firstWeight.date - nextWeight.date)
          .map((w) => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: "Berat Badan",
            data: ws
              .sort(
                (firstWeight, nextWeight) => firstWeight.date - nextWeight.date
              )
              .map((w) => w.weight),
            backgroundColor: "rgba(6, 214, 160, 0.5)",
            borderColor: "rgba(6, 214, 160, 0.5)",
            borderWidth: 1,
            fill: true,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
      },
    });
  });
}
</script>

<template>
  <main>
    <h1>Pelacak Berat Badan</h1>

    <div class="current">
      <span>{{ currentWeight.weight }}kg</span>
      <small>Berat Badan Saat ini</small>
      <div v-if="currentWeight.weight > 60">
        <br>
        <bold>Kamu harus diet.</bold>
        <p>🙄</p>
      </div>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" min="0.1" step="0.1" v-model="weightInput" placeholder="Berat badan kamu saat ini..."/>
      <input type="submit" value="Tambah" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Data Berat Badan 7 Hari Terakhir</h2>

      <div class="canvas-box">
        <canvas ref="weightChartElement"></canvas>
      </div>

      <div class="weight-history">
        <h2>Catatan Berat Badan</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>

    </div>
  </main>
</template>

<style></style>
