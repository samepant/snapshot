<script setup lang="ts">
import { ExtendedSpace } from '@/helpers/interfaces';

defineProps<{
  space: ExtendedSpace;
  preview: boolean;
  bodyLimit: number;
}>();

const { formatNumber } = useIntl();
const { form, formDraft, getValidation } = useFormSpaceProposal();

const imageDragging = ref(false);
const textAreaEl = ref<HTMLTextAreaElement | null>(null);

const inputName = computed({
  get: () => form.value.name,
  set: value => {
    form.value.name = value;
    formDraft.value.name = value;
  }
});

const inputBody = computed({
  get: () => form.value.body,
  set: value => {
    form.value.body = value;
    formDraft.value.body = value;
    formDraft.value.isBodySet = true;
  }
});

const injectImageToBody = image => {
  const cursorPosition = textAreaEl.value?.selectionStart;
  const currentBody = textAreaEl.value?.value;
  const currentBodyWithImage = `${currentBody?.substring(
    0,
    cursorPosition
  )} \n![${image.name}](${image.url})
    ${currentBody?.substring(cursorPosition as number)}`;

  form.value.body = currentBodyWithImage;
};

const { upload, imageUploadError, isUploadingImage } = useImageUpload();

const handlePaste = e => {
  for (const item of e.clipboardData.items) {
    if (item.kind === 'file' && item.type.startsWith('image/')) {
      const file = item.getAsFile();
      upload(new File([file], 'image', { type: file.type }), injectImageToBody);
    }
  }
};

const handleDrop = e => {
  for (const item of e.dataTransfer.files) {
    if (item.type.startsWith('image/')) {
      upload(item, injectImageToBody);
    }
  }
};
</script>

<template>
  <div class="mb-5 px-4 md:px-0">
    <div class="flex flex-col space-y-3">
      <BlockLink
        v-if="space?.guidelines"
        :title="$t('settings.proposal.guidelines.title')"
        :link="space.guidelines"
      />

      <h1
        v-if="preview"
        class="w-full break-all"
        v-text="form.name || $t('create.untitled')"
      />
      <BaseInput
        v-else
        v-model="inputName"
        :title="$t('create.proposalTitle')"
        :max-length="128"
        :error="getValidation('name')"
        focus-on-mount
        data-testid="input-proposal-title"
      />

      <div v-if="!preview">
        <div class="flex justify-between">
          <LabelInput>
            {{ $t('create.proposalDescription') }}
          </LabelInput>
          <div class="text-xs">
            {{ formatNumber(form.body.length) }} /
            {{ formatNumber(bodyLimit) }}
          </div>
        </div>
        <div
          @drop.prevent="handleDrop"
          @dragover="imageDragging = true"
          @dragleave="imageDragging = false"
        >
          <div
            class="peer min-h-[240px] overflow-hidden rounded-t-xl border focus-within:border-skin-text"
          >
            <textarea
              ref="textAreaEl"
              v-model="inputBody"
              class="s-input mt-0 h-full min-h-[240px] w-full !rounded-xl border-none pt-0 text-base"
              :maxlength="bodyLimit"
              data-testid="input-proposal-body"
              @paste="handlePaste"
            />
          </div>

          <label
            class="relative flex items-center justify-between rounded-b-xl border border-t-0 border-skin-border py-1 px-2 peer-focus-within:border-skin-text"
          >
            <input
              accept="image/jpg, image/jpeg, image/png"
              type="file"
              class="absolute top-0 right-0 bottom-0 left-0 ml-0 w-full p-[5px] opacity-0"
              @change="e => upload((e.target as HTMLInputElement)?.files?.[0], injectImageToBody)"
            />

            <span class="pointer-events-none relative pl-1 text-sm">
              <span v-if="isUploadingImage" class="flex">
                <LoadingSpinner small class="mr-2 -mt-[2px]" />
                {{ $t('create.uploading') }}
              </span>
              <span v-else-if="imageUploadError !== ''">
                {{ imageUploadError }}</span
              >
              <span v-else>
                {{ $t('create.uploadImageExplainer') }}
              </span>
            </span>
            <BaseLink
              v-tippy="{ content: $t('create.markdown') }"
              class="relative inline"
              link="https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax"
              hide-external-icon
            >
              <BaseIcon name="markdown" class="text-skin-text" />
            </BaseLink>
          </label>
        </div>
      </div>

      <div v-if="form.body && preview" class="mb-4">
        <BaseMarkdown :body="form.body" />
      </div>

      <InputUrl
        v-if="!preview"
        v-model.trim="form.discussion"
        placeholder="https://forum.balancer.fi/proposal"
        :title="$t('create.discussion')"
        :error="getValidation('discussion')"
        data-testid="input-proposal-discussion"
      />
    </div>
  </div>
</template>
