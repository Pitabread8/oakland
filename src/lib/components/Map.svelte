<script>
    import { onMount, getContext } from "svelte";
    import L from "leaflet";
    import "leaflet/dist/leaflet.css";
    const d3 = getContext("d3");

    let map;
    let data;

    onMount(async () => {
        // create map centered on oakland
        map = L.map("map").setView([37.81, -122.23], 12);
        L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            attribution: "&copy; OpenStreetMap contributors",
        }).addTo(map);

        map.scrollWheelZoom.disable();
        map.dragging.disable();

        // fetch geojson data
        const base = import.meta.env.BASE_URL || "/";
        const res = await fetch(`${base}data/2020-tract-crosswalk.geojson`);
        // const res = await fetch(`${base}data/alameda-census-tracts.geojson`);
        // const res = await fetch(`${base}data/holc-map.json`);
        data = await res.json();

        drawOverlay();
        map.on("zoomend moveend", drawOverlay);
    });

    function drawOverlay() {
        const svg = d3.select("#overlay");
        svg.selectAll("*").remove();
        // resize overlay with zoom
        svg.attr("width", map.getSize().x).attr("height", map.getSize().y);

        const g = svg.append("g");

        // convert every x, y geojson point to leaflet pixel coordinate
        const transform = d3.geoTransform({
            point: function (x, y) {
                const point = map.latLngToLayerPoint(new L.LatLng(y, x));
                this.stream.point(point.x, point.y);
            },
        });
        const path = d3.geoPath().projection(transform);

        console.log(data.features);
        g.selectAll("path")
            .data(data.features)
            .enter()
            .append("path")
            .attr("class", "area")
            .attr("d", path)
            .attr("stroke", "#000")
            .attr("fill", (d) => d.properties.fill)
            .attr("fill-opacity", 0.5)
            .style("pointer-events", "all");
    }
</script>

<div id="map" class="relative w-screen h-screen">
    <svg id="overlay" class="absolute top-0 left-0 z-[1000]"></svg>
</div>
