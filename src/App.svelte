<script>
    import { setContext } from "svelte";
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import Hero from "./lib/components/Hero.svelte";
    import Introduction from "./lib/components/Introduction.svelte";
    import Card from "./lib/components/Card.svelte";
    import Map from "./lib/components/Map.svelte";
    import Conclusion from "./lib/components/Conclusion.svelte";

    setContext("scrollama", scrollama);
    setContext("d3", d3);
</script>

<main>
    <Hero title="Oakland Unscrolled" subtitle="Mapping Racial Inequity" />
    <Introduction />
    <Card text="But discriminatory policies existed all over the country, not least right here at home. Though they no longer exist on paper, they continue to impact access to wealth for families and communities. One practice is particularly visible in the Bay Area: " span="redlining" suffix="." />
    <Map />
    <Card text="Redlining not only hindered Black people from building household wealth, but also made it difficult for them to leave unsafe areas. And all of this happened and is continuing to happen " span="next door" suffix=" to me." />
    <Conclusion />
</main>
