<script>
  import { writable } from 'svelte/store';
  import { SvelteFlow, Background, Controls } from '@xyflow/svelte';
  import '@xyflow/svelte/dist/style.css';

  // Store for nodes and edges
  const nodes = writable([]);
  const edges = writable([]);

  let nodeIdCounter = 1; // Unique ID counter for new nodes

  // Handle node dragging start
  function handleDragStart(event, type) {
    event.dataTransfer.setData("text/plain", type);
  }

  // Allow dropping on the canvas
  function allowDrop(event) {
    event.preventDefault();
  }

  // Handle node drop with accurate positioning
  function handleDrop(event) {
    event.preventDefault();

    const type = event.dataTransfer.getData("text/plain");
    const flowArea = event.currentTarget.getBoundingClientRect(); // Get canvas bounds

    // Adjust position relative to the flow container
    const position = {
      x: event.clientX - flowArea.left,
      y: event.clientY - flowArea.top
    };

    nodes.update(n => [...n, { id: `${nodeIdCounter++}`, position, data: { label: type } }]);
  }
</script>

<main>
  <!-- Left Panel for Dragging Elements -->
  <aside class="sidebar">
    <h3>Elements</h3>
    <div class="draggable" draggable="true" on:dragstart={(e) => handleDragStart(e, 'A')}>Node A</div>
    <div class="draggable" draggable="true" on:dragstart={(e) => handleDragStart(e, 'B')}>Node B</div>
    <!-- <div class="draggable" draggable="true" role="button" tabindex="0" on:dragstart={(e) => handleDragStart(e, 'B')}>Node A</div> -->
    <!-- <div class="draggable" draggable="true" role="button" tabindex="0" on:dragstart={(e) => handleDragStart(e, 'B')}>Node B</div> -->
  </aside>

  <!-- Flow Area (Drop Target) -->
  <section class="flow-area" on:drop={handleDrop} on:dragover={allowDrop}>
    <SvelteFlow {nodes} {edges}>
      <Background bgColor="rgba(255,255,255,0.25)" patternColor="red" />
      <Controls />
    </SvelteFlow>
  </section>
  <!-- <section class="flow-area" role="region" on:drop={handleDrop} on:dragover={allowDrop}>
    <SvelteFlow {nodes} {edges}>
      <Background bgColor="rgba(255,255,255,0.25)" patternColor="red" />
      <Controls />
    </SvelteFlow>
  </section> -->
</main>

<style>
  main {
    display: flex;
    height: 100vh;
  }

  .sidebar {
    width: 200px;
    background: #f4f4f4;
    padding: 10px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    border-right: 1px solid #ddd;
  }

  .draggable {
    padding: 10px;
    background: #007bff;
    color: white;
    text-align: center;
    border-radius: 5px;
    cursor: grab;
  }

  .flow-area {
    flex-grow: 1;
    position: relative;
  }
</style>
