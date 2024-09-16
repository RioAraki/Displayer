<script lang="ts">
    import { Handle, Position } from '@xyflow/svelte';
    export let data: { name: string };
</script>
 


<div class="gui-node">
    <div class="gui-label">C# GUI</div>
    <div class="gui-content">
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
    .gui-node {
        position: relative;
        padding: 1rem;
        border: 1px solid #00aeff;
        border-radius: 10px;
        font-size: 0.9rem;
        background: white;
    }

    .gui-label {
        position: absolute;
        top: -0.75rem; 
        left: -1.5rem; 
        white-space: nowrap;
        background-color: white;
        padding: 0 0.5rem;
        /* font-weight: bold; */
        color: #003f5c;
    }

    .gui-content {
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