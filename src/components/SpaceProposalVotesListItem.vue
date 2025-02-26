<script lang="ts" setup>
import { ExtendedSpace, Proposal, Vote, Profile } from '@/helpers/interfaces';
import { shorten, getIpfsUrl } from '@/helpers/utils';

const props = defineProps<{
  space: ExtendedSpace;
  proposal: Proposal;
  vote: Vote;
  profiles: Record<string, Profile>;
}>();

defineEmits(['openReceiptModal']);

const authorIpfsHash = ref('');
const relayerIpfsHash = ref('');

const titles = computed(() =>
  props.proposal.strategies.map(strategy => strategy.params.symbol || '')
);

const { formatCompactNumber } = useIntl();
</script>

<template>
  <div class="flex items-center gap-3 border-t px-3 py-[14px]">
    <BaseUser
      :key="vote.voter"
      :profile="profiles[vote.voter]"
      :address="vote.voter"
      :space="space"
      :proposal="proposal"
      width-class="w-[110px] min-w-[110px] xs:w-[130px] xs:min-w-[130px] text-left"
    />

    <SpaceProposalVotesListItemChoice :proposal="proposal" :vote="vote" />
    <div
      class="flex w-[110px] min-w-[110px] items-center justify-end whitespace-nowrap text-right text-skin-link xs:w-[130px] xs:min-w-[130px]"
    >
      <span
        v-tippy="{
          content: vote.scores
            ?.map(
              (score, index) => `${formatCompactNumber(score)} ${titles[index]}`
            )
            .join(' + ')
        }"
      >
        {{
          `${formatCompactNumber(vote.balance)} ${shorten(
            proposal.symbol || space.symbol,
            'symbol'
          )}`
        }}
      </span>
      <BasePopover>
        <template #button>
          <BaseButtonIcon @click="authorIpfsHash = vote.ipfs">
            <BaseIcon name="signature" />
          </BaseButtonIcon>
        </template>
        <template #content>
          <div class="m-4 space-y-4">
            <h3 class="text-center">{{ $t('receipt') }}</h3>
            <BaseBlock slim class="p-4 text-skin-link">
              <div class="flex">
                <span
                  class="mr-1 flex-auto text-skin-text"
                  v-text="$t('author')"
                />
                <BaseLink
                  :link="getIpfsUrl(authorIpfsHash)"
                  class="text-skin-link"
                >
                  #{{ authorIpfsHash.slice(0, 7) }}
                </BaseLink>
              </div>
              <div v-if="relayerIpfsHash" class="flex">
                <span
                  class="mr-1 flex-auto text-skin-text"
                  v-text="$t('relayer')"
                />
                <BaseLink
                  :link="getIpfsUrl(relayerIpfsHash)"
                  class="text-skin-link"
                >
                  #{{ relayerIpfsHash.slice(0, 7) }}
                </BaseLink>
              </div>
            </BaseBlock>
            <BaseLink
              :link="`https://signator.io/view?ipfs=${authorIpfsHash}`"
              class="mb-2 block"
              hide-external-icon
            >
              <BaseButton class="w-full">
                {{ $t('verifyOnSignatorio') }}
                <i-ho-external-link
                  class="ml-1 mb-[2px] inline-block text-xs"
                />
              </BaseButton>
            </BaseLink>
          </div>
        </template>
      </BasePopover>
      <BaseButtonIcon
        v-if="vote.reason !== '' && props.proposal.privacy !== 'shutter'"
        v-tippy="{
          content: vote.reason
        }"
        class="cursor-default p-0"
      >
        <i-ho-annotation class="text-[16px]" />
      </BaseButtonIcon>
    </div>
  </div>
</template>
