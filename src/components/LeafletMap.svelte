<script>
    import {createEventDispatcher, setContext} from 'svelte';
    import L from 'leaflet';
    import 'leaflet.markercluster'
    import 'leaflet.markercluster/dist/MarkerCluster.css'
    import 'leaflet.markercluster/dist/MarkerCluster.Default.css'

    import EventBridge from '../lib/EventBridge';

    export let options = {};
    export let events = [];
    export let clusterGroupOptions = undefined

    let map = null;
    let clusterGroup = clusterGroupOptions ? L.markerClusterGroup(clusterGroupOptions, { chunkedLoading: true }) : null

    $: {
        if(clusterGroupOptions !== undefined && options.maxZoom === undefined) {
            options.maxZoom = 19
        }
    }

    setContext(L, {
        getMap: () => map,
        getClusterGroup: () => clusterGroup
    });

    const dispatch = createEventDispatcher();
    let eventBridge;

    function initialize(container) {
        map = L.map(container, options);
        map.addLayer(clusterGroup)

        eventBridge = new EventBridge(map, dispatch, events);
        return {
            destroy: () => {
                eventBridge.unregister();
                map.remove();
            },
        };
    }

    export function getMap() {
        return map;
    }
</script>

<div style="height:100%; width:100%;" use:initialize>
    {#if map}
        <slot/>
    {/if}
</div>
