<script>
    import { getContext } from "svelte";
    const scrollama = getContext("scrollama");
    const d3 = getContext("d3");

    import { onMount, onDestroy } from "svelte";

    const pages = ["https://images.ctfassets.net/hrltx12pl8hq/28ECAQiPJZ78hxatLTa7Ts/2f695d869736ae3b0de3e56ceaca3958/free-nature-images.jpg?fit=fill&w=1200&h=630", "https://plus.unsplash.com/premium_photo-1664474619075-644dd191935f?fm=jpg&q=60&w=3000&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8aW1hZ2V8ZW58MHx8MHx8fDA%3D", "https://images.ctfassets.net/hrltx12pl8hq/28ECAQiPJZ78hxatLTa7Ts/2f695d869736ae3b0de3e56ceaca3958/free-nature-images.jpg?fit=fill&w=1200&h=630", "https://plus.unsplash.com/premium_photo-1664474619075-644dd191935f?fm=jpg&q=60&w=3000&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8aW1hZ2V8ZW58MHx8MHx8fDA%3D"];

    onMount(() => {
        const main = d3.select("main");
        const scrolly = main.select("#scrolly");
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
            console.log(response);
            // response = { element, direction, index }

            // add color to current step only
            step.classed("is-active", function (d, i) {
                return i === response.index;
            });

            // update graphic based on step
            // figure.select("p").text(response.index + 1);
            document.querySelector("img").src = pages[response.index];
        }

        function init() {
            // 1. force a resize on load to ensure proper dimensions are sent to scrollama
            handleResize();

            // 2. setup the scroller passing options
            // 		this will also initialize trigger observations
            // 3. bind scrollama event handlers (this can be chained like below)
            scroller
                .setup({
                    step: "#scrolly article .step",
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

<p>Growing up in the San Francisco Bay Area, I learned a lot about racial inequity at school, especially in history class. But the history we learned felt distant, in terms of both time and space.</p>

<section id="scrolly">
    <article>
        <div class="step" data-step="1">
            <p>STEP 1</p>
        </div>
        <div class="step" data-step="2">
            <p>STEP 2</p>
        </div>
        <div class="step" data-step="3">
            <p>STEP 3</p>
        </div>
        <div class="step" data-step="4">
            <p>STEP 4</p>
        </div>
    </article>

    <figure>
        <img src="https://images.ctfassets.net/hrltx12pl8hq/28ECAQiPJZ78hxatLTa7Ts/2f695d869736ae3b0de3e56ceaca3958/free-nature-images.jpg?fit=fill&w=1200&h=630" alt="textbook page" />
    </figure>
</section>

<style>
    #scrolly {
        position: relative;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        background-color: #f3f3f3;
        padding: 1rem;
    }

    #scrolly > * {
        -webkit-box-flex: 1;
        -ms-flex: 1;
        flex: 1;
    }

    article {
        position: relative;
        padding: 0 1rem;
        max-width: 20rem;
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

    img {
        height: 500px;
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
