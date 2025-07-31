<script>
    import { onMount, getContext } from "svelte";
    import L from "leaflet";
    import "leaflet/dist/leaflet.css";
    const scrollama = getContext("scrollama");
    const d3 = getContext("d3");
    import ImageEnlarger from "./ImageEnlarger.svelte";

    let map;
    let data;
    let descData;
    let activeGrade = null;
    let index = 0;
    let activeAreaData = null;
    let quote = null;

    onMount(async () => {
        // create map centered on oakland
        map = L.map("map", { zoomControl: false }).setView([37.81, -122.4], 11);
        L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            attribution: "&copy; OpenStreetMap contributors",
        }).addTo(map);

        map.scrollWheelZoom.disable();
        map.dragging.disable();

        // fetch geojson data
        const base = import.meta.env.BASE_URL || "/";
        const res = await fetch(`${base}data/2020-tract-crosswalk.geojson`);
        data = await res.json();

        // fetch area descriptions data
        const descRes = await fetch(`${base}data/area-descriptions.json`);
        descData = await descRes.json();

        // drawOverlay();
        map.on("moveend", drawOverlay);

        // initialize the scrollama
        const scroller = scrollama();

        // generic window resize listener event
        function handleResize() {
            // tell scrollama to update new element dimensions
            scroller.resize();
        }

        // scrollama event handlers
        function handleStepEnter(response) {
            // response = { element, direction, index }

            const svg = d3.select("#overlay");
            svg.selectAll("*").remove();
            index = response.index;

            const steps = [null, { coords: [37.81, -122.2], zoom: 12 }, { coords: [37.81, -122.27], zoom: 11.9 }, { coords: [37.83, -122.21], zoom: 12.8 }, { coords: [37.78, -122.24], zoom: 13 }, { coords: [37.81, -122.23], zoom: 11.7 }, { coords: [37.81, -122.27], zoom: 12 }, { coords: [37.79, -122.23], zoom: 13 }];

            const step = steps[index];
            if (step) {
                map.flyTo(step.coords, step.zoom, {
                    animate: true,
                    duration: 0.8,
                    easeLinearity: 0.25,
                });
            } else {
                map.flyTo([37.81, -122.4], 11, {
                    animate: true,
                    duration: 0.8,
                    easeLinearity: 0.25,
                });
                map.once("moveend", () => {
                    svg.selectAll("*").remove();
                });
            }
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

        const tooltip = d3.select("#tooltip");

        // draw overlay areas
        g.selectAll("path")
            .data(data.features)
            .enter()
            .append("path")
            .attr("class", "area")
            .attr("d", path)
            .attr("stroke", "#000")
            .attr("fill", (d) => d.properties.fill || "red")
            .attr("fill-opacity", 0.5)
            .style("pointer-events", "all")
            .style("cursor", "pointer")
            .on("mouseover", function (event, d) {
                activeAreaData = descData[d.properties.area_id];
                if (index === 1) {
                    d3.selectAll("path.area")
                        .filter((d) => d3.select(this).attr("fill") === d.properties.fill)
                        .attr("stroke-width", 1.5)
                        .attr("fill-opacity", 0.6);
                    d3.selectAll("path.area")
                        .filter((d) => d3.select(this).attr("fill") != d.properties.fill)
                        .attr("fill-opacity", 0.2);
                } else {
                    tooltip.style("display", "block").text(`Area: ${d.properties.label} | Grade: ${d.properties.grade || "N/A"} | Tract: #${d.properties.GEOID.slice(-6)}`);
                }
                activeGrade = d3.select(this).attr("fill");
                d3.select(this).attr("fill-opacity", 0.8);
                quote = activeAreaData ? pickQuote(activeAreaData) : "";
            })
            .on("mousemove", function (event) {
                tooltip.style("left", event.pageX + 10 + "px").style("top", event.pageY - 28 + "px");
            })
            .on("mouseout", function () {
                tooltip.style("display", "none");
                d3.selectAll("path.area").attr("stroke-width", 1);
                d3.selectAll("path.area").attr("fill-opacity", 0.5);
                activeGrade = null;
                activeAreaData = null;
                quote = null;
            });
    }

    const grades = {
        "#76a865": {
            title: 'A ("Best")',
            description: "Typically affluent, white neighborhoods with new or well-maintained housing.",
        },
        "#7cb5bd": {
            title: 'B ("Still Desirable")',
            description: "Generally considered stable and were often inhabited by middle-class residents.",
        },
        "#ffff00": {
            title: 'C ("Definitely Declining")',
            description: "Characterized by older housing, and mostly inhabited by racial minorities.",
        },
        "#d9838d": {
            title: 'D ("Hazardous")',
            description: "Deemed high-risk areas for investment, residents were predominantly people of color.",
        },
        "#000000": {
            title: '"Industrial and Commercial"',
            description: "Not a grade, but was marked on the HOLC map in gray.",
        },
    };

    function pickQuote(data) {
        const text = data["clarifying_remarks"] || "";
        // const expressions = [/infiltration[^.;]*/i, /racial[^.;]*/i, /undesirables[^.;]*/i, /Negro[^.;]*/i, /Oriental[^.;]*/i, /Asiatic[^.;]*/i, /Mexican[^.;]*/i, /laborer[^.;]*/i, /foreign[^.;]*/i];
        const expressions = [/(?<=^|(?<=\. ))[^.]*\binfiltration\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\bracial\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\bundesirables?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\bnegro(es)?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\borientals?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\basiatics?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\bmexicans?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\blaborer(s)?\b[^.]*\./i, /(?<=^|(?<=\. ))[^.]*\bforeign\b[^.]*\./i];

        for (const regex of expressions) {
            const match = text.match(regex);
            if (match) return match[0].trim();
        }

        return;
    }
