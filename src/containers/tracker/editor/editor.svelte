<script>
  // components

  import NItem from "../../../components/list-item/list-item.svelte";
  import NModal from "../../../components/modal/modal.svelte";
  import NToggle from "../../../components/toggle-switch/toggle-switch.svelte";
  import ColorPicker from "../../../components/color-picker/color-picker.svelte";

  // Utils
  import { createEventDispatcher } from "svelte";
  import NomieUOM from "../../../utils/nomie-uom/nomie-uom";

  // modules
  import Tracker from "../../../modules/tracker/tracker";
  import TrackerTypes from "../../../modules/tracker-types/tracker-types";

  // Stores
  import config from "../../../../config/global";
  import { UserStore } from "../../../store/user";
  import { Interact } from "../../../store/interact";
  import { TrackerStore } from "../../../store/trackers";

  const dispatch = createEventDispatcher();

  export let tracker = new Tracker();
  export let show = false;

  let data = {
    groupedUOMs: NomieUOM.toGroupedArray(),
    types: Object.keys(TrackerTypes).map(id => {
      let type = TrackerTypes[id];
      type.id = id;
      return type;
    })
  };

  $: if (tracker.type === "timer") {
    tracker.uom = "timer";
    tracker.min = null;
    tracker.max = null;
  } else if (tracker.uom == "timer" && tracker.type != "timer") {
    tracker.uom = "num";
  } else if (tracker.type === "range" && !tracker.min) {
    tracker.min = 1;
    tracker.max = 10;
  }

  const methods = {
    tracker_save() {
      if (!tracker.tag || !tracker.label) {
        Interact.alert(
          "Missing Data",
          "Please fill out all required fields: title, tag and emoji"
        );
      } else {
        TrackerStore.saveTracker(tracker).then(() => {
          dispatch("save", tracker);
        });
      }
    },
    labelChanged(event) {
      if (tracker._dirty) {
        let tag = event.target.value.replace(/[^A-Z0-9]/gi, "_").toLowerCase();
        tracker.tag = tag;
      }
    },
    cancel() {
      tracker = new Tracker();
      dispatch("close");
    }
  };
</script>

<style lang="scss">
  .n-tracker-editor {
    select {
      min-width: 190px;
      text-align: right;
    }
    .item-divider.compact {
      background-color: var(--color-faded-1);
    }
  }
</style>

<div class="n-tracker-editor">
  <NModal title="Tracker Editor" isVisible={show} allowClose>
    <div class="bg-faded">
      <NItem className="item-divider compact" />
      <ColorPicker bind:value={tracker.color} />
      <NItem className="item-divider compact" />
      <NItem title="Label">
        <div slot="right">
          <input
            type="text"
            class="form-control"
            bind:value={tracker.label}
            on:keyup={methods.labelChanged} />
        </div>
      </NItem>
      <NItem title="Emoji">
        <div slot="right">
          <input
            type="text"
            on:focus={event => {
              event.target.select();
            }}
            maxlength="3"
            class="form-control text-center"
            bind:value={tracker.emoji} />
        </div>
      </NItem>
      {#if tracker._dirty}
        <NItem title="Tag">
          <div slot="right">
            <input
              type="text"
              class="form-control"
              bind:value={tracker.tag}
              autocomplete="off"
              autocorrect="off"
              maxlength="10"
              autocapitalize="off"
              spellcheck="false" />
          </div>
        </NItem>
      {/if}
      <NItem className="item-divider compact" />
      <NItem title="Type">
        <div slot="right">
          <select class="form-control w-100" bind:value={tracker.type}>
            {#each data.types as type}
              <option value={type.id}>{type.label}</option>
            {/each}
          </select>
        </div>

      </NItem>
      {#if tracker.type == 'tick'}
        <NItem title="Auto Save On Tap">
          <div slot="right">
            <NToggle bind:value={tracker.one_tap} />
          </div>
        </NItem>
      {/if}
      {#if tracker.type == 'range'}
        <NItem title="Min">
          <div slot="right">
            <input type="text" class="form-control" bind:value={tracker.min} />
          </div>
        </NItem>
        <NItem title="Max" borderBottom>
          <div slot="right">
            <input type="text" class="form-control" bind:value={tracker.max} />
          </div>
        </NItem>
      {/if}
      <NItem className="item-divider compact" />

      {#if tracker.type !== 'timer'}
        <NItem>
          <div class="title truncate">Measure By</div>
          <div slot="right">
            <select bind:value={tracker.uom} class="form-control">
              {#each Object.keys(data.groupedUOMs) as groupKey (groupKey)}
                <option disabled>-- {groupKey}</option>
                {#each data.groupedUOMs[groupKey] as uom (`${groupKey}-${uom.key}`)}
                  <option
                    value={uom.key}
                    disabled={uom.key == 'time' && tracker.type != 'timer'}>
                    {NomieUOM.plural(uom.key)}
                  </option>
                {/each}
              {/each}
            </select>
          </div>
        </NItem>
      {/if}

      <NItem title="Calculate">
        <div slot="right">
          <select class="form-control" bind:value={tracker.math}>
            {#each ['sum', 'avg'] as math_key}
              <option value={math_key}>{math_key}</option>
            {/each}
          </select>
        </div>
      </NItem>
      <NItem className="item-divider compact" />
    </div>

    <button
      slot="footer"
      on:click={methods.cancel}
      class="btn btn-light btn-lg flex-grow mr-1">
      Cancel
    </button>
    <button
      slot="footer"
      class="btn btn-primary btn-lg flex-grow ml-1"
      on:click={methods.tracker_save}>
      Save
    </button>
  </NModal>
</div>
