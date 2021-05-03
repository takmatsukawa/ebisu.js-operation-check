<template>
  <main class="container">
    <h1>Ebisu.js</h1>
    <p>
      <a href="https://fasiha.github.io/ebisu.js/"
        >https://fasiha.github.io/ebisu.js/</a
      >
    </p>
    <p>Your guess as to the half-life of any given fact: {{ halfLife }}</p>
    <pre><code>import {defaultModel, predictRecall, updateRecall} from 'ebisu-js';
let t = {{halfLife}};
let model = defaultModel(t);
</code></pre>
    <p>model: {{ model }}</p>

    <section v-for="(elapsed, i) in elapseds" :key="i">
      <hr />
      <p>
        Elapsed: {{ elapsed.elapsed }} Successes:
        {{ elapsed.successes }} Total:{{ elapsed.total }}
      </p>
      <pre><code>let elapsed = {{elapsed.elapsed}};
let predictedRecall = predictRecall(model, elapsed, true);
</code></pre>
      <p>
        model: {{ models[i] }}<br />
        predictedRecall: {{ predictedRecalls[i] }}
      </p>
    </section>
    <button class="button button-outline" type="button" @click="forwardTime">
      Elapse
    </button>
  </main>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import { defaultModel, predictRecall, updateRecall } from "ebisu-js";

type Elapse = {
  elapsed: number;
  successes: number;
  total: number;
};

export default defineComponent({
  name: "App",
  setup: () => {
    const halfLife = ref(24);
    const model = ref(defaultModel(halfLife.value));
    const elapseds = ref<Elapse[]>([]);

    const forwardTime = () => {
      elapseds.value.push({
        elapsed: 1,
        successes: elapseds.value.length
          ? elapseds.value[elapseds.value.length - 1].successes
          : 0,
        total: elapseds.value.length
          ? elapseds.value[elapseds.value.length - 1].total
          : 0,
      });
    };

    const models = computed(() =>
      elapseds.value
        .reduce(
          (prev, curr) => [
            ...prev,
            updateRecall(
              prev[prev.length - 1],
              curr.successes,
              curr.total,
              curr.elapsed
            ),
          ],
          [model.value]
        )
        .slice(1)
    );
    const predictedRecalls = computed(() =>
      elapseds.value.map((elapsed, i) =>
        predictRecall(models.value[i], elapsed.elapsed, true)
      )
    );

    return { halfLife, model, forwardTime, elapseds, models, predictedRecalls };
  },
});
</script>

<style scoped>
.container {
  margin-top: 5rem;
  margin-bottom: 10rem;
}
</style>
