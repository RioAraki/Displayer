<script lang="ts">
    import { useSvelteFlow, useNodes, useEdges } from '@xyflow/svelte';     
    import { get } from 'svelte/store';

    const nodes = useNodes();
    const edges = useEdges();

    let fileInput: HTMLInputElement;

    // Function to load nodes and edges from the uploaded JSON file
    function handleFileUpload(event) {
        const file = event.target.files[0];
        if (file) {
            const reader = new FileReader();

            reader.onload = (e) => {
                try {
                    const data = JSON.parse(e.target.result as string);
                    const { nodes: loadedNodes, edges: loadedEdges } = data;

                    if (loadedNodes && loadedEdges) {
                        nodes.set(loadedNodes);
                        edges.set(loadedEdges);
                    } else {
                        console.error('Invalid JSON format: missing nodes or edges');
                    }
                } catch (error) {
                    console.error('Error parsing JSON file:', error);
                }
            };

            reader.readAsText(file);
        }
    }

    // Trigger the file input dialog when clicking the button
    function triggerFileUpload() {
        fileInput.click();
    }
</script>

<!-- Hidden input field to trigger file upload -->
<input type="file" accept=".json" on:change={handleFileUpload} bind:this={fileInput} style="display: none;" />

<!-- Button to trigger file upload -->
<button on:click={triggerFileUpload}>Load from Json</button>
