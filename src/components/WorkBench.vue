<template>
  <div
    :class="[bemm(), bemm('', getHistoryLength ? 'has-history' : 'no-history')]"
  >
    <div :class="bemm('search')">
      <input type="text" v-model="find" placeholder="Find..." />
      <input type="text" v-model="replace" placeholder="Replace.." />
    </div>
    <div :class="bemm('history')">
      <ul :class="bemm('list')">
        <li
          :class="[
            bemm('item'),
            bemm('item', idx == historyStep ? 'active' : 'inactive'),
          ]"
          v-for="(item, idx) in input"
          :key="idx"
        >
          <span :class="bemm('action')">{{ item.action }}</span>
          <span :class="bemm('text')">{{ item.text }}</span>
        </li>
      </ul>
      <button v-if="getHistoryLength > 1" @click="actions.backHistory(0)">
        Back to Original
      </button>
      <button v-if="getHistoryLength > 1" @click="actions.backHistory(1)">
        ({{ getHistoryLength }}) Go Back
      </button>
    </div>
    <div :class="bemm('input')">
      <textarea :value="getCurrentText" @change="setCurrentText"></textarea>
    </div>
    <div :class="bemm('actions')">
      <h3>Actions</h3>
      <button @click="actions.removeCapitals">Remove All Capitals</button>
      <button @click="actions.lowerCaseAll">Lowercase all</button>
      <button @click="actions.upperCaseAll">Uppercase all</button>
      <button @click="actions.replaceAll" v-if="find.length > 0">
        <template v-if="replace.length">
          Replace all <strong>{{ find }}</strong> with
          <strong>{{ replace }}</strong>
        </template>
        <template v-else>
          Remove all <strong>{{ find }}</strong>
        </template>
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from "vue";
import { createBemm } from "bemm";

export default defineComponent({
  setup() {
    const bemm = createBemm("workbench");

    enum Action {
      SET_INPUT = "setInput",
      BACK_HISTORY = "backHistory",
      SET_HISTORY = "setHistory",
      REMOVE_CAPITALS = "removeCapitals",
      LOWERCASE_ALL = "lowercaseAll",
      UPPERCASE_ALL = "uppercaseAll",
      REPLACE_ALL = "replaceAll",
    }
    interface History {
      action: Action;
      text: string;
      step: number;
    }
    const find = ref<string>("");
    const replace = ref<string>("");
    const input = ref<History[]>([]);
    const historyStep = ref(-1);

    const setInput = (str: string, action: Action, history = 0): void => {
      input.value.push({ text: str, action: action, step: historyStep.value });
      historyStep.value = history ? history : historyStep.value + 1;
    };

    const getInputText = (step: number): string =>
      input.value.find((value) => value.step == step)?.text || "";

    const getCurrentText = computed(() => {
      const currentStep = input.value[historyStep.value];
      return currentStep?.text || "";
    });

    const setCurrentText = (event: any): void => {
      const newValue = event.target.value;
      const currentStep = input.value[historyStep.value];
      if (currentStep?.text) currentStep.text = newValue;
      else {
        setInput(newValue, Action.SET_INPUT);
      }
    };

    const getHistoryLength = computed(() => Object.keys(input.value).length);

    const actions = {
      backHistory: (step: number) => {
        setInput(
          getInputText(step > 0 ? input.value.length - step : 1),
          Action.BACK_HISTORY,
          step > 0 ? step : 0
        );
      },
      removeCapitals: () => {
        setInput(
          getCurrentText.value.replace(/[A-Z]*/g, ""),
          Action.REMOVE_CAPITALS
        );
      },
      lowerCaseAll: () => {
        setInput(getCurrentText.value.toLowerCase(), Action.LOWERCASE_ALL);
      },
      upperCaseAll: () => {
        setInput(getCurrentText.value.toUpperCase(), Action.UPPERCASE_ALL);
      },
      replaceAll: () => {
        setInput(
          getCurrentText.value.replaceAll(find.value, replace.value),
          Action.REPLACE_ALL
        );
      },
    };

    return {
      bemm,
      input,
      find,
      replace,
      actions,
      historyStep,
      getHistoryLength,
      getCurrentText,
      setCurrentText,
    };
  },
});
</script>

<style lang="scss">
.workbench {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-areas:
    "search search search search"
    "history input input input"
    "actions actions actions actions";
  gap: 1em;
  padding: 1em;
  width: 100%;
  max-width: 960px;
  margin: auto;
  &--no-history {
    grid-template-areas:
      "search search search search"
      "input input input input"
      "actions actions actions actions";
  }

  &__search {
    grid-area: search;
    display: flex;
    gap: 1em;
  }
  &__input {
    grid-area: input;
    width: 100%;
  }
  &__history {
    grid-area: history;
  }
  &__actions {
    grid-area: actions;
    display: flex;
    gap: 0.5em;
    align-items: center;
  }
  &__list {
    margin: 0;
    display: flex;
    gap: 0.5em;
    padding: 0;
    flex-direction: column;
  }
  &__item {
    padding: 1em;
    border-radius: 4px;
    background-color: rgb(179, 223, 255);
    list-style-type: none;

    &--inactive {
      opacity: 0.5;
    }
  }
  // HISTORY

  &__text {
    display: block;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    width: 200px;
  }
  &__action {
    display: block;
    font-weight: bold;
    text-transform: uppercase;
    opacity: 0.5;
    font-size: 0.75em;
    & + * {
      margin-top: 0.5em;
    }
  }
}

input,
textarea {
  margin: 0;
  padding: 1em;
  width: 100%;
  border-radius: 4px;
  border: none;
  box-shadow: 0 0 1em 0 rgba(0, 0, 0, 0.15);
  font-family: inherit;

  &:placeholder-shown:not(:focus) {
    opacity: 0.5;
  }
  &:focus {
    outline: 2px solid rgb(179, 223, 255);
  }
}
textarea {
  min-height: 50vh;
}

button {
  padding: 0.5em 1em;
  line-height: 1.5;
  border-radius: 0.25em;
  border: none;
  background-color: #cf8;
}
</style>
