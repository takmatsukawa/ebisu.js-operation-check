<template>
  <main class="container">
    <h1>Ebisu.js</h1>
    <p>
      <a href="https://fasiha.github.io/ebisu.js/"
        >https://fasiha.github.io/ebisu.js/</a
      >
    </p>
    <p>
      <label for="halfLife"
        >Your guess as to the half-life of any given fact:</label
      >
      <input id="halfLife" type="number" v-model.number="halfLife" />
    </p>
    <table>
      <thead>
        <tr>
          <th>Elapsed time</th>
          <th>Total elapsed time</th>
          <th>Model</th>
          <th>Successes of trials correct</th>
          <th>Total number of trials</th>
          <th>Predicted Recalls</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(elapsed, i) in elapseds" :key="i">
          <td>{{ elapsed.elapsed }}</td>
          <td>{{ totalElapseds[i] }}</td>
          <td>{{ models[i] }}</td>
          <td>{{ elapsed.successes }}</td>
          <td>{{ elapsed.total }}</td>
          <td>{{ predictedRecalls[i] }}</td>
        </tr>
      </tbody>
    </table>
    <hr />
    <form @submit.prevent="forwardTime">
      <h4>Advance clocks</h4>

      <label for="elapsed">Elapsed time</label>
      <input id="elapsed" type="number" v-model.number="newElapsedTime" />

      <label for="successes">Successes of trials correct</label>
      <input id="successes" type="number" v-model.number="successesCount" />

      <label for="total">Total number of trials</label>
      <input id="total" type="number" v-model.number="totalCount" />

      <button class="button button-outline" type="submit">Elapse</button>
    </form>
  </main>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import { defaultModel, predictRecall, updateRecall, Model } from "ebisu-js";

type Elapse = {
  elapsed: number;
  successes: number;
  total: number;
};

export default defineComponent({
  name: "App",
  setup: () => {
    const halfLife = ref(24);
    const elapseds = ref<Elapse[]>([{ elapsed: 0, successes: 0, total: 0 }]);

    const newElapsedTime = ref(1);
    const successesCount = ref(0);
    const totalCount = ref(0);

    const forwardTime = () => {
      elapseds.value.push({
        elapsed: newElapsedTime.value,
        successes: successesCount.value,
        total: totalCount.value,
      });
    };

    const models = computed(() =>
      elapseds.value.reduce(
        (prev: Model[], curr, i) => [
          ...prev,
          i == 0
            ? defaultModel(halfLife.value)
            : curr.total == 0
            ? prev[prev.length - 1]
            : updateRecall(
                prev[prev.length - 1],
                curr.successes,
                curr.total,
                curr.elapsed
              ),
        ],
        []
      )
    );

    const totalElapseds = computed(() =>
      elapseds.value.reduce(
        (total: number[], curr) => [
          ...total,
          (total.length ? total[total.length - 1] : 0) + curr.elapsed,
        ],
        []
      )
    );

    const predictedRecalls = computed(() =>
      elapseds.value.map((elapsed, i) =>
        predictRecall(models.value[i], elapsed.elapsed, true)
      )
    );

    return {
      halfLife,
      forwardTime,
      elapseds,
      models,
      totalElapseds,
      predictedRecalls,
      newElapsedTime,
      successesCount,
      totalCount,
    };
  },
});
</script>

<style scoped>
.container {
  margin-top: 5rem;
  margin-bottom: 10rem;
}
</style>
