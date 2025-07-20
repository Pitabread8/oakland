<!-- bg image -->

<script>
    let { title, subtitle } = $props();
</script>

<div class="w-screen h-screen flex items-center justify-center border-box">
    <div class="content border-red-600 border-8 bg-red-900 p-12 text-shadow-xs text-shadow-red-400">
        <h1 class="text-7xl">{title}</h1>
        <p class="text-xl mt-4">{subtitle}</p>
    </div>
</div>

<style>
    .content {
        border-radius: 2rem;
        box-shadow: 16px 16px oklch(70.4% 0.191 22.216);
    }
</style>
