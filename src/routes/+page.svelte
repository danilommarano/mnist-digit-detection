<script>
  import { onMount } from "svelte";
  import * as tf from "@tensorflow/tfjs";
  import "@tensorflow/tfjs-converter";

  let model;
  let drawing = false;

  onMount(async () => {
    // Load the model from local files
    const modelURL = "model/model.json";
    const modelArchitecture = "model/group1-shard1of1";
    model = await tf.loadLayersModel(tf.io.browserFiles([modelURL, modelArchitecture]));
  });

  function predictDigit(canvas) {
    // Convert the drawing into a tensor
    const ctx = canvas.getContext("2d");
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    const pixels = new Array(canvas.width * canvas.height);
    for (let i = 0; i < data.length; i += 4) {
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      pixels[i / 4] = avg / 255.0;
    }
    const input = tf.tensor4d(pixels, [1, 28, 28, 1]);

    // Make the digit prediction
    const prediction = model.predict(input);

    // Get the prediction result
    const digit = prediction.argMax(1).dataSync()[0];

    return digit;
  }
</script>

<style>
  canvas {
    border: 1px solid #ccc;
    margin-top: 20px;
    cursor: crosshair;
  }
</style>

<main>
  <h1>Reconhecimento de Dígitos</h1>
  <p>Desenhe um dígito no quadro abaixo:</p>
  <canvas
    width="280"
    height="280"
    let:canvas
    on:mousemove={() => {
      if (drawing) {
        const ctx = canvas.getContext("2d");
        ctx.fillRect(event.offsetX, event.offsetY, 14, 14);
      }
    }}
    on:mousedown={() => (drawing = true)}
    on:mouseup={() => {
      drawing = false;
      const digit = predictDigit(canvas);
      alert("Dígito desenhado: " + digit);
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }}
    on:mouseout={() => (drawing = false)}
  ></canvas>
</main>
