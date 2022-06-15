<script lang="ts">
    import { formatDistanceToNow, differenceInDays } from "date-fns";
    export let date: Date;

    function formatColor(date: Date): string {
        const days = differenceInDays(date, new Date());

        if (days > 3) {
            return "green-badge";
        } else if (days >= 1 && days <= 3) {
            return "orange-badge";
        } else {
            return "red-badge";
        }
    }
</script>

{#if date}
    <span class="date small {formatColor(date)}"
        >{formatDistanceToNow(date, { addSuffix: true })}</span
    >
{/if}

<style>
    .date {
        display: inline-block;
        padding: 4px 8px;
        color: var(--text-color);
        font-weight: 600;
        border-radius: 12px;
    }

    .small {
        font-size: smaller;
    }
</style>