</script>

<section id="scrolly-two" class="my-8 relative grid">
    <article class="w-full relative px-2 space-y-32">
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:right-[30vw] xl:right-[35vw]" data-step="1">
            <p>This is Oakland.</p>
            <p>Oakland sits just east of San Francisco, and is home for over 400,000 people.</p>
            <div class="flex flex-row gap-2 items-baseline">
                <div>
                    <img src="images/graffiti.jpg" alt="graffiti wall" />
                    <p class="text-center mt-4 text-xs">Getty Images/UIG.</p>
                </div>
                <div>
                    <img src="images/bay-bridge.jpg" alt="bay bridge" />
                    <p class="text-center mt-4 text-xs">Tyler Casey/Unsplash.</p>
                </div>
            </div>
            <p>With a rich history dating almost two centuries, the city is known for its cultural vibrancy, art community, and political activism.</p>
            <img src="images/black-panthers.jpg" alt="black panther party" />
            <p class="text-center text-xs">David Fenton/Getty Images.</p>
            <img src="images/mural.jpg" alt="mural" />
            <p class="text-center text-xs">&quot;Women of the Black Panther Party&quot; by Rachel Wolfe-Goldsmith (2021).</p>
            <p>Yet beneath its progressive reputation lies a legacy of inequality.</p>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:left-[30vw] xl:left-[35vw]" data-step="2">
            <p>The Home Owners' Loan Corporation (HOLC) was established in 1993 by President Franklin D. Roosevelt, under the New Deal. In 1937, they commissioned a report on the city of Oakland and the surrounding area.</p>
            <p>HOLC split Oakland into several neighborhoods and gave each of them a mortgage risk grade based on its neighborhoods and racial demographics.</p>
            <ImageEnlarger src="images/holc-oakland.jpg" alt="holc map" caption={'<span class="font-bold">Thomas Bros.; Home Owners\' Loan Corporation (1937).</span> Note: click on images to enlarge them.'} />
            {#if grades[activeGrade]}
                <div class="border-2 p-2" style="border-color: {activeGrade}">
                    <h3 class="text-lg font-bold" style="color: {activeGrade}">{grades[activeGrade].title}</h3>
                    <p>{grades[activeGrade].description}</p>
                </div>
            {:else}
                <div class="border-2 p-2">
                    <h3 class="text-lg font-bold">Risk Grades</h3>
                    <p>Hover over each HOLC-designated area in the large map to learn more about the grades.</p>
                </div>
            {/if}
            <p>Areas with more people of color, especially Black people, resulted in a lower rating, which meant that residents would be denied loans and investments, decreasing their economic opportunities and access to homeownership, business equity, and other forms of household wealth.</p>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:right-[30vw] xl:right-[35vw]" data-step="3">
            <p>The 1937 map is overlaid on a modern map of Oakland and the surrounding Alameda county. Hover over each HOLC-designated area to find its grade and the 2020 census tract it falls into now.</p>
            <p>A short report was filed for each area, with details about the environment, residents, and other observations. These documents provide insight into the agency's decision-making process, revealing explicit racial prejudice.</p>
            <p class="text-xs">Please note that some of the language presented here may reflect outdated and potentially offensive terminology.</p>
            <div class="h-[65vh] text-left text-sm">
                {#if activeAreaData}
                    <h3 class="text-lg font-bold" style="color: {activeGrade}">Area Description - {activeAreaData.label}</h3>
                    <!-- <p>Estimated Annual Family Income: {activeAreaData.estimated_annual_family_income.replace(/,000/g, "k").replace(/- /g, "- $").replace(/\s/g, "")}</p> -->
                    <p><span class="font-bold mt-2">Infiltration of:</span> {activeAreaData.infiltration_of}</p>
                    <p><span class="font-bold mt-2">Foreign-Born Residents:</span> {activeAreaData.foreign_born_nationality}, {activeAreaData.foreign_born_percent.trim() || "0%"}</p>
                    <p><span class="font-bold mt-2">Black Residents Noted:</span> {activeAreaData.negro_yes_or_no}, {activeAreaData.negro_percent.replace("-", "") || "0%"}</p>
                    <p><span class="font-bold mt-2">Occupation:</span> {activeAreaData.occupation_or_type}</p>
                    {#if quote}
                        <p class="font-bold text-center mt-2">Selected Clarifying Remarks:</p>
                        <p>&quot;{quote}&quot;</p>
                    {/if}
                    <p class="font-bold text-center mt-2">Favorable Influences:</p>
                    <p>&quot;{activeAreaData.favorable_influences.trim() || "0%"}&quot;</p>
                    <p class="font-bold text-center mt-2">Detrimental Influences:</p>
                    <p>&quot;{activeAreaData.detrimental_influences.trim() || "0%"}&quot;</p>
                {:else}
                    <h3 class="text-lg font-bold">Area Description</h3>
                    <p>Hover over each HOLC-designated area to read what was written in its report.</p>
                {/if}
            </div>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:left-[30vw] xl:left-[35vw]" data-step="3">
            <p>Below is a map of Oakland showing Black homeownership rates. Specifically: &quot;Percent of households living in owner-occupied housing. A housing unit is owner occupied if the owner or co-owner lives in the unit, even if it is mortgaged or not fully paid for.&quot;</p>
            <ImageEnlarger src="images/homeownership-map.png" alt="homeownership map" caption={'Screenshot sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">American Community Survey (2023).'} captionShow={false} />
            <ImageEnlarger src="images/homeownership-scale.png" alt="homeownership scale" caption={'Screenshots sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">American Community Survey (2023).</span>'} />
            <p>Notice how the darker areas of the map above tend to overlap with the green (A) and blue (B) parts of the HOLC map? It's not an exact match, because there are other factors that impact homeownership. It's still significant correlation, though.</p>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:right-[30vw] xl:right-[35vw]" data-step="4">
            <p>Now, let's look at social vulnerability, which &quot;measures the likelihood a community faces negative consequences from environmental, economic or health-related disasters, with a higher score reflecting greater risk.&quot;</p>
            <ImageEnlarger src="images/socialvulnerability-map.png" alt="social vulnerability map" caption={'Screenshot sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">Center for Disease Control (2022).'} captionShow={false} />
            <ImageEnlarger src="images/socialvulnerability-scale.png" alt="social vulnerability scale" caption={'Screenshots sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">Center for Disease Control (2022).</span>'} />
            <p>This time, the yellow (C) and red (D) areas overlap with the darker areas.</p>
            <p>Again, it might not just be redlining that caused this disparity. It may even be a different cause. However, we at least know that redlining and social vulnerability are linked, and by combining history knowledge with contemporary data, we're able to identify areas most in need of support.</p>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-left lg:left-[30vw] xl:left-[35vw]" data-step="5">
            <p>Here's another visualization of social vulnerability.</p>
            <ImageEnlarger src="images/ncrc-socialvulnerability.jpg" alt="social vulnerability map" caption={"National Community Reinvestment Coalition (2020).</span>"} />
            <p>And one showing life expectancy in relation to redlined areas.</p>
            <ImageEnlarger src="images/ncrc-lifeexpectancy.jpg" alt="life expectancy map" caption={"National Community Reinvestment Coalition (2020).</span>"} />
            <p>Both charts show strong correlations with the HOLC map.</p>
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:right-[30vw] xl:right-[35vw]" data-step="6">
            <p>Studies have also proven strong links between redlining and public health issues. Nardone et al. found that, &quot;Historically redlined census tracts have significantly higher rates of emergency department visits due to asthma.&quot;</p>
            <ImageEnlarger src="images/asthma-heatmap.jpg" alt="asthma heatmap" caption={'<span class="font-bold">Associations between historical residential redlining and current age-adjusted rates of emergency department visits due to asthma across eight cities in California: an ecological study.</span> Nardone, Anthony et al. The Lancet Planetary Health, Volume 4, Issue 1, e24 - e31. (2020).'} captionShow={false} />
            <ImageEnlarger src="images/asthma-boxplot.jpg" alt="asthma boxplot" caption={'<span class="font-bold">Associations between historical residential redlining and current age-adjusted rates of emergency department visits due to asthma across eight cities in California: an ecological study.</span> Nardone, Anthony et al. The Lancet Planetary Health, Volume 4, Issue 1, e24 - e31. (2020).'} />
        </div>
        <div class="step bg-[#ddd] dark:bg-[#444] relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:left-[30vw] xl:left-[35vw]" data-step="7">
            <p>How do we know the relationship between redlining and asthma is not purely coincidental? We can't say anything for sure, especially just with the data presented here. But here's another visualization that can give us a clue. &quot;This metric measures how many days nearby sensors recorded air quality levels over 50, the threshold at which sensitive groups can begin to be affected by pollutants. Air quality index (AQI) is determined by concentration of PM2.5 air particles.&quot;</p>
            <ImageEnlarger src="images/airquality-map.png" alt="air quality map" caption={'Screenshot sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">Environmental Protection Agency (2024), PurpleAir (2024).'} captionShow={false} />
            <ImageEnlarger src="images/airquality-scale.png" alt="air quality scale" caption={'Screenshots sourced from: <span class="font-bold">Black Wealth Data Center.</span> Data sourced from: <span class="font-bold">Environmental Protection Agency (2024), PurpleAir (2024).</span>'} />
            <p>There is significant overlap between areas with higher rates of emergency department visits due to asthma and areas with lower air quality. We know that poor air quality causes asthma, so this makes sense. But what's causing the bad air?</p>
            <p>Look more closely at West Oakland. That's where you can find the Industrial and Commercial areas, as marked on the HOLC map. And because industrial activities release a lot of pollutants, the air quality tends to be poor.</p>
            <p>Notice how all the areas around the Industrial and Commercial areas are marked as yellow (C) and red (D)? Banks would refuse or be reluctant to give out loans to these areas' residents, making it harder for them to move out and into cleaner areas.</p>
        </div>
        <div class="relative z-1 w-96 p-4 flex flex-col gap-4 text-justify lg:left-[30vw] xl:left-[35vw] h-screen" data-step="8"></div>
    </article>

    <figure class="sticky top-0 h-screen w-full overflow-hidden z-0 m-0">
        <div id="map" class="relative w-full h-full">
            <svg id="overlay" class="absolute top-0 left-0 z-[1000]"></svg>
        </div>
    </figure>
</section>

<div id="tooltip" class="absolute bg-white text-black p-1 text-sm hidden"></div>

<style>
    #scrolly-two > * {
        -webkit-box-flex: 1;
        -ms-flex: 1;
        flex: 1;
        grid-column: 1;
        grid-row: 1;
    }

    .step {
        margin: 0 auto 2rem auto;
    }

    .step:last-child {
        margin-bottom: 0;
    }
</style>
