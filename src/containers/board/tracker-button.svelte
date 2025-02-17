<script>
  // TODO: Move this to components/tracker-button

  // svelte
  import { createEventDispatcher, onMount } from "svelte";

  // modules
  import Tracker from "../../modules/tracker/tracker";

  // Utils

  // Components
  import Counter from "../../components/counter/counter.svelte";

  // Props
  export let tracker = new Tracker();
  export let value = null;
  export let refreshing = false;
  export let id = undefined;
  export let className = undefined;

  // Define Dispatch
  const dispatch = createEventDispatcher();

  let data = {
    pressing: false
  };

  let timeout = null;
  // Functions
  const methods = {
    // Clicked
    clicked() {
      dispatch("click", {});
    },

    // On Mouse Release / Touch Stop
    mouseup() {
      data.pressing = false;
      clearInterval(timeout);
    },

    // On Mouse / Touch
    mousedown(event) {
      if (!window.scrolling) {
        // Figure long press
        data.pressing = true;
        // Clear by default
        clearInterval(timeout);
        // Timeout after 900ms
        timeout = setInterval(() => {
          // kill any propagation
          event.stopPropagation();
          event.preventDefault();
          // clear interval
          clearInterval(timeout);
          // fire long press - if we're not scrolling
          // the user will tend to press and hold a tracker when scrolling.
          // this should limit false fires
          if (!document.body.classList.contains("scrolling")) {
            dispatch("longpress");
          }
        }, 900);
      }
    }
  };
</script>

<style lang="scss" type="text/scss">
  @import "../../scss/vendor/bootstrap/base";
  :global(body.scrolling .n-tracker-button) {
    pointer-events: none !important;
  }
  .n-tracker-button {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border: solid 1px rgba(0, 0, 0, 0.1);
    background-color: var(--color-solid);
    color: var(--color-inverse);
    box-shadow: 0px 6px 10px -5px rgba(0, 0, 0, 0.1);
    border-radius: 1rem;
    width: 5.9rem;
    height: 5.9rem;
    outline: none;
    transition: all 0.2s ease-in-out;

    @include media-breakpoint-up(md) {
      width: 7.8rem;
      height: 7.8rem;
      .emoji {
        font-size: 3.5rem;
        font-style: normal;
      }
    }

    @include media-breakpoint-down(md) {
      width: 6.8rem;
      height: 6.8rem;
      .emoji {
        font-size: 2.8rem;
        font-style: normal;
      }
    }

    @include media-breakpoint-down(sm) {
      width: 5.7rem;
      height: 5.7rem;
      margin: 6px 3px !important;

      .emoji {
        font-size: 2.2rem;
        font-style: normal;
      }
    }

    &.pressing {
      box-shadow: 0px 4px 6px -3px rgba(0, 0, 0, 0.1);
      transform: scale(0.98);
      opacity: 0.49;
    }

    .one-tap {
      position: absolute;
      height: 4px;
      left: 20px;
      right: 20px;
      bottom: 2px;
      border-radius: 2px;
      background-color: rgba(0, 0, 0, 0.1);
    }
    header {
      display: flex;
      justify-content: space-between;
      position: absolute;
      top: 0.4rem;
      left: 6px;
      right: 6px;
      font-size: 11px;
      line-height: 11px;

      .value.left {
        height: 20px;
        border-radius: 8px;
        font-size: 14px;
        line-height: 14px;
        min-width: 16px;
        position: absolute;
        display: flex;
        align-items: center;
        font-weight: bold;
        top: 0px;
        left: 6px;
        background: transparent;
      }
    }
    footer {
      position: absolute;
      bottom: 0.5rem;
      left: 4px;
      right: 4px;
      text-align: center;
      line-height: 100%;
      font-size: 0.8rem;
    }
    .countdown {
      position: absolute;
      top: 0;
      left: 10%;
      right: 10%;
      bottom: 0;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
</style>

<button
  {id}
  class="n-tracker-button {data.pressing ? 'pressing' : ''}
  {className}"
  on:click={methods.clicked}
  on:touchstart={methods.mousedown}
  on:mousedown={methods.mousedown}
  on:touchend={methods.mouseup}
  on:mouseout={methods.mouseup}
  on:mouseup={methods.mouseup}>
  <header>
    {#if value}
      <span class="value left">{value}</span>
    {/if}
    <div class="filler" />
  </header>
  {#if tracker.started}
    <div class="center countdown">
      <Counter started={tracker.started} />
    </div>
  {/if}
  <i class="emoji">{tracker.emoji}</i>
  <footer>{tracker.label}</footer>
  {#if tracker.one_tap}
    <div class="one-tap" />
  {/if}
</button>
