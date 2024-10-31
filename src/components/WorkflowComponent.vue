<template>
  <div class="workflow">
    <h2>Workflow : {{ workflow.name }}</h2>
    <h4>Data source : {{ workflow.data_source }}</h4>
    <div v-for="(step, index) in workflow.steps" :key="index" class="step">
      <p>Step {{ index + 1 }}</p>
      <p>Type : {{ step.step_type }}</p>
      <p>Status : {{ step.status }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WorkflowComponent',
  props: {
    workflow: {
      type: Object,
      required: true,
    },
  },
};
</script>

<style scoped>
.workflow {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
}

.step {
  padding: 5px;
  border: 1px solid #ddd;
  margin: 5px 0;
}
</style>