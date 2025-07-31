<script>
    let { title, subtitle } = $props();
</script>

<div class="w-screen h-screen flex flex-col gap-8 sm:gap-12 items-center justify-center border-box background">
    <div class="content border-red-600 border-8 bg-red-900 p-12 text-shadow-xs text-shadow-red-400">
        <h1 class="text-xl sm:text-3xl md:text-7xl">{title}</h1>
        <p class="text-sm sm:text-lg md:text-xl sm:mt-1 md:mt-4">{subtitle}</p>
    </div>
    <p class="md:fixed md:bottom-4 md:text-black lg:hidden">Please view this on a laptop or bigger screen (&ge;1024px).</p>
</div>

<style>
    @media (min-width: 768px) {
        .background {
            background-image: url("/images/title.jpg");
            background-attachment: fixed;
            background-repeat: no-repeat;
            background-size: cover;
        }
    }
    @media (min-width: 640px) {
        .content {
            box-shadow: 16px 16px oklch(70.4% 0.191 22.216);
        }
    }
    .content {
        border-radius: 2rem;
    }
</style>
