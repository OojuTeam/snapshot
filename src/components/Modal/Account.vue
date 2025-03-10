<script setup>
import { toRefs, ref, watch, computed } from 'vue';
import { getInjected } from '@snapshot-labs/lock/src/utils';
import { shorten, explorerUrl, getIpfsUrl } from '@/helpers/utils';
import connectors from '@/helpers/connectors.json';
import { useWeb3 } from '@/composables/useWeb3';

const props = defineProps(['open']);

const emit = defineEmits(['login', 'close']);

const { open } = toRefs(props);
const { web3, logout } = useWeb3();

const step = ref(null);

const injected = computed(() => getInjected());

async function handleLogout() {
  await logout();
  emit('close');
}

const path =
    // TODO make this an environment variable
  './assets';

watch(open, () => (step.value = null));
</script>

<template>
  <UiModal :open="open" @close="$emit('close')">
    <template v-slot:header>
      <h3 v-if="!web3.account || step === 'connect'">
        {{ $t('connectWallet') }}
      </h3>
      <h3 v-else>{{ $t('account') }}</h3>
    </template>
    <div v-if="!web3.account || step === 'connect'">
      <div class="m-4 space-y-2">
        <a
          v-for="(connector, id, i) in connectors"
          :key="i"
          @click="$emit('login', connector.id)"
          target="_blank"
          class="block"
        >
          <UiButton
            v-if="id === 'injected' && injected"
            class="button-outline w-full flex justify-center items-center"
          >
            <img
              :src="`${path}/${injected.id}.png`"
              height="28"
              width="28"
              class="mr-2 -mt-1"
              :alt="injected.name"
            />
            {{ injected.name }}
          </UiButton>
          <UiButton
            v-else-if="id !== 'gnosis'"
            class="button-outline w-full flex justify-center items-center gap-2"
          >
            <img
              :src="`${path}/${connector.id}.png`"
              height="25"
              width="25"
              :alt="connector.name"
            />
            <span>{{ connector.name }}</span>
          </UiButton>
        </a>
      </div>
    </div>
    <div v-else>
      <div v-if="$auth.isAuthenticated.value" class="m-4 space-y-2">
        <a
          :href="explorerUrl(web3.network.key, web3.account, /*klaytn's scope wants the string 'account' in the URL*/ 'account')"
          target="_blank"
          class="block"
        >
          <UiButton
            class="button-outline w-full flex justify-center items-center"
          >
            <UiAvatar
              :imgsrc="getIpfsUrl(web3.profile?.image)"
              :address="web3.account"
              size="18"
              class="mr-2 -ml-1"
            />
            <span v-if="web3.profile.name" v-text="web3.profile.name" />
            <span v-else-if="web3.profile.ens" v-text="web3.profile.ens" />
            <span v-else v-text="shorten(web3.account)" />
            <Icon name="external-link" class="ml-1" />
          </UiButton>
        </a>
        <UiButton @click="step = 'connect'" class="button-outline w-full">
          {{ $t('connectWallet') }}
        </UiButton>
        <UiButton @click="handleLogout" class="button-outline w-full !text-red">
          {{ $t('logout') }}
        </UiButton>
      </div>
    </div>
  </UiModal>
</template>
