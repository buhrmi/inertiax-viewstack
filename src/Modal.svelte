<script module>
  /**
   * Svelte action that turns any link into a trigger for the modal stack.
   *
   * Attach it to a link (or a wrapper containing links):
   *
   *   <a href="/holders" use:modal>Holders</a>
   */
  export function modal(node) {
    node.addEventListener('click', onClickLink)
    return {
      destroy() {
        node.removeEventListener('click', onClickLink)
      }
    }
  }

  function onClickLink(e) {
    e.preventDefault()
    const href = e.target.closest('a')?.getAttribute('href')
    if (!href) return
    const state = history.state || {}
    state["modal"] = [{ src: href }]
    history.pushState(state, undefined)
    window.dispatchEvent(new PopStateEvent("popstate", { state }))
  }
</script>

<script>
  import ViewStack from "./ViewStack.svelte"
  import { fade } from "svelte/transition"

  let stack = $state([])

  function close() {
    history.go(-stack.length)
  }
</script>

{#if stack.length}
  <div class="blur" transition:fade={{ duration: 200 }} onclick={close}></div>
{/if}

<div class="wrapper">
  <div class="stack" class:open={stack.length > 0} scroll-region>
    <ViewStack id="modal" bind:stack />
  </div>
</div>

<style>
  .wrapper {
    pointer-events: none;
    display: grid;
    place-items: center;
    position: fixed;
    inset: 0;
  }

  .stack {
    visibility: hidden;
    pointer-events: none;
    position: fixed;
    width: 100%;

    bottom: 0;
    height: 100dvh;
    transform: translateX(100%);
    transition: transform 400ms cubic-bezier(0.215, 0.61, 0.355, 1),
      visibility 400ms;

    background-color: var(--ivs-surface, #0d0d13);
    border: 1px solid var(--ivs-border-color, var(--color-border, #d5d5dd22));
    overflow: hidden;
    display: grid;
  }

  .stack.open {
    visibility: visible;
    pointer-events: all;
    transform: translateX(0);
  }

  @media (min-width: 640px) {
    .blur {
      backdrop-filter: blur(12px);
      position: fixed;
      inset: 0;
    }
    .stack {
      width: 400px;
      bottom: auto;
      height: 640px;
      max-height: 90vh;
      transform: scale(0.7);
      opacity: 0;
      border-radius: 1rem;
      transition: transform 300ms cubic-bezier(0.215, 0.61, 0.355, 1),
        opacity 300ms cubic-bezier(0.215, 0.61, 0.355, 1), visibility 300ms;
    }
    .stack.open {
      transform: scale(1);
      opacity: 1;
    }
  }
</style>
