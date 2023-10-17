<script lang="ts">
    export let current_pace_locked_to_average_pace: boolean;
    export let showLabel = true;
</script>

<button
    class="btn variant-filled"
    class:variant-filled={current_pace_locked_to_average_pace}
    class:variant-ghost={!current_pace_locked_to_average_pace}
    on:click={() => (current_pace_locked_to_average_pace = !current_pace_locked_to_average_pace)}
>
    <span>
        {#if current_pace_locked_to_average_pace}
            <!-- lock -->
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 16 16"
                ><path
                    fill="currentColor"
                    d="M4 4a4 4 0 0 1 8 0v2h.25c.966 0 1.75.784 1.75 1.75v5.5A1.75 1.75 0 0 1 12.25 15h-8.5A1.75 1.75 0 0 1 2 13.25v-5.5C2 6.784 2.784 6 3.75 6H4Zm8.25 3.5h-8.5a.25.25 0 0 0-.25.25v5.5c0 .138.112.25.25.25h8.5a.25.25 0 0 0 .25-.25v-5.5a.25.25 0 0 0-.25-.25ZM10.5 6V4a2.5 2.5 0 1 0-5 0v2Z"
                /></svg
            >
        {:else}
            <!-- unlock -->
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 16 16"
                ><path
                    fill="currentColor"
                    d="M5.5 4v2h7A1.5 1.5 0 0 1 14 7.5v6a1.5 1.5 0 0 1-1.5 1.5h-9A1.5 1.5 0 0 1 2 13.5v-6A1.5 1.5 0 0 1 3.499 6H4V4a4 4 0 0 1 7.371-2.154a.75.75 0 0 1-1.264.808A2.5 2.5 0 0 0 5.5 4Zm-2 3.5v6h9v-6h-9Z"
                /></svg
            >
        {/if}
    </span>
    {#if showLabel}
    <span> Curr = Avg </span>
    {/if}
</button>