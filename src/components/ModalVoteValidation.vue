<script setup lang="ts">
import { VoteValidation } from '@/helpers/interfaces';
import { clone } from '@snapshot-labs/snapshot.js/src/utils';
import { validateForm } from '@/helpers/validation';

const DEFAULT_PARAMS: Record<string, any> = {};

const props = defineProps<{ open: boolean; validation: VoteValidation }>();

const emit = defineEmits(['add', 'close']);

const { open } = toRefs(props);

const isValidJson = ref(true);
const formRef = ref();

const input = ref({
  name: '',
  params: clone(DEFAULT_PARAMS)
});

type Validations = Record<
  string,
  {
    key: string;
    example?: Record<string, any>[];
    schema?: Record<string, any>;
    about?: string;
  }
>;

const validations = ref<Validations | null>(null);
const isValidationsLoaded = ref(false);

const validationDefinition = computed(() => {
  return (
    validations.value?.[input.value.name]?.schema?.definitions?.Validation ||
    null
  );
});

const validationErrors = computed(() => {
  return validateForm(validationDefinition.value || {}, input.value.params);
});

const isValid = computed(() => {
  return Object.values(validationErrors.value).length === 0;
});

function select(n: string) {
  input.value.name = n;

  if (props.validation.name !== n) {
    input.value.params = clone(DEFAULT_PARAMS);
    if (n === 'basic') {
      input.value.params = {
        minScore: 1,
        strategies: [
          {
            name: 'ticket',
            network: '1',
            params: {
              symbol: 'DAI'
            }
          }
        ]
      };
    }
  }

  if (n === 'any') {
    handleSubmit();
  }
}

function handleSubmit() {
  if (!isValid.value || !isValidJson.value)
    return formRef?.value?.forceShowError();

  emit('add', clone(input.value));
  emit('close');
}

async function getValidations() {
  if (validations.value) return;
  const fetchedValidations: Validations = await fetch(
    `${import.meta.env.VITE_SCORES_URL}/api/validations`
  ).then(res => res.json());
  const validationsWithAny = {
    any: {
      key: 'any'
    },
    ...fetchedValidations
  };
  validations.value = validationsWithAny || null;
  isValidationsLoaded.value = true;
}

watch(open, () => {
  getValidations();
  input.value.name = '';
  if (props.validation?.params) {
    input.value.params = props.validation.params;
  } else {
    input.value = {
      name: '',
      params: clone(DEFAULT_PARAMS)
    };
  }
});
</script>

<template>
  <BaseModal :open="open" @close="$emit('close')">
    <template #header>
      <h3>
        {{
          input.name
            ? $t('votingValidation.settingsTitle')
            : $t('votingValidation.title')
        }}
      </h3>
    </template>

    <div class="my-4 mx-0 min-h-[250px] md:mx-4">
      <div v-if="input.name" class="text-skin-link">
        <TuneForm
          v-if="validationDefinition"
          ref="formRef"
          v-model="input.params"
          :definition="validationDefinition"
          :error="validationErrors"
        />
        <TuneTextareaJson
          v-else
          v-model="input.params"
          :placeholder="$t('votingValidation.paramPlaceholder')"
          @update:is-valid="value => (isValidJson = value)"
        />
      </div>
      <div v-if="!input.name">
        <LoadingRow v-if="!isValidationsLoaded" block class="px-0" />
        <div v-else class="space-y-3">
          <BaseModalSelectItem
            v-for="v in validations"
            :key="v.key"
            :title="$t(`votingValidation.${v.key}.label`)"
            :description="$t(`votingValidation.${v.key}.description`)"
            :selected="validation.name === v.key"
            :tag="v.key === 'passport-gated' ? 'Beta' : ''"
            @click="select(v.key)"
          />
        </div>
      </div>
    </div>
    <template v-if="input.name" #footer>
      <BaseButton class="w-full" primary @click="handleSubmit">
        {{ $t('save') }}
      </BaseButton>
    </template>
  </BaseModal>
</template>
