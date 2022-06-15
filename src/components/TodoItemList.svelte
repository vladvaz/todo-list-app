<script lang="ts">
    import type { TodoType } from "src/types/todo.type";
    import { onDestroy, onMount } from "svelte";
    import { flip } from "svelte/animate";
    import { fly } from "svelte/transition";
    import TodoItem from "./TodoItem.svelte";

    export let list: Array<TodoType> = [];

    let startItem: number;
    let targetItem: number;

    function handleStartDrag(event: CustomEvent<{ id: number }>) {
        startItem = event.detail.id;
    }

    function handleDrop(event: CustomEvent<{ id: number }>) {
        targetItem = event.detail.id;
        swap(startItem, targetItem);
    }

    function swap(startItem: number, targetItem: number) {
        const newTracklist = list;

        if (startItem < targetItem) {
            newTracklist.splice(targetItem + 1, 0, newTracklist[startItem]);
            newTracklist.splice(startItem, 1);
        } else {
            newTracklist.splice(targetItem, 0, newTracklist[startItem]);
            newTracklist.splice(startItem + 1, 1);
        }
        list = newTracklist;
    }

    let hasDelay = true;
    let timeoutRef: NodeJS.Timeout;
    onMount(() => {
        timeoutRef = setTimeout(() => {
            hasDelay = false;
        }, 755);
    });

    onDestroy(() => clearTimeout(timeoutRef));
</script>

{#if list.length}
    <ul in:fly={{ x: -400, duration: 400, delay: 210 }}>
        {#each list as todo, index (todo.id)}
            <li
                animate:flip={{ duration: 300 }}
                in:fly={{
                    y: 200,
                    duration: 200,
                    delay: hasDelay ? (index * 750) / (list.length - 1) : 0,
                }}
                out:fly={{ x: 400, duration: 400 }}
            >
                <TodoItem
                    {todo}
                    {index}
                    on:dragStart={handleStartDrag}
                    on:drop={handleDrop}
                    on:remove
                    on:update
                />
            </li>
        {/each}
    </ul>
{:else}
    <h1
        in:fly={{ x: -400, duration: 400, delay: 410 }}
        out:fly={{ y: 200, duration: 200 }}
    >
        No todos for now!
    </h1>
{/if}

<style>
    ul {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-start;
        list-style: none;
        margin: 0;
        padding: 0;
        width: 100%;
        padding-bottom: 100px;
    }

    li {
        width: inherit;
    }

    h1 {
        margin-top: 256px;
        color: #363636;
        margin-bottom: 16px;
        background-color: #dfdfdf;
        border-radius: 12px;
        padding: 8px 16px;
    }
</style>
