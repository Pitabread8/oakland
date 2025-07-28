<script>
    import { onMount, getContext } from "svelte";
    const AOS = getContext("aos");
    import "aos/dist/aos.css";
    onMount(() => {
        AOS.init();
    });

    let selected = "individual";

    function toggle() {
        selected = selected === "individual" ? "local" : "individual";
        setTimeout(() => {
            AOS.refreshHard();
        }, 50);
    }
</script>

<div class="w-screen h-screen flex flex-col items-center justify-center p-4 gap-3">
    <h2 data-aos="fade-up" data-aos-duration="800" class="text-4xl font-extrabold text-red-600">What next?</h2>
    <p data-aos="fade-up" data-aos-duration="800" class="mb-3">Fixing racial inequity can seem like a daunting task. But we can tackle it by starting small.</p>
    <button data-aos="fade-up" data-aos-duration="800" class="relative flex w-64 rounded-full border-2 border-red-400 cursor-pointer p-1 bg-white overflow-hidden mt-4 mb-2" on:click={toggle}>
        <div class="absolute inset-y-1 left-1 w-1/2 bg-red-600 rounded-full transition-transform duration-300 ease-in-out" class:translate-x-[calc(100%-0.5rem)]={selected === "local"}></div>
        <div class={`relative z-10 flex-1 text-center py-2 rounded-full transition-colors duration-300 ${selected === "individual" ? "text-white" : "text-[#242424]"}`}>Individual</div>
        <div class={`relative z-10 flex-1 text-center py-2 rounded-full transition-colors duration-300 ${selected === "local" ? "text-white" : "text-[#242424]"}`}>Government</div>
    </button>
    {#if selected === "individual"}
        <div data-aos="fade-in" data-aos-offset="100" data-aos-duration="800" data-aos-easing="ease-in-out" class="w-2/3 p-6 rounded-lg border-red-600 border-4 border-double text-justify">
            <p>Start by educating yourself. What discriminatory policies historically existed in your area? Look into data and trends about demographics today, collected by organizations like <a href="https://blackwealthdata.org/" target="_blank" class="underline text-red-400 underline-offset-4">Black Wealth Data Center</a>. What multitude of factors created the disparities you see around you? This could include redlining, as well as communities replaced by newer development, freeways, and transit lines (like Whiskey Gulch and 7th Street). Help build awareness of these issues in your community. Then, look towards helping marginalized groups build wealth. Support Black businesses! Finally, reach out to your local government officials and ask them to take large-scale actions.</p>
        </div>
    {:else}
        <div data-aos="fade-in" data-aos-offset="100" data-aos-duration="800" data-aos-easing="ease-in-out" class="w-2/3 p-6 rounded-lg border-red-600 border-4 border-double text-justify">
            <p>Local governments can play a large role in counteracting the long-term consequences of redlining by targeting housing and businesses, which are the best ways to generate household wealth. For housing, one tool is zoning reform: cities can revise outdated zoning codes that restrict multi-family housing and ADUs, making it easier to build diverse and affordable housing options in historically excluded neighborhoods. Municipalities can also create community land trusts, which help keep housing permanently affordable and give residents shared ownership. For small businesses, cities can offer grants or low-interest lo ans. Incubator programs are also a good option.</p>
        </div>
    {/if}
    <p data-aos="fade-up" class="mt-6 text-lg">With specific, targeted, and data-informed actions, we can rectify centuries of discrimination and marginalization and build a better future.</p>
</div>
