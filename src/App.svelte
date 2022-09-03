<script>
  "use strict";
  import { onMount } from "svelte";

  import * as ml5 from "ml5";
  import * as tf from "@tensorflow/tfjs";
  import * as Tone from "tone";

  let synth = null;

  onMount(async () => {
    navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
    synth = new Tone.Synth().toDestination();
  });

  const hello = tf.zeros([3]);

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

    // a velocity value might not be included with a noteOff command
    var velocity = message.data.length > 2 ? message.data[2] : 0;

    switch (command) {
      case 144: // note on
        const note = Tone.Frequency(midiNote, "midi").toNote();
        const duration = "8n";
        console.log(note);
        synth.triggerAttackRelease(note, duration);
        break;
      case 128: // note off
        console.log("NOTE OFF");
        break;
    }
  }

  function startLoggingMIDIInput(midiAccess) {
    midiAccess.inputs.forEach((entry) => {
      entry.onmidimessage = onMIDIMessage;
    });
  }
</script>

<main>
  <p>{hello}</p>
  <h1>Hello</h1>
</main>

<style>
</style>
