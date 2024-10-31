<template>
  <div class="flex items-center mb-2">
    <slot name="input">
      <input 
        v-model="localStep" 
        type="text" 
        placeholder="Type d'étape" 
        class="border rounded p-2 mr-2"
        @input="updateStep"
      />
    </slot>
    <button 
      @click="removeStep" 
      class="bg-red-500 text-white rounded p-2 hover:bg-red-600"
    >
      Supprimer étape
    </button>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  name: 'StepInput',
  props: {
    initialStep: String,
  },
  setup(props, { emit }) {
    const localStep = ref(props.initialStep);

    function updateStep() {
      emit('update', localStep.value);
    }

    function removeStep() {
      emit('remove');
    }

    watch(() => props.initialStep, (newVal) => {
      localStep.value = newVal;
    });

    return {
      localStep,
      updateStep,
      removeStep,
    };
  },
};
</script>