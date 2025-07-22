<script>
    import { onMount, getContext } from "svelte";
    import { fade } from "svelte/transition";
    const scrollama = getContext("scrollama");
    const d3 = getContext("d3");
    import Card from "./Card.svelte";

    let pageIndex = 0;

    onMount(() => {
        const main = d3.select("main");
        const scrolly = main.select("#scrolly-one");
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

            const figureHeight = window.innerHeight / 1.25;
            const figureMarginTop = (window.innerHeight - figureHeight) / 2;

            figure.style("height", figureHeight + "px").style("top", figureMarginTop + "px");

            // 3. tell scrollama to update new element dimensions
            scroller.resize();
        }

        // scrollama event handlers
        function handleStepEnter(response) {
            // response = { element, direction, index }

            // update graphic based on step
            // figure.select("p").text(response.index + 1);
            pageIndex = response.index;
        }

        function init() {
            // 1. force a resize on load to ensure proper dimensions are sent to scrollama
            handleResize();

            // 2. setup the scroller passing options
            // 		this will also initialize trigger observations
            // 3. bind scrollama event handlers (this can be chained like below)
            scroller
                .setup({
                    step: "#scrolly-one article .step",
                    offset: 0.33,
                    debug: false,
                })
                .onStepEnter(handleStepEnter);
        }

        // kick things off
        init();

        return () => {
            scroller.destroy();
        };
    });
</script>

<Card text="Growing up in the San Francisco Bay Area, I learned a lot about racial inequity at school, especially in history class. But the history we learned felt distant, in terms of both time and space." span="" suffix="" />

<section id="scrolly-one" class="my-8 relative grid">
    <article class="w-full">
        <div class="step relative flex flex-col justify-center text-left z-1 w-xs text-xl left-[34vw]" data-step="1">
            <p>We learned about the Civil War, which took place 165 years ago and was fought between the North and the South.</p>
        </div>
        <div class="step relative flex flex-col justify-center text-left z-1 w-xs text-xl right-[34vw]" data-step="2">
            <p>There was the Great Migration a century ago, during which a large number of Black people moved from the South to the North and Midwest.</p>
        </div>
        <div class="step relative flex flex-col justify-center text-left z-1 w-xs text-xl left-[34vw]" data-step="3">
            <p>We also know about Jim Crow laws, which constituted systemic racial discrimination.</p>
        </div>
        <div class="step relative flex flex-col justify-center text-left z-1 w-xs text-xl right-[34vw]" data-step="4">
            <p>There is also a lot of material about legislation and historical figures in the South, especially relating to race.</p>
        </div>
        <div class="step relative flex flex-col justify-center text-left z-1 w-xs text-xl left-[36vw]" data-step="5">
            <p>This is by no means a complete timeline, and it's important history. But there's also local history that we should know.</p>
            <p class="text-sm">Textbook: TEKS United States History Since 1877 (2016). <span class="text-xs">Not the textbook used in my history class.</span></p>
            <p class="text-sm">Photo: Oakland Public Library.</p>
        </div>
    </article>

    <figure class="flex justify-center">
        {#key pageIndex}
            <img src={`images/history-${pageIndex + 1}.jpeg` || "/images/history-1.jpeg"} alt="textbook page" transition:fade={{ duration: 1000 }} class="absolute max-w-full max-h-full object-contain" />
        {/key}
    </figure>
</section>

<style>
    #scrolly-one > * {
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
    }

    .step:last-child {
        margin-bottom: 0;
    }

    .step p {
        padding: 1rem;
    }
</style>
