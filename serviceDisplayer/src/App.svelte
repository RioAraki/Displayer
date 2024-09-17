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

import ButtonEdge from './edges/BaseEdge.svelte';
import CustomEdgeMarker from './edges/CustomEdgeMarker.svelte';

import SaveJsonButton from './SaveJsonButton.svelte';
import ContextMenu from './ContextMenu.svelte';

import * as Constants from './Constants.svelte';

const nodeTypes = {
    'gui': GuiNode,
    'backend': BackendNode
};

const edgeTypes = {
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
            height: 50,
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
    console.log("change edge input");

    selectedEdge.update((edge) => {
        if (edge) {
            switch (key) {
                case "type":
                    edge.markerStart = value;
                    edge.style = `stroke-width: 2px; stroke: ${Constants.COLOR_MAP[value]}`;
                    edge.markerEnd = Constants.MARKER_END_TYPES[value];
                    break;
                case "data":
                case "link":
                case "rule":
                    edge.data = edge.data || {}; 
                    edge.data[key] = value;
                    
                    break;
                default:
                    console.warn(`Unhandled key: ${key}`);
            }

            edges.update((n) =>
                n.map((nd) => (nd.id === edge.id ? { ...nd, 
                    type: edge.type, 
                    data: { ...edge.data }, 
                    markerStart: edge.markerStart, 
                    style: edge.style,
                    markerEnd: edge.markerEnd
                } : nd)),
            );
        }
        return edge;
    });
}

function handleEdgeCreate(connection) {
  return {
    id: `edge-${connection.source}-${connection.target}`,
    deletable: true,
    source: connection.source,
    target: connection.target,
    animated: true,
    style: 'stroke-width: 2px; stroke: #FF6F00',
    markerStart: "kafka",
    data: {
        "data":"",
        "link":"",
        "rule":""
    },
    markerEnd: {
      type: MarkerType.ArrowClosed,
      width: 10,
      height: 10,
      color: '#FF6F00'
    },
    label: ``
  };
}

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

            <hr class="separator" />

            {#if $selectedEdge}
                <p>Selected Edge: {$selectedEdge.id}</p>
                <div class="inputGroup">
                    <label for="edgeType">Type:</label>
                    <select id="edgeType" on:change={(e) => handleEdgeInputChange("type", e.target.value)}>
                        <option value="kafka">Kafka</option>
                        <option value="https">Https</option>
                        <option value="redis">Redis</option>
                    </select>
                </div>

                {#each Object.entries($selectedEdge.data) as [key, value]}
                    <div class="inputGroup">
                        <label for={key}>{key}:</label>
                        <input
                            id={key}
                            type="text"
                            value={value}
                            on:input={(e) => handleEdgeInputChange(key, e.target.value)}
                        />
                    </div>
                {/each}
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
            onedgecreate={handleEdgeCreate}
            on:edgeclick={handleEdgeClick}
            fitView
            bind:this={instance}
        >
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
            <CustomEdgeMarker />
            <MiniMap />
        </SvelteFlow>
    </SvelteFlowProvider>
</div>

<style>
    .nodeEdgeEditor {
        position: absolute;
        top: 0;
        right: 0;
        width: 300px; /* Adjust width as needed */
        height: 100%;
        overflow-y: auto;
        padding: 10px;
        background-color: #f9f9f9;
        border-left: 1px solid #ddd;
        z-index: 999;
        box-shadow: -2px 0 5px rgba(0,0,0,0.1);
    }
    .inputGroup {
        margin-bottom: 10px;
    }

    .inputGroup label {
        margin-right: 10px;
        font-weight: bold;
    }

    .separator {
        margin: 20px 0;
        border: 0;
        border-top: 1px solid #ddd;
    }
</style>