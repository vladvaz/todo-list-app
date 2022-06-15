<script lang="ts">
    import type { TodoType } from "../types/todo.type";
    import { createEventDispatcher } from "svelte";
    import DateBadge from "./DateBadge.svelte";

    export let todo: TodoType;
    export let index: number;

    let editMode = false;
    let editModeValue = "";
    let isHovering = false;

    const dispatch = createEventDispatcher<{
        dragStart: { id: number };
        drop: { id: number };
        remove: { id: number };
        update: { todo: TodoType };
    }>();

    function dragStart(event: DragEvent, target: number) {
        dispatch("dragStart", {
            id: target,
        });
    }
    function drop(event: DragEvent, target: number) {
        isHovering = false;
        dispatch("drop", {
            id: target,
        });
    }

    function onRemove() {
        dispatch("remove", { id: todo.id });
    }

    function onUpdate() {
        dispatch("update", { todo: todo });
    }

    function onEdit() {
        editMode = true;
        editModeValue = todo.description;
    }

    function onCancelEdit() {
        editMode = false;
        editModeValue = "";
    }

    function onCheckEdit() {
        todo.isDone = !todo.isDone;
        onUpdate();
    }

    function onSaveEdit() {
        editMode = false;
        todo.description = editModeValue;
        editModeValue = "";
        onUpdate();
    }
</script>

<div
    class="todo-item"
    class:is-hovered-by-item={isHovering}
    draggable="true"
    on:dragstart={(event) => dragStart(event, index)}
    on:drop|preventDefault={(event) => drop(event, index)}
    on:dragover={(event) => event.preventDefault()}
    on:dragenter={() => (isHovering = true)}
    on:dragleave={() => (isHovering = false)}
>
    {#if editMode}
        <input type="text" bind:value={editModeValue} />
        <div class="btn-group">
            <button type="button" class="btn-tertiary" on:click={onSaveEdit}>
                Save <span class="visually-hidden">{todo.description}</span>
            </button>
            <button type="button" class="btn-secondary" on:click={onCancelEdit}>
                Cancel <span class="visually-hidden">{todo.description}</span>
            </button>
        </div>
    {:else}
        <div class="todo-data">
            <div class="todo-item-cb">
                <input
                    type="checkbox"
                    name="checkbox"
                    id="todo-{todo.id}"
                    checked={todo.isDone}
                    on:click={onCheckEdit}
                />

                <label for="todo-{todo.id}" class="todo-label">
                    {todo.description}
                </label>
            </div>

            <DateBadge date={todo.dueDate} />
        </div>
        <div class="btn-group">
            <button type="button" class="btn-secondary" on:click={onEdit}>
                Edit <span class="visually-hidden">{todo.description}</span>
            </button>
            <button type="button" class="btn-tertiary" on:click={onRemove}>
                Delete <span class="visually-hidden">{todo.description}</span>
            </button>
        </div>
    {/if}
</div>

<style>
    .todo-data {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        width: 100%;
    }
    .todo-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
        margin-bottom: 1rem;
        padding: 8px;
        border-radius: 12px;
        background-color: rgb(241, 241, 229);
        gap: 32px;
        font-weight: 600;
    }

    .todo-item:hover {
        outline: 1px solid var(--tertiary-color);
        outline-offset: -1px;
    }

    .todo-item-cb > input[type="checkbox"] {
        cursor: pointer;
        border: 1px solid grey;
        height: 20px;
        width: 20px;
        min-height: 20px;
        min-width: 20px;
    }

    .todo-item > input[type="text"] {
        width: 100%;
        height: 22px;
        margin-right: 12px;
        border-radius: 6px;
        border: 1px solid grey;
        font-size: medium;
        color: grey;
    }

    .todo-item-cb {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .todo-item-cb > label {
        padding-left: 12px;
        padding-right: 12px;
        font-size: medium;
        text-align: center;
    }

    .btn-group {
        display: flex;
        gap: 4px;
    }

    .todo-item.is-hovered-by-item {
        background-color: var(--tertiary-color);
        outline: 2px dashed rgb(243, 243, 243);
        outline-offset: -2px;
        color: rgb(243, 243, 243);
    }

    .todo-item.is-hovered-by-item * {
        pointer-events: none;
    }
</style>
