<script>
    import { onMount, getContext } from "svelte";
    const AOS = getContext("aos");
    import "aos/dist/aos.css";
    onMount(() => {
        AOS.init();
    });

    let { text, span, suffix } = $props();
</script>

<div class="w-screen h-screen flex items-center justify-center">
    <div data-aos="fade-up" data-aos-offset="200" data-aos-duration="800" data-aos-easing="ease-in-out" class="w-1/2 p-6 rounded-lg border-red-600 border-4 border-double">
        <p>{text}<span class="text-red-400">{span}</span>{suffix}</p>
    </div>
</div>
