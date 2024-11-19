<script lang="ts">
    import { Chord } from "tonal";
    import * as Tone from "tone";

    export let keys: string[] = ['C', 'G', 'D', 'A', 'E', 'B', 'F#', 'C#', 'G#', 'D#', 'A#', 'F'];
  
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
      isDragging = false;
      startIndex = null;
      currentPosition = null;
      connections = []; // Reset the connections
      detectedChords = Chord.detect(hitCircles);

      // Play notes
      const synth = new Tone.PolySynth(Tone.MonoSynth).toDestination();
      synth.triggerAttackRelease(["F#4", "A#4", "C#5", "F5"], "8n");
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
  </script>

<svg
  width="400"
  height="400"
  viewBox="0 0 400 400"
  class="circle-of-fifths"
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
  on:mouseleave={handleMouseUp}>

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
  .circle-of-fifths {
    margin: auto;
    display: block;
  }
  circle:hover {
    fill: darkblue;
  }
  .hit-circles {
    margin-top: 20px;
    text-align: center;
    font-family: Arial, sans-serif;
  }
  .hit-circles ul {
    list-style: none;
    padding: 0;
  }
  .hit-circles li {
    display: inline;
    margin: 0 10px;
    font-weight: bold;
  }
</style>
