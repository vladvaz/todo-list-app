<script lang="ts">
    import { createEventDispatcher } from "svelte";

    export let data = { inputValue: "", dateValue: "" };

    const dispatch = createEventDispatcher();

    function submit() {
        dispatch("submit");
    }
</script>

<form class="add-todo-container" on:submit|preventDefault={submit}>
    <input type="text" bind:value={data.inputValue} />
    <input type="datetime-local" bind:value={data.dateValue} />
    <button class="btn-primary" type="submit">Add</button>
</form>

<style>
    .add-todo-container {
        position: fixed;
        left: 0;
        right: 0;
        bottom: 32px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .add-todo-container > input[type="text"] {
        width: 50%;
        max-width: 600px;
        min-height: 24px;
        margin-right: 12px;
        border-radius: 6px;
        border: 1px solid grey;
    }

    .add-todo-container > button {
        margin-left: 12px;
        min-height: 30px;
        min-width: 80px;
    }
</style>
