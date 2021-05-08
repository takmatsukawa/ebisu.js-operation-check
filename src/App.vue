<template>
  <main class="container">
    <h1>How to use Ebisu.js</h1>
    <p>
      <a href="https://fasiha.github.io/ebisu.js/">Ebisu.js</a> is a JavaScript
      library to help deal with forgetting curves.
    </p>
    <pre><code>import { defaultModel, predictRecall, updateRecall } from 'ebisu-js';
// <strong><a href="#log0"><u>t=0</u></a></strong> Initialize the model
let model = defaultModel({{halfLife}});
// After <strong><a href="#log1"><u>t=1</u></a></strong>
console.log(predictRecall(model, 1, true));
// ...
// After <strong><a href="#log11"><u>t=11</u></a></strong>, 1 success of 1 trial
model = updateRecall(model, 1, 1 11);
// ...
// After <strong><a href="#log21"><u>t=10</u></a></strong> since last `updateRecall`, unsuccess of 1 trial
model = updateRecall(model, 0, 1 10);
</code></pre>
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
          <th>Model</th>
          <th>Successes of trials correct</th>
          <th>Total number of trials</th>
          <th>Predicted Recalls</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(log, i) in logs" :key="i" :id="`log${i}`">
          <td>{{ log[0] }}</td>
          <td>{{ log[1] }}</td>
          <td :style="{ backgroundColor: log[2] > 0 ? '#ff572247' : '' }">
            {{ log[2] }}
          </td>
          <td :style="{ backgroundColor: log[3] > 0 ? '#ff572247' : '' }">
            {{ log[3] }}
          </td>
          <td>{{ log[4] }}</td>
        </tr>
      </tbody>
    </table>
  </main>
  <footer class="container">
    <a
      class="github"
      href="https://github.com/takmatsukawa/ebisu.js-operation-check"
    >
      <img src="/GitHub-Mark-32px.png" alt="GitHub" width="32" height="32" />
    </a>
  </footer>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import { defaultModel, predictRecall, updateRecall, Model } from "ebisu-js";

export default defineComponent({
  name: "App",
  setup: () => {
    const halfLife = ref(10);
    const logs = computed(() => {
      let logs: [number, Model, number, number, number][] = [];
      let model = defaultModel(halfLife.value);

      let totalEpalsed = 0;
      for (let i = 0; i < 48; i++, totalEpalsed++) {
        let successes = 0;
        let total = 0;

        if (i == 11) {
          successes = 1;
          total = 1;
          model = updateRecall(model, successes, total, totalEpalsed);
          totalEpalsed = 0;
        }

        if (i == 21) {
          successes = 0;
          total = 1;
          model = updateRecall(model, successes, total, totalEpalsed);
          totalEpalsed = 0;
        }

        logs.push([
          i,
          model,
          successes,
          total,
          predictRecall(model, totalEpalsed, true),
        ]);
      }
      return logs;
    });

    return {
      halfLife,
      logs,
    };
  },
});
</script>

<style scoped>
main {
  margin-top: 5rem;
  margin-bottom: 5rem;
}

footer {
  height: 10rem;
}

.github {
  float: right;
}
</style>
