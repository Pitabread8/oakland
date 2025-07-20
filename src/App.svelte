<script>
    import { setContext } from "svelte";
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import Introduction from "./lib/components/Introduction.svelte";

    setContext("scrollama", scrollama);
    setContext("d3", d3);
</script>

<main>
    <h1>Oakland</h1>

    <Introduction />
</main>
