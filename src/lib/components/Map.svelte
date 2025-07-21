<script>
    import { onMount, getContext } from "svelte";
    import L from "leaflet";
    import "leaflet/dist/leaflet.css";
    const scrollama = getContext("scrollama");
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

        const main = d3.select("main");
        const scrolly = main.select("#scrolly-two");
        const figure = scrolly.select("figure");
        const article = scrolly.select("article");
        const step = article.selectAll(".step");

        // initialize the scrollama
        const scroller = scrollama();

        // generic window resize listener event
        function handleResize() {
            // 1. update height of step elements
            const stepH = Math.floor(window.innerHeight * 0.75);
            step.style("height", stepH + "px");

            const figureHeight = window.innerHeight / 2;
            const figureMarginTop = (window.innerHeight - figureHeight) / 2;

            figure.style("height", figureHeight + "px").style("top", figureMarginTop + "px");

            // 3. tell scrollama to update new element dimensions
            scroller.resize();
        }

        // scrollama event handlers
        function handleStepEnter(response) {
            // response = { element, direction, index }

            // add color to current step only
            step.classed("is-active", function (d, i) {
                return i === response.index;
            });

            // update graphic based on step
            // figure.select("p").text(response.index + 1);
        }

        function init() {
            // 1. force a resize on load to ensure proper dimensions are sent to scrollama
            handleResize();

            // 2. setup the scroller passing options
            // 		this will also initialize trigger observations
            // 3. bind scrollama event handlers (this can be chained like below)
            scroller
                .setup({
                    step: "#scrolly-two article .step",
                    offset: 0.33,
                    debug: false,
                })
                .onStepEnter(handleStepEnter);
        }

        // kick things off
        init();
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

        // draw overlay areas
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

<section id="scrolly-two" class="my-8 relative grid">
    <article class="w-full">
        <div class="step relative z-1 w-xs left-[35vw]" data-step="1">
            <p>STEP 1</p>
        </div>
        <div class="step relative z-1 w-xs right-[35vw]" data-step="2">
            <p>STEP 2</p>
        </div>
        <div class="step relative z-1 w-xs left-[35vw]" data-step="3">
            <p>STEP 3</p>
        </div>
        <div class="step relative z-1 w-xs right-[35vw]" data-step="4">
            <p>STEP 4</p>
        </div>
    </article>

    <figure class="flex items-center justify-center">
        <div id="map" class="relative w-screen h-screen">
            <svg id="overlay" class="absolute top-0 left-0 z-[1000]"></svg>
        </div>
    </figure>
</section>

<style>
    #scrolly-two > * {
        -webkit-box-flex: 1;
        -ms-flex: 1;
        flex: 1;
        grid-column: 1;
        grid-row: 1;
    }

    article {
        position: relative;
        padding: 0 1rem;
    }

    figure {
        position: -webkit-sticky;
        position: sticky;
        width: 100%;
        margin: 0;
        -webkit-transform: translate3d(0, 0, 0);
        -moz-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
        z-index: 0;
    }

    .step {
        margin: 0 auto 2rem auto;
        background-color: #3b3b3b;
        color: #fff;
    }

    .step:last-child {
        margin-bottom: 0;
    }

    .step.is-active {
        background-color: goldenrod;
        color: #3b3b3b;
    }

    .step p {
        text-align: center;
        padding: 1rem;
        font-size: 1.5rem;
    }
</style>
