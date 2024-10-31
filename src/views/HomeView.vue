<template>
  <div class="mx-4">
    <p>{{ state.message }}</p>
    
    <p v-if="lastWorkflowTitle">Dernier workflow créé : {{ lastWorkflowTitle }}</p>

    <div class="mb-4">
      <input 
        v-model="state.newWorkflowTitle" 
        type="text" 
        placeholder="Titre" 
        class="border rounded p-2 mr-2"
      />
      <input 
        v-model="state.dataSource" 
        type="text" 
        placeholder="Data Source" 
        class="border rounded p-2 mr-2"
      />
      <button 
        @click="addWorkflow" 
        class="bg-blue-500 text-white rounded p-2 hover:bg-blue-600"
      >
        Ajouter Workflow
      </button>
    </div>

    <div class="mb-4">
      <h2>Étapes</h2>
      <div v-for="(step, index) in state.steps" :key="index" class="mb-2">
        <StepInput 
          :initialStep="step.step_type" 
          @update="updateStep(index, $event)" 
          @remove="removeStep(index)"
        >
          <template #input>
            <input 
              v-model="state.steps[index].step_type"
              type="text" 
              placeholder="Type personnalisé" 
              class="border rounded p-2 mr-2"
            />
          </template>
        </StepInput>
      </div>
      <button 
        @click="addStep" 
        class="bg-green-500 text-white rounded p-2 hover:bg-green-600"
      >
        Ajouter étape
      </button>
    </div>

    <div class="mb-4">
      <h2>Uploader un fichier</h2>
      <input 
        type="file" 
        @change="handleFileUpload" 
        class="border rounded p-2 mr-2"
      />
      <p v-if="uploadedFileName">Fichier sélectionné : {{ uploadedFileName }}</p>
    </div>

    <div v-if="state.workflows" class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <WorkflowComponent 
        v-for="(workflow, index) in state.workflows" 
        :key="index" 
        :workflow="workflow" 
        class="border rounded p-4"
      />
    </div>

    <div class="mt-4">
      <h2>Compteur (ref + setTimeout) : {{ counter }}</h2>
      <button @click="incrementCounter" class="bg-blue-500 text-white rounded p-2">++</button>
    </div>
  </div>
</template>

<script>
import WorkflowComponent from '@/components/WorkflowComponent.vue';
import StepInput from '@/components/StepInput.vue';
import { reactive, ref, computed } from 'vue';

export default {
  name: 'HomeView',
  components: {
    WorkflowComponent,
    StepInput,
  },
  setup() {
    const state = reactive({
      message: '',
      workflows: [],
      newWorkflowTitle: '',
      dataSource: '',
      steps: [
        {
          step_type: '',
          parameters: {},
          status: 'created',
        }
      ],
    });

    const counter = ref(0);
    const uploadedFileName = ref('');

    function setCookie(name, value, days) {
      const d = new Date();
      d.setTime(d.getTime() + (days * 24 * 60 * 60 * 1000));
      const expires = "expires=" + d.toUTCString();
      document.cookie = name + "=" + value + ";" + expires + ";path=/";
    }

    function getCookie(name) {
      const nameEQ = name + "=";
      const ca = document.cookie.split(';');
      for(let i = 0; i < ca.length; i++) {
        let c = ca[i];
        while (c.charAt(0) === ' ') c = c.substring(1, c.length);
        if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
      }
      return null;
    }

    const lastWorkflowTitle = computed(() => {
      return getCookie('lastWorkflowTitle');
    });

    async function created() {
      try {
        const responseGetHello = await window.axios.get('http://localhost:3001');
        state.message = responseGetHello.data;

        const responseGetWorkflows = await window.axios.get('http://localhost:3001/workflows');
        state.workflows = responseGetWorkflows.data;
      } catch (error) {
        console.error('Erreur lors de la récupération du message:', error);
      }
    }

    async function addWorkflow() {
      const newWorkflow = {
        name: state.newWorkflowTitle,
        data_source: state.dataSource,
        steps: state.steps,
      };

      try {
        const responsePost = await fetch('http://localhost:3001/workflow', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(newWorkflow)
        });
        if (!responsePost.ok) throw new Error('Erreur lors de l\'ajout du workflow');

        setCookie('lastWorkflowTitle', state.newWorkflowTitle, 7);

        state.newWorkflowTitle = '';
        state.dataSource = '';
        state.steps = [{ step_type: '', parameters: {}, status: 'created' }];

        const responseGetWorkflows = await fetch('http://localhost:3001/workflows');
        if (!responseGetWorkflows.ok) throw new Error('Erreur lors de la récupération des workflows');
        state.workflows = await responseGetWorkflows.json();
      } catch (error) {
        console.error('Erreur lors de l\'ajout du workflow:', error);
      }
    }

    function addStep() {
      state.steps.push({ step_type: '', parameters: {}, status: 'created' });
    }

    function removeStep(index) {
      state.steps.splice(index, 1);
    }

    function updateStep(index, newTitle) {
      state.steps[index].step_type = newTitle;
    }

    const handleFileUpload = (event) => {
      const file = event.target.files[0];
      if (file) {
        uploadedFileName.value = file.name;
      }
    };

    const incrementCounter = () => {
      setTimeout(() => {
        counter.value++;
      }, 1000);
    };

    created();

    return {
      state,
      counter,
      addWorkflow,
      addStep,
      removeStep,
      updateStep,
      incrementCounter,
      lastWorkflowTitle,
      uploadedFileName,
      handleFileUpload,
    };
  },
};
</script>