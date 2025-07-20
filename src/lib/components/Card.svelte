<script>
    let { text, span, suffix } = $props();
</script>

<div class="w-full flex items-center justify-center my-6">
    <div class="w-1/2 p-6 rounded-lg border-red-600 border-4 border-double">
        <p>{text}<span class="underline underline-offset-4 text-red-400">{span}</span>{suffix}</p>
    </div>
</div>
