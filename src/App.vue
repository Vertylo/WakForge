<template>
  <div v-if="globalError" class="severe-error-state">
    <p-image :src="wakforgeLogoURL" image-style="width: 140px" />
    <div class="text-lg mt-3">{{ $t('app.globalErrorMessage') }}</div>
    <div class="text-lg mt-1">{{ $t('app.globalErrorContact') }}</div>
    <p-button icon="mdi mdi-discord" class="mt-3" label="Discord Server" @click="onDiscord" />
    <div class="error-message px-3 py-3 mt-4" v-html="globalError?.stack?.replaceAll(' at', '<br \>- at')" />
  </div>
  <div v-else class="flex">
    <AppSidebar />
    <div class="flex flex-column" style="height: 100vh; width: calc(100vw - 130px)">
      <router-view />
      <div class="disclaimer">{{ $t('app.disclaimer') }}</div>
    </div>
  </div>

  <p-confirmPopup group="popup" />
  <p-confirmDialog group="dialog" />
  <p-toast position="top-left" />
  <OldDataDialog ref="oldDataDialog" />
</template>

<script setup>
import { ref, watch, provide, nextTick, onMounted, inject } from 'vue';
import { useRoute } from 'vue-router';
import { EventBus, Events } from '@/eventBus';

import { masterData, useStorage } from '@/models/useStorage.js';
import { useCharacterBuilds } from '@/models/useCharacterBuilds.js';
import { useItems } from '@/models/useItems.js';
import { useStats } from '@/models/useStats';
import { useSpells } from '@/models/spells/useSpells';
import { useLevels } from '@/models/useLevels';
import { useAutoBuilder } from '@/models/useAutoBuilder';

import OldDataDialog from '@/components/OldDataDialog.vue';
import AppSidebar from '@/components/AppSidebar.vue';

import wakforgeLogoURL from '@/assets/images/branding/wakforge.svg';

const route = useRoute();
const oldDataDialog = ref(null);
const globalError = inject('globalError');

// const showSidebar = ref(true);

// First thing we do is grab data from storage
const { setup: storageSetup } = useStorage();
const { errors: storageErrors } = storageSetup();

const { setup: setupCharacterBuilds, setContext } = useCharacterBuilds(masterData);
const { currentCharacter } = setupCharacterBuilds();

const { setup: setupLevels } = useLevels(currentCharacter);
setupLevels();

const { itemFilters, setup: setupItems } = useItems();
const { currentItemList } = setupItems();

const { setup: setupSpells } = useSpells(currentCharacter);
setupSpells();

const { setup: setupStats } = useStats(currentCharacter);
setupStats();

const { setup: setupAutoBuilder } = useAutoBuilder();
setupAutoBuilder();

const setContextIds = () => {
  setContext();
};

watch(
  [() => route.name, () => route.query],
  () => {
    nextTick(() => {
      setContextIds();
    });
  },
  { immediate: true }
);

provide('masterData', masterData);
provide('currentCharacter', currentCharacter);
provide('itemFilters', itemFilters);
provide('currentItemList', currentItemList);

onMounted(() => {
  console.log(
    // eslint-disable-next-line quotes
    "%cIf you're reading this, then I may be able to use your help!\nJoin the Discord server at https://discord.gg/k3v2fXQWJp if you are interested!",
    'font-size: 1rem'
  );
});

EventBus.on(Events.OPEN_OLD_DATA_DIALOG, (data) => {
  setTimeout(() => {
    oldDataDialog.value.open(data);
  }, 100);
});

const onDiscord = () => {
  window.open('https://discord.gg/k3v2fXQWJp', '_blank').focus();
};
</script>

<style lang="scss" scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}

.disclaimer {
  padding: 3px 0;
  width: 100%;
  text-align: center;
  font-size: 14px;
  background-color: var(--secondary-10);
}

.severe-error-state {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  .error-message {
    border: 1px solid var(--error);
    border-radius: 8px;

    :first-child {
      margin-left: 20px;
    }
  }
}
</style>
