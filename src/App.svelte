<script>
  "use strict";
  import { onMount } from "svelte";

  import * as ml5 from "ml5";
  import * as tf from "@tensorflow/tfjs";
  import * as Tone from "tone";
  import * as p5 from "p5";

  let synth = null;
  let videoSource = null;
  let loading = false;
  let result = null;

  onMount(async () => {
    console.log("Mounted");
  });

  async function startHandtracking() {
    // Start MIDI input and audio synth.
    navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
    synth = new Tone.PluckSynth().toDestination();
    Tone.start();

    // Start webcam.
    try {
      loading = true;
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { width: 1280, height: 720 },
      });
      videoSource.srcObject = stream;
      videoSource.play();
      loading = false;
    } catch (error) {
      console.log(error);
    }

    // Create a new handpose method.
    const options = {
      flipHorizontal: false, // boolean value for if the video should be flipped, defaults to false
      maxContinuousChecks: Infinity, // How many frames to go without running the bounding box detector. Defaults to infinity, but try a lower value if the detector is consistently producing bad predictions.
      detectionConfidence: 0.8, // Threshold for discarding a prediction. Defaults to 0.8.
      scoreThreshold: 0.75, // A threshold for removing multiple (likely duplicate) detections based on a "non-maximum suppression" algorithm. Defaults to 0.75
      iouThreshold: 0.3, // A float representing the threshold for deciding whether boxes overlap too much in non-maximum suppression. Must be between [0, 1]. Defaults to 0.3.
    };
    const handpose = ml5.handpose(videoSource, options, modelLoaded);

    // When the model is loaded
    function modelLoaded() {
      console.log("Model Loaded!");
    }

    // Listen to new 'hand' events.
    handpose.on("hand", (results) => {
      result = JSON.stringify(results, null, 2);

      if (results.length > 0) {
        // TODO Control in a fun way.
        const note = Tone.Frequency(60, "midi").toNote();
        const now = Tone.immediate();
        const velocity = 100;
        const duration = "16n";
        synth.triggerAttackRelease(note, duration, now, velocity);
      }
    });
  }

  let midi = null;
  function onMIDISuccess(midiAccess) {
    midi = midiAccess;
    startLoggingMIDIInput(midi);
  }

  function onMIDIFailure(msg) {
    console.error(`Failed to get MIDI access - ${msg}`);
  }

  function listInputsAndOutputs(midiAccess) {
    for (const entry of midiAccess.inputs) {
      const input = entry[1];
      console.log(
        `Input port [type:'${input.type}']` +
          ` id:'${input.id}'` +
          ` manufacturer:'${input.manufacturer}'` +
          ` name:'${input.name}'` +
          ` version:'${input.version}'`
      );
    }

    for (const entry of midiAccess.outputs) {
      const output = entry[1];
      console.log(
        `Output port [type:'${output.type}'] id:'${output.id}' manufacturer:'${output.manufacturer}' name:'${output.name}' version:'${output.version}'`
      );
    }
  }

  function onMIDIMessage(message) {
    var command = message.data[0];
    var midiNote = message.data[1];
    var velocity = message.data.length > 2 ? message.data[2] : 0;

    switch (command) {
      case 144:
        console.log("Note on");
        const note = Tone.Frequency(midiNote, "midi").toNote();
        const duration = "16n";
        const now = Tone.immediate();
        if (velocity > 0) {
          synth.triggerAttackRelease(note, duration, now, velocity);
        }
        break;
      case 128:
        console.log("Note off");
        break;
    }
  }

  function startLoggingMIDIInput(midiAccess) {
    listInputsAndOutputs(midiAccess);
    midiAccess.inputs.forEach((entry) => {
      entry.onmidimessage = onMIDIMessage;
    });
  }
</script>

<main>
  {#if loading}
    <h1>LOADING</h1>
  {:else}
    <h1>Hello {name}!</h1>
  {/if}
  <video id="video" bind:this={videoSource} />
  <button on:click={startHandtracking}>Start</button>

  <pre>{result}</pre>
</main>

<style>
  main {
    background: beige;
    padding: 1em;
  }

  video {
    background-color: lightgrey;
  }

  :global(body) {
    background-color: #f2eee2;
  }
</style>
