<script lang="ts">
    import { 
      useSvelteFlow,      
      useEdges,
      useNodes } from '@xyflow/svelte';     

    const nodes = useNodes();
    const edges = useEdges();
    const { toObject } = useSvelteFlow();

    function saveJson() {
      console.log("nodes and edges")
      console.log( nodes);
      console.log( edges);


      let flowData = toObject();
      const jsonString = JSON.stringify(flowData);
      console.log(jsonString)
      const blob = new Blob([jsonString], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'svelte-flow-graph.json';
      a.click();
      URL.revokeObjectURL(url);
    } 


</script>

<button on:click={saveJson}> Save as Json</button>
