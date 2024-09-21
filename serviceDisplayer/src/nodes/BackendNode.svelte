<script lang="ts">
    import { Handle, Position } from '@xyflow/svelte';
    export let data: { name: string };
    export let selected;
</script>
 


<div class={`backend-node ${selected ? "selected-node" : ''}`}>
    <div class="backend-label">C++ Backend</div>
    <div class="backend-content">
        <strong>{data.name}</strong>
    </div>

    <div class="connector">
        <Handle id="value" type="target" position={Position.Top} />
        <div class="onHoverText">Upstream</div>
    </div>
    
    <div class="connector">
        <Handle id="value" type="source" position={Position.Bottom} />
        <div class="onHoverText">Downstream</div>
    </div>
</div>

<style>
    .backend-node {
        position: relative;
        padding: 1rem;
        border: 1.5px solid #42c44f;
        border-radius: 10px;
        font-size: 0.9rem;
        background: white;
    }

    .selected-node {
        position: relative;
        padding: 1rem;
        border: 2px solid #42c44f;
        border-radius: 10px;
        font-size: 1rem;
        background: white;
    }

    .backend-label {
        position: absolute;
        top: -0.75rem; 
        left: -4rem; 
        white-space: nowrap;
        background-color: white;
        padding: 0 0.5rem;
        /* font-weight: bold; */
        color: #003f5c;
    }

    .backend-content {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        text-align: center;
    }

    /*TODO: connector placement is a bit misaligned */
    .onHoverText {
        visibility: hidden;
        background-color: black;
        color: #fff;
        text-align: center;
        border-radius: 4px;
        padding: 0.25rem;
        position: absolute;
        z-index: 1;
        bottom: 125%;
        left: 75%;
        transform: translateX(-50%);
        opacity: 0;
        transition: opacity 0.3s;
        font-size: 0.6rem;
        white-space: nowrap;
    }

    .connector:hover .onHoverText {
        visibility: visible;
        opacity: 1;
    }

</style>