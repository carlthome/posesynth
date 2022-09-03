<script>
  import * as ml5 from "ml5";
  import * as tf from "@tensorflow/tfjs";

  console.log(ml5);

  import svelteLogo from "./assets/svelte.svg";
  import Counter from "./lib/Counter.svelte";

  const hello = tf.zeros([3]);
</script>

<main>
  <p>{hello}</p>
  <h1>Hello</h1>
  <div class="card">
    <Counter />
  </div>
</main>

<style>
</style>
