<script lang="ts">
import {
    writable
} from 'svelte/store';
import {
    SvelteFlow,
    SvelteFlowProvider,
    Controls,
    Background,
    BackgroundVariant,
    MiniMap, 
    MarkerType,
    type Node,
    type Edge,
} from '@xyflow/svelte';

import '@xyflow/svelte/dist/style.css';

import GuiNode from './nodes/GuiNode.svelte';
import BackendNode from './nodes/BackendNode.svelte';

import KafkaEdge from './edges/KafkaEdge.svelte';
import ButtonEdge from './edges/ButtonEdge.svelte';

import SaveJsonButton from './SaveJsonButton.svelte';
import ContextMenu from './ContextMenu.svelte';

const nodeTypes = {
    'gui': GuiNode,
    'backend': BackendNode
};

const edgeTypes = {
    'kafka': KafkaEdge,
    'button': ButtonEdge
    // 'https': HttpsNode,
    // 'redis': RedisNode
};

const nodes = writable < Node[] > ([]);
const edges = writable < Edge[] > ([]);

let menu: {
    id: string;top ? : number;left ? : number;right ? : number;bottom ? : number
} | null;
let width: number;
let height: number;
let instance;
let selectedNode = writable<string | null>(null);
let selectedEdge = writable<string | null>(null);

// Function to add a new node
function addGuiNode() {
    nodes.update((n) => {
        const newNodeId = `GUI_${(n.length+1)}`
        var newNode: Node = {
            id: newNodeId,
            type: 'gui',
            data: {
                name: "default gui",
            },
            position: {
                x: Math.round(Math.random() * 100),
                y: Math.round(Math.random() * 100)
            },
            width: 150,
            height: 100,
        };
        return [...n, newNode];
    });
}

function addBackendNode() {
    nodes.update((n) => {
        const newNodeId = `Backend_${(n.length+1)}`
        var newNode: Node = {
            id: newNodeId,
            type: 'backend',
            data: {
                name: "default backend",
            },
            position: {
                x: Math.round(Math.random() * 100),
                y: Math.round(Math.random() * 100)
            },
            width: 150,
            height: 100,
        };
        return [...n, newNode];
    });
}

function handleContextMenu({ detail: { event, node } }) {
    // Prevent native context menu from showing
    event.preventDefault();
    menu = {
        id: node.id,
        top: event.clientY < height - 200 ? event.clientY : undefined,
        left: event.clientX < width - 200 ? event.clientX : undefined,
        right: event.clientX >= width - 200 ? width - event.clientX : undefined,
        bottom: event.clientY >= height - 200 ? height - event.clientY : undefined
    };
    console.log("menu?", menu)
}

// Close the context menu if it's open whenever the window is clicked.
function handlePaneClick() {
  menu = null;
}

function handleNodeClick({ detail: { node } }) {
  selectedNode.set(node);
}

function handleEdgeClick({ detail: { edge } }) {
  console.log("click edge")  
  selectedEdge.set(edge);
}

function handleNodeInputChange(key: string, value: string) {
    selectedNode.update((node) => {
        if (node) {
            node.data[key] = value;
            nodes.update((n) =>
                n.map((nd) => (nd.id === node.id ? { ...nd, data: { ...node.data } } : nd))
            );
        }
        return node;
    });
}

function handleEdgeInputChange(key: string, value: string) {
    selectedEdge.update((edge) => {
        if (edge) {
            edge.data = {}
            edge.data[key] = value;
            edges.update((n) =>
                n.map((nd) => (nd.id === edge.id ? { ...nd, data: { ...edge.data }, animated:true, type: "button"} : nd)),
            );
        }
        return edge;
    });
}

function handleEdgeCreate(connection) {
  console.log("create marker")
  return {
    id: `edge-${connection.source}-${connection.target}`,
    type: "button",
    label: "default",
    deletable: true,
    source: connection.source,
    target: connection.target,
    animated: true,
    markerStart: MarkerType.ArrowClosed,
  };
}

const handleNodeDragStop = (event) => {
  let node = event.detail.targetNode;
  selectedNode.set(node);

  node.position = {
    x: Math.round(node.position.x / 50) * 50,
    y: Math.round(node.position.y / 50) * 50
  };

  nodes.update((n) =>
      n.map((nd) => (nd.id === node.id ? { ...nd, position: { ...node.position } } : nd))
  );
};



</script>

<div style="height:100vh;" bind:clientWidth={width} bind:clientHeight={height}>

    <SvelteFlowProvider>
        <button on:click={addGuiNode}>Add GUI Node</button>
        <button on:click={addBackendNode}>Add Backend Node</button>

        <div class="nodeEdgeEditor">
            {#if $selectedNode}
                <p>Selected Node: {$selectedNode.id}</p>
                {#each Object.entries($selectedNode.data) as [key, value]}
                    <div class="inputGroup">
                        <label for={key}>{key}:</label>
                        <input
                            id={key}
                            type="text"
                            value={value}
                            on:input={(e) => handleNodeInputChange(key, e.target.value)}
                        />
                    </div>
                {/each}
                <p>Position x: {$selectedNode.position.x}, y: {$selectedNode.position.y}</p>
            {:else}
                No selected Node.
            {/if}

            {#if $selectedEdge}
              <p>Selected Edge: {$selectedEdge.id}</p>
              <div class="inputGroup">
                  <label for="edgeType">Type:</label>
                  <select id="edgeType" on:change={(e) => handleEdgeInputChange("type", e.target.value)}>
                      <option value="Kafka">Kafka</option>
                      <option value="Https">Https</option>
                      <option value="Redis">Redis</option>
                  </select>
              </div>
            {:else}
                No selected Edge.
            {/if}
        </div>
        <SaveJsonButton />
        <SvelteFlow
        {nodeTypes}
        {edgeTypes}
        {nodes}
        {edges}
        on:nodecontextmenu={handleContextMenu}
        on:paneclick={handlePaneClick}
        on:nodeclick={handleNodeClick}
        on:nodedragstop={handleNodeDragStop}
        onedgecreate={handleEdgeCreate}
        on:edgeclick={handleEdgeClick}
        fitView
        bind:this={instance}>
            <Controls />
            <Background variant={BackgroundVariant.Lines} gap=50/>
            {#if menu}
                <ContextMenu
                    onClick={handlePaneClick}
                    id={menu.id}
                    top={menu.top}
                    left={menu.left}
                    right={menu.right}
                    bottom={menu.bottom}
                />
            {/if}
            <MiniMap />
        </SvelteFlow>
    </SvelteFlowProvider>
</div>

<style>
    .nodeEdgeEditor {
        margin-top: 20px;
        padding: 10px;
        background-color: #f9f9f9;
        border: 1px solid #ddd;
        z-index: 999;
    }

    .inputGroup {
        margin-bottom: 10px;
    }

    .inputGroup label {
        margin-right: 10px;
        font-weight: bold;
    }
</style>