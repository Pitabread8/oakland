<script>
    import { setContext } from "svelte";
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import Introduction from "./lib/components/Introduction.svelte";
    import Card from "./lib/components/Card.svelte";

    setContext("scrollama", scrollama);
    setContext("d3", d3);
</script>

<main>
    <!-- title component -->
    <h1>Oakland</h1>
    <Introduction />
    <Card text="But discriminatory policies existed all over the country, not least right here at home. Though they no longer exist on paper, they continue to impact access to wealth for families and communities. One practice is particularly visible in the Bay Area: " span="redlining" suffix="." />
</main>
