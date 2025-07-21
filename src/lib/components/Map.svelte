<script>
    import { onMount, getContext } from "svelte";
    const d3 = getContext("d3");

    onMount(async () => {
        const width = window.innerWidth;
        const height = window.innerHeight;
        const svg = d3.select("svg");
        const g = svg.append("g");

        const base = import.meta.env.BASE_URL || "/";
        const url = `${base}data/alameda-census-tracts.geojson`;
        // const url = `${base}data/holc-map.json`;

        const response = await fetch(url);
        if (!response.ok) {
            throw new Error(`Failed to fetch GeoJSON: ${response.statusText}`);
        }
        const data = await response.json();

        const projection = d3.geoIdentity().reflectY(true).fitSize([width, height], data);
        const path = d3.geoPath().projection(projection);

        g.selectAll("path.tract")
            .data(data.features)
            .enter()
            .append("path")
            .attr("class", "tract")
            .attr("d", path)
            .on("mouseover", (event, d) => {
                const id = d.properties.DIST_NAME;
                d3.select(event.currentTarget).attr("fill", "orange");
                tooltip.style("visibility", "visible").text(`Tract ${id}`);
            })
            .on("mousemove", (event) => {
                tooltip.style("top", event.pageY + 10 + "px").style("left", event.pageX + 10 + "px");
            })
            .on("mouseout", (event) => {
                d3.select(event.currentTarget).attr("fill", null);
                tooltip.style("visibility", "hidden");
            });

        const zoom = d3
            .zoom()
            .scaleExtent([1, 5])
            .on("zoom", (event) => {
                g.attr("transform", event.transform);
            });

        svg.call(zoom).on("wheel.zoom", null);

        function zoomOnClick(event, [x, y]) {
            event.stopPropagation();
            svg.transition()
                .duration(750)
                .call(
                    zoom.transform,
                    d3.zoomIdentity
                        .translate(width / 2, height / 2)
                        .scale(40)
                        .translate(-x, -y)
                    // d3.pointer(event)
                );
        }

        d3.select("#zoomIn").on("click", () => svg.transition().call(zoom.scaleBy, 1.5));
        d3.select("#zoomOut").on("click", () => svg.transition().call(zoom.scaleBy, 0.75));

        const tooltip = d3.select("body").append("div").style("position", "absolute").style("background", "rgba(255,255,255,0.9)").style("padding", "4px 8px").style("border", "1px solid #aaa").style("border-radius", "4px").style("visibility", "hidden").style("font", "12px sans-serif");
    });

    let zoomOnClick;
</script>

<button id="zoomIn" class="zoom-button" on:click={() => zoomOnClick}>Zoom In</button>
<button id="zoomOut" class="zoom-button" style="left:80px">Zoom Out</button>

<svg width="100%" height="100vh"></svg>
<div id="tooltip" style="position: absolute; visibility: hidden; background: white; border: 1px solid #ccc; padding: 4px;"></div>

<style>
    svg {
        width: 100vw;
        height: 100vh;
    }
    .tract {
        fill: steelblue;
        stroke: white;
        stroke-width: 0.5px;
    }
    .zoom-button {
        position: relative;
        top: 10px;
        left: 10px;
        padding: 6px;
        background: rgba(255, 255, 255, 0.8);
        border: 1px solid #ccc;
        cursor: pointer;
    }
</style>
