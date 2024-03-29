<script setup lang="ts">
import { PropType } from 'vue';
import { TextInput, SecondaryButton } from '~~/Components';
import { AmpSetting } from '~~/types';

const props = defineProps({
  modelValue: {
    type: Array as PropType<AmpSetting[]>,
    default: () => [],
  },
});

const inputEmitName = 'update:modelValue';

const emits = defineEmits<{
  (eventName: typeof inputEmitName, newValue: typeof props.modelValue): void;
}>();

const updateValue = (newValue: typeof props.modelValue) => {
  emits(inputEmitName, newValue);
};

const addNewSetting = () => {
  updateValue(props.modelValue.concat([{ knob: '', value: '' }]));
};

const removeSetting = (settingsKey: number) => {
  if (settingsKey >= props.modelValue.length) return;

  const firstPart = props.modelValue.slice(0, settingsKey);
  const secondPart = props.modelValue.slice(settingsKey + 1);
  updateValue(firstPart.concat(secondPart));
};

const updateSettingKnob = (newSettingsKey: string, settingsKey: number) => {
  const newSetting = {
    knob: newSettingsKey,
    value: props.modelValue[settingsKey].value,
  };
  const firstPart = props.modelValue.slice(0, settingsKey).concat([newSetting]);
  const secondPart = props.modelValue.slice(settingsKey + 1);
  updateValue(firstPart.concat(secondPart));
};

const updateSettingValue = (newSettingsValue: string, settingsKey: number) => {
  const newSetting = {
    knob: props.modelValue[settingsKey].knob,
    value: newSettingsValue,
  };
  const firstPart = props.modelValue.slice(0, settingsKey).concat([newSetting]);
  const secondPart = props.modelValue.slice(settingsKey + 1);
  updateValue(firstPart.concat(secondPart));
};
</script>

<template>
  <div>
    <div
      v-for="(setting, settingKey) in modelValue"
      :key="settingKey"
      class="flex"
    >
      <div class="mb-2 mr-2 flex-initial w-1/4">
        <TextInput
          id="name"
          :model-value="setting.knob"
          type="text"
          class="w-full"
          placeholder="Knob"
          @input="updateSettingKnob($event.target.value, settingKey)"
        />
      </div>
      <div class="mb-2 mr-2 flex-initial w-3/4">
        <TextInput
          id="name"
          :model-value="setting.value ?? ''"
          type="text"
          class="w-full"
          placeholder="Value"
          @input="updateSettingValue($event.target.value, settingKey)"
        />
      </div>
      <div class="flex-none">
        <SecondaryButton
          type="button"
          class="py-2 px-2 mt-1 rounded-full"
          @click="removeSetting(settingKey)"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            stroke="currentColor"
            class="w-4 h-4 stroke-2 fill-none"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </SecondaryButton>
      </div>
    </div>
    <SecondaryButton @click="addNewSetting">Add</SecondaryButton>
  </div>
</template>
