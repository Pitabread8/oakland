<script>
    let { text, span, suffix } = $props();
</script>

<div class="w-screen h-screen flex items-center justify-center">
    <div class="w-1/2 p-6 rounded-lg border-red-600 border-4 border-double">
        <p>{text}<span class="text-red-400">{span}</span>{suffix}</p>
    </div>
</div>
