<script setup lang="ts">
import { ExtendedSpace } from '@/helpers/interfaces';
import draggable from 'vuedraggable';

const props = defineProps<{
  space: ExtendedSpace;
  dateStart: number;
  dateEnd: number;
}>();

const {
  form,
  sourceProposalLoaded,
  userSelectedDateStart,
  userSelectedDateEnd,
  formDraft
} = useFormSpaceProposal();

const disableChoiceEdit = computed(() => form.value.type === 'basic');

function addChoices(num) {
  for (let i = 1; i <= num; i++) {
    form.value.choices.push({ key: form.value.choices.length, text: '' });
    formDraft.value.choices.push({ key: form.value.choices.length, text: '' });
  }
}

function setDateStart(ts) {
  form.value.start = ts;
  userSelectedDateStart.value = true;
}

function setDateEnd(ts) {
  form.value.end = ts;
  userSelectedDateEnd.value = true;
}

watch(
  () => form.value.type,
  (newType, oldType) => {
    if (newType !== 'basic' && oldType === 'basic') {
      form.value.choices = [
        { key: 0, text: '' },
        { key: 1, text: '' }
      ];
    }
    if (form.value.type === 'basic') {
      form.value.choices = [
        { key: 1, text: 'For' },
        { key: 2, text: 'Against' },
        { key: 3, text: 'Abstain' }
      ];
    }
  },
  { immediate: true }
);

const { getSnapshot } = useSnapshot();

onMounted(async () => {
  // Initialize the start date to current
  if (!sourceProposalLoaded.value && !userSelectedDateStart.value)
    form.value.start = Number((Date.now() / 1e3).toFixed());
  // Initialize the proposal type if set in space
  if (props.space?.voting?.type) form.value.type = props.space.voting.type;
  form.value.snapshot = await getSnapshot(props.space.network);
});
</script>

<template>
  <div class="mb-5 space-y-4">
    <BaseBlock :title="$t('create.voting')">
      <InputSelectVoteType
        :type="space.voting?.type || form.type"
        :is-disabled="!!space.voting?.type"
        @update:type="value => (form.type = value)"
      />

      <h4 class="mt-3 mb-1" v-text="$t('create.choices')" />
      <div class="flex">
        <div class="w-full overflow-hidden">
          <draggable
            v-model="form.choices"
            v-bind="{ animation: 200 }"
            :disabled="disableChoiceEdit"
            item-key="id"
            handle=".drag-handle"
            class="space-y-2"
          >
            >
            <template #item="{ element, index }">
              <UiInput
                v-model="element.text"
                maxlength="32"
                :disabled="disableChoiceEdit"
                :placeholder="index > 0 ? $t('optional') : ''"
                class="group"
                :focus-on-mount="index === 0"
                :data-testid="`input-proposal-choice-${index}`"
                @update:model-value="formDraft.choices[index].text = $event"
              >
                <template #label>
                  <div
                    class="drag-handle flex cursor-grab items-center active:cursor-grabbing"
                    :class="{
                      'cursor-not-allowed active:cursor-not-allowed':
                        disableChoiceEdit
                    }"
                  >
                    <BaseIcon name="draggable" size="16" class="mr-[12px]" />
                    {{ $tc('create.choice', [index + 1]) }}
                  </div>
                </template>
                <template #info>
                  <span
                    class="hidden text-xs text-skin-text group-focus-within:block"
                  >
                    {{ `${element.text.length}/32` }}
                  </span>
                </template>
              </UiInput>
            </template>
          </draggable>
        </div>
        <div v-if="!disableChoiceEdit" class="ml-2 flex w-[48px] items-end">
          <ButtonSidebar
            v-if="!disableChoiceEdit"
            class="!h-[42px] !w-[42px]"
            @click="addChoices(1)"
          >
            <i-ho-plus-sm class="text-skin-link" />
          </ButtonSidebar>
        </div>
      </div>
    </BaseBlock>

    <BaseBlock
      :title="$t('create.period')"
      :information="$t('create.votingPeriodExplainer')"
    >
      <div class="space-y-2 md:flex md:space-x-3 md:space-y-0">
        <SpaceCreateVotingDateStart
          :delay="space.voting?.delay"
          :date="dateStart"
          @select="value => setDateStart(value)"
        />

        <SpaceCreateVotingDateEnd
          :period="space.voting?.period"
          :date="dateEnd"
          @select="value => setDateEnd(value)"
        />
      </div>
    </BaseBlock>

    <BaseBlock v-if="$route.query.snapshot" :title="$t('snapshot')">
      <UiInput
        v-model="form.snapshot"
        :number="true"
        :placeholder="$t('create.snapshotBlock')"
      >
        <template #label>
          {{ $t('snapshot') }}
        </template>
      </UiInput>
    </BaseBlock>
  </div>
</template>
