<script lang="ts">
    import { Note, Chord } from "tonal";
    import * as Tone from "tone";

    export let keys: string[] = ['C', 'G', 'D', 'A', 'E', 'B', 'Gb', 'Db', 'Ab', 'Eb', 'Bb', 'F'];
  
    const radius: number = 150; // Circle radius
    const center: number = 200; // Center of the SVG
  
    let isDragging = false; // Whether the mouse is held down
    let startIndex: number | null = null; // Index of the current starting point
    let currentPosition: { x: number; y: number } | null = null; // Current cursor position
    let connections: { start: number; end: number }[] = []; // Store temporary connections during drag
    let hitCircles: string[] = []; // Circles hit during the drag
    let detectedChords: string[] | null = null;
  
    // Calculate positions of all keys
    const positions = keys.map((_, index) => ({
      x: center + radius * Math.cos((index * Math.PI) / 6 - Math.PI / 2),
      y: center + radius * Math.sin((index * Math.PI) / 6 - Math.PI / 2),
    }));
  
    // Start dragging from a point
    function handleMouseDown(index: number) {
      isDragging = true;
      startIndex = index;
      hitCircles = [keys[index]]; // Initialize hit circles with the first point
      detectedChords = null;
      connections = [];
    }
  
    // Update hover index during drag
    function handleMouseOver(index: number) {
      if (isDragging && startIndex !== null && startIndex !== index) {
        connections = [...connections, { start: startIndex, end: index }];
        startIndex = index; // Update startIndex to the new point
        hitCircles = [...hitCircles, keys[index]]; // Add the hit circle
      }
    }
  
    // End dragging and reset states
    function handleMouseUp() {
      if (!isDragging) return;

      isDragging = false;
      startIndex = null;
      currentPosition = null;

      if (hitCircles.length >= 1) {
        detectedChords = Chord.detect(hitCircles);
        const relativePitches = calculateOrderedPitches(hitCircles);

        const synth = new Tone.PolySynth(Tone.MonoSynth).toDestination();
        synth.triggerAttackRelease(relativePitches, "8n");
      }
    }
  
    // Update current position during drag
    function handleMouseMove(event: MouseEvent) {
      if (isDragging) {
        const svg = event.currentTarget as SVGElement;
        const { left, top } = svg.getBoundingClientRect();
        currentPosition = {
          x: event.clientX - left,
          y: event.clientY - top,
        };
      }
    }

    function calculateOrderedPitches(notes: string[]): string[] {
      const result: string[] = [];
      let currentOctave = 4; // Start at octave 4

      for (let i = 0; i < notes.length; i++) {
        const currentNote = notes[i];
        if (i === 0) {
          // First note gets the initial octave
          result.push(`${currentNote}${currentOctave}`);
          continue;
        }

        const previousMidi = Note.midi(result[i - 1])!; // Get MIDI value of the previous note
        const currentMidi = Note.midi(`${currentNote}${currentOctave}`)!; // Assume same octave initially

        // Adjust octave if the current note's pitch is less than or equal to the previous note
        if (currentMidi <= previousMidi) {
          currentOctave += 1;
        }

        result.push(`${currentNote}${currentOctave}`);
      }

      return result;
    }
  </script>

<svg
  width="400"
  height="400"
  viewBox="0 0 400 400"
  class="circle-of-fifths"
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
  on:mouseleave={() => {if (isDragging) handleMouseUp()}}>

  <!-- Outer circle -->
  <circle cx={center} cy={center} r={radius} fill="none" stroke="black" />

  <!-- Persisted connections -->
  {#each connections as { start, end }}
    <line
      x1={positions[start].x}
      y1={positions[start].y}
      x2={positions[end].x}
      y2={positions[end].y}
      stroke="red"
      stroke-width="2"
    />
  {/each}

  <!-- Hovering line -->
  {#if isDragging && startIndex !== null && currentPosition}
    <line
      x1={positions[startIndex].x}
      y1={positions[startIndex].y}
      x2={currentPosition.x}
      y2={currentPosition.y}
      stroke="gray"
      stroke-dasharray="4"
      stroke-width="2"
    />
  {/if}

  <!-- Dots for keys -->
  {#each keys as key, index}
    <circle
      data-dot
      cx={positions[index].x}
      cy={positions[index].y}
      r="10"
      fill="blue"
      on:mousedown={() => handleMouseDown(index)}
      on:mouseover={() => handleMouseOver(index)}
      style="cursor: pointer"
    />
    <text
      x={positions[index].x}
      y={positions[index].y + 20}
      text-anchor="middle"
      dominant-baseline="middle">
      {key}
    </text>
  {/each}
</svg>

<!-- Display hit circles below -->
<div class="hit-circles">
  <h3>Hit Circles:</h3>
  <p>
    {#if hitCircles.length > 0}
      {hitCircles.join(' ')}
      {#if detectedChords && detectedChords.length > 0}
        => {detectedChords.join(' / ')}
      {/if}
    {:else}
      No circles hit.
    {/if}
  </p>
</div>

<style>
  /* General body styles to make the entire background black */
  body {
    background-color: black; /* Full black background */
    margin: 0; /* Remove default margins */
    height: 100vh; /* Ensure full viewport height */
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .circle-of-fifths {
    margin: auto;
    display: block;
    background-color: black; /* Black SVG background */
  }

  /* Outer circle border (static circle representing the Circle of Fifths) */
  circle {
    fill: black; /* Black inner circle */
    stroke: white; /* White outer border */
  }

  /* Dots on the circle */
  circle[data-dot] {
    fill: white; /* White dots */
    stroke: none; /* No border for dots */
  }

  /* Text inside the circle */
  text {
    fill: white; /* White text for notes */
    font-family: Arial, sans-serif;
    font-size: 1.1vw;
  }

  /* Hovering line */
  line[stroke-dasharray] {
    stroke: #d3d3d3; /* Grayish white for dashed line */
    stroke-width: 2;
  }

  /* Solid line */
  line:not([stroke-dasharray]) {
    stroke: red; /* Solid red line for connections */
    stroke-width: 2;
  }

  /* Dots hover effect */
  circle[data-dot]:hover {
    fill: gray; /* Gray on hover for dots */
    cursor: pointer;
  }

  /* Hit circles and chords display */
  .hit-circles {
    margin-top: 20px;
    text-align: center;
    font-family: Arial, sans-serif;
    color: white; /* White text for below display */
    font-size: 2vw;
  }
</style>
