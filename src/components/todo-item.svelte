<script lang="ts">
  import { createEventDispatcher, tick } from 'svelte'
  import type { Todo, UpdatedTodo } from '@/types/todo.type'
  import { selectOnFocus } from '@/actions/select-onfocus'

  // props
  export let todo: Todo

  // local
  const dispatch = createEventDispatcher()
  let isEditing = false
  let isEditBtnPressed = false
  let task = todo.task
  let taskEl: HTMLElement

  // listeners on dispatched events get a `detail` prop on `event`. for
  // clarity, i added an object on `detail` with the name of the data. see
  // code in `todo-list.svelte`.
  function updateTodo(updatedTodo: UpdatedTodo) {
    todo = { ...todo, ...updatedTodo }
    dispatch('update', { todo })
  }

  function onCancel() {
    isEditing = false
    task = todo.task
  }

  function onSave() {
    updateTodo({ task })
    isEditing = false
  }

  function onRemove() {
    dispatch('remove', { todo })
  }

  // after setting `isEditing`, svelte needs to re-render the dom, and mount
  // the taskEl node. so, we wait a tick before accessing `taskEl`.
  //
  // a better way to focus the input task field is:
  // 1. remove: `let taskEl`, `await tick()` and `taskEl.focus()`
  // 1. remove `async` from `onEdit`
  // 1. create a new action:
  //      const focusOnInit = (node) =>
  //        typeof node?.focus === 'function' && node.focus()
  // 1. use action;
  //      <input use:focusOnInit ...etc />
  async function onEdit() {
    isEditing = true
    isEditBtnPressed = true
    await tick()
    taskEl.focus()
  }

  function onToggle() {
    updateTodo({ completed: !todo.completed })
  }

  // action
  function focusEditButton(node: HTMLButtonElement): void {
    if (typeof node?.focus !== 'function') {
      throw new Error('Use action for HTMLButtonElement only')
    }
    isEditBtnPressed && node.focus()
  }
</script>

<!--
  the keydown is scoped to the form. it only works when cursor is in field.
-->
<li>
  {#if isEditing}
    <form
      on:submit|preventDefault={onSave}
      on:keydown={(e) => e.key === 'Escape' && onCancel()}
    >
      <label>
        New task name for '{todo.task}'
        <input
          type="text"
          autocomplete="off"
          placeholder="Add a task (i.e. Buy Groceries)"
          bind:value={task}
          bind:this={taskEl}
          use:selectOnFocus
        />
      </label>
      <button type="button" on:click={onCancel}>
        Cancel <span class="sr-only">renmaing {todo.task}</span>
      </button>
      <button type="submit" disabled={!task}>
        Save <span class="sr-only">new name for {todo.task}</span>
      </button>
    </form>
  {:else}
    <label>
      <input type="checkbox" checked={todo.completed} on:click={onToggle} />
      {todo.task}
    </label>
    <button type="button" on:click={onEdit} use:focusEditButton>
      Edit <span class="sr-only">{todo.task}</span>
    </button>
    <button type="button" on:click={onRemove}>
      Delete <span class="sr-only">{todo.task}</span>
    </button>
  {/if}
</li>
