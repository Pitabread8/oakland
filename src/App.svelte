<script>
    import { setContext } from "svelte";
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import AOS from "aos";
    import Hero from "./lib/components/Hero.svelte";
    import Introduction from "./lib/components/Introduction.svelte";
    import Card from "./lib/components/Card.svelte";
    import Map from "./lib/components/Map.svelte";
    import Conclusion from "./lib/components/Conclusion.svelte";

    setContext("scrollama", scrollama);
    setContext("d3", d3);
    setContext("aos", AOS);
</script>

<main>
    <Hero title="Oakland Unscrolled" subtitle="Mapping Racial Inequity" />
    <div class="hidden lg:block">
        <Introduction />
        <Card text="Discriminatory policies existed all over the country, including right here at home. Though they no longer exist on paper, they continue to impact access to wealth for families and communities. One practice is particularly visible in the Bay Area: " span="redlining" suffix="." />
        <Map />
        <Card text="Redlining not only hindered Black people from building household wealth, but also made it difficult for them to leave unsafe areas. And it all happened, and is continuing to happen, " span="next door" suffix=" to me." />
        <Conclusion />
        <footer class="w-screen text-center text-red-300 bottom-0 py-4 text-lg">&copy; {new Date().getFullYear()} Oakland Unscrolled</footer>
    </div>
</main>
