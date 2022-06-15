<script lang="ts">
    import type { TodoType } from "../types/todo.type";
    import type { Options } from "../types/options.type";
    import { compareAsc, formatISO9075 } from "date-fns";
    import AddTodo from "./AddTodo.svelte";
    import TodoItemList from "./TodoItemList.svelte";

    let data = { inputValue: "", dateValue: "" };
    let urlBase = "http://localhost:9000/todos";
    let backImageUrl = "background.jpg";
    let todos: Array<TodoType> = [];
    let displayedTodos: Array<TodoType> = [];
    let loadDataReq = loadData();
    let options: Options = { show: "all", sort: "date" };
    $: displayedTodos = handleOptions(todos, options);

    function handleOptions(
        todos: Array<TodoType>,
        options: Options
    ): Array<TodoType> {
        let result: Array<TodoType> = todos;
        switch (options.show) {
            default:
            case "all":
                break;
            case "completed":
                result = todos.filter((t) => t.isDone);
                break;
            case "notCompleted":
                result = todos.filter((t) => !t.isDone);
                break;
        }

        switch (options.sort) {
            case "description":
                result = result.sort((a, b) =>
                    a.description.localeCompare(b.description)
                );
                break;
            case "date":
                result = result.sort((a, b) =>
                    compareAsc(a.dueDate, b.dueDate)
                );
                break;
            default:
                break;
        }
        return result;
    }

    async function loadData() {
        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "text/json");
        const res = await fetch(`${urlBase}`, {
            method: "GET",
            headers: myHeaders,
        });

        if (res.ok) {
            const todosList: Array<TodoType> = await res.json();
            todos = todosList.map((todo) => {
                if (todo.dueDate) {
                    todo.dueDate = new Date(todo.dueDate);
                }
                return todo;
            });
        } else {
            throw new Error(`${res.status} - ${res.statusText}`);
        }
    }

    function handleSubmit() {
        if (data.inputValue) {
            postNewTodo();
        }
        data = { inputValue: "", dateValue: "" };
    }

    function handleDelete(event: CustomEvent<{ id: number }>) {
        deleteTodo(event.detail.id);
    }

    function handleUpdate(event: CustomEvent<{ todo: TodoType }>) {
        updateTodo(event.detail.todo);
    }

    async function postNewTodo() {
        let parseDate = "";

        if (data.dateValue) {
            const d = new Date(data.dateValue);
            parseDate = `${formatISO9075(d, {
                representation: "date",
            })} ${formatISO9075(d, { representation: "time" })}`;
        }

        let todo = {
            description: data.inputValue,
            isDone: false,
            dueDate: parseDate,
        };

        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "text/json");

        const res = await fetch(urlBase, {
            method: "POST",
            headers: myHeaders,
            body: JSON.stringify(todo),
        });

        try {
            if (res.ok) {
                const todoResponse: TodoType = await res.json();
                if (todoResponse.dueDate) {
                    todoResponse.dueDate = new Date(todo.dueDate);
                }
                todos = [...todos, todoResponse];
            }
        } catch (err) {
            console.log(err);
        }
    }

    async function deleteTodo(todoId: number) {
        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "text/json");
        const res = await fetch(`${urlBase}/${todoId}`, {
            method: "DELETE",
            headers: myHeaders,
        });
        try {
            if (res.ok) {
                const _ = await res.json();
                todos = todos.filter((t) => t.id !== todoId);
            }
        } catch (err) {
            console.log(err);
        }
    }

    async function updateTodo(todo: TodoType) {
        let todoToUpdate = {
            description: todo.description,
            isDone: todo.isDone,
            dueDate: todo.dueDate
                ? todo.dueDate.toISOString().split(".")[0].replace("T", " ")
                : "",
            orderId: todo.orderId,
        };

        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "text/json");
        try {
            const res = await fetch(`${urlBase}/${todo.id}`, {
                method: "PUT",
                headers: myHeaders,
                body: JSON.stringify({ ...todoToUpdate }),
            });
            if (res.ok) {
                const json = await res.json();

                todos = [...todos];
            } else {
                console.log("Error: ", res.status, res.statusText);
            }
        } catch (err) {
            console.log(err);
        }
    }

    function sortByDescription() {
        options.sort = "description";
    }

    function sortByDate() {
        options.sort = "date";
    }

    function showAll() {
        options.show = "all";
    }

    function showCompleted() {
        options.show = "completed";
    }

    function showNotCompleted() {
        options.show = "notCompleted";
    }
</script>

{#await loadDataReq}
    <div class="container">
        <h1>Loading...</h1>
    </div>
{:then}
    <div class="full-back" style="background-image: url('{backImageUrl}')">
        <div class="container">
            <div class="title-row">
                <h1>Todo List</h1>
            </div>

            <div class="options-menu">
                <div class="display-options">
                    <span>Show: </span>
                    <button
                        class="btn-primary"
                        class:btn-primary--active={options.show === "all"}
                        on:click={showAll}>All</button
                    >
                    <button
                        class="btn-primary"
                        class:btn-primary--active={options.show === "completed"}
                        on:click={showCompleted}>Completed</button
                    >
                    <button
                        class="btn-primary"
                        class:btn-primary--active={options.show ===
                            "notCompleted"}
                        on:click={showNotCompleted}>Not Completed</button
                    >
                </div>

                <div class="sort-options">
                    <span>Sort by: </span>
                    <button
                        class="btn-primary"
                        class:btn-primary--active={options.sort ===
                            "description"}
                        on:click={sortByDescription}>description</button
                    >
                    <button
                        class="btn-primary"
                        class:btn-primary--active={options.sort === "date"}
                        on:click={sortByDate}>date</button
                    >
                </div>
            </div>
            <TodoItemList
                list={displayedTodos}
                on:remove={handleDelete}
                on:update={handleUpdate}
            />
            <AddTodo bind:data on:submit={handleSubmit} />
        </div>
    </div>
{:catch error}
    <div class="container">
        <h1>{error.message}</h1>
    </div>
{/await}

<style>
    h1 {
        color: #363636;
        margin: 0;
        margin-bottom: 16px;
        background-color: #dfdfdf;
        border-radius: 12px;
        padding: 8px 16px;
    }

    .title-row {
        position: relative;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-around;
    }

    .full-back {
        background-size: cover;
        position: absolute;
        left: 0;
        right: 0;
        overflow-x: hidden;
        top: 0;
        bottom: 0;
        background-position: center;
    }

    .container {
        margin-right: auto;
        margin-left: auto;
        margin-top: 20px;
        max-width: 800px;
        text-align: center;
        padding-left: 10px;
        padding-right: 10px;
    }

    .options-menu {
        display: flex;
        flex-direction: row;
        justify-content: space-around;
        align-items: center;
        gap: 12px;
        background-color: #dfdfdf;
        border-radius: 12px;
        padding: 8px 16px;
        margin-bottom: 16px;
    }

    .display-options {
        display: flex;
        justify-content: space-around;
        gap: 12px;
    }

    .sort-options {
        display: flex;
        justify-content: space-around;
        gap: 12px;
    }

    @media (min-width: 820px) {
        .container {
            padding-left: 0;
            padding-right: 0;
        }
    }
</style>
