import {
    MarkerType
} from '@xyflow/svelte';

<script context="module">
    export const KAFKA_COLOR = '#FF6F00';
    export const HTTPS_COLOR = '#007ACC';
    export const REDIS_COLOR = '#D82C20';
    export const COLOR_MAP = {
        kafka: KAFKA_COLOR,
        https: HTTPS_COLOR,
        redis: REDIS_COLOR
    };
    export const MARKER_END_TYPES = {
        kafka: {
            type: "arrowclosed",
            width: 10,
            height: 10,
            color: KAFKA_COLOR
        },
        https: {
            type: "arrowclosed",
            width: 10,
            height: 10,
            color: HTTPS_COLOR
        },
        redis: {
            type: "arrowclosed",
            width: 10,
            height: 10,
            color: REDIS_COLOR
        }
    }
</script>