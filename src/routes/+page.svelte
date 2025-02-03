<script>
  import { writable } from 'svelte/store';
  import { SvelteFlow, Background, Controls } from '@xyflow/svelte';
 
  import '@xyflow/svelte/dist/style.css';
 
  const nodes = writable([
    {
      id: '1',
      position: { x: 0, y: 0 },
      data: { label: 'A' }
    },
    {
      id: '2',
      position: { x: 100, y: 100 },
      data: { label: 'B' }
    }
  ]);
 
  const edges = writable([{ id: '1-2', source: '1', target: '2' }]);
</script>
 
<main>
  <SvelteFlow {nodes} {edges}>
    <Background bgColor="rgba(255,255,255,0.25)" patternColor="red" />
    <Controls />
  </SvelteFlow>
</main>
 
<style>
  main {
    height: 100vh;
  }
</style>