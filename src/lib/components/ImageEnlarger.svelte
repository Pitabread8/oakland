<script>
    export let src = "";
    export let alt = "";
    export let caption = "";
    export let captionShow = true;

    let modalShow = false;

    $: {
        if (modalShow) {
            document.body.classList.add("no-scroll");
        } else {
            document.body.classList.remove("no-scroll");
        }
    }
</script>

<button on:click={() => (modalShow = true)}>
    <img {src} {alt} />
    {#if captionShow}
        <p class="text-xs mt-4 text-justify">{@html caption}</p>
    {/if}
</button>

{#if modalShow}
    <div class="fixed inset-0 z-[9999] flex items-center justify-center bg-black/75 p-4 w-screen h-screen">
        <div class="relative max-w-5xl max-h-full w-full">
            <button class="absolute top-2 right-2 text-[#dcd3ca] text-3xl font-bold hover:text-[#3a5e7d]" on:click={() => (modalShow = false)} aria-label="close">X</button>
            <img {src} {alt} class="w-full h-auto max-h-[80vh] object-contain mx-auto rounded shadow-lg" />
            <p class="text-center mt-4 text-sm">{@html caption}</p>
        </div>
    </div>
{/if}
