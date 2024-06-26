<template>
  <router-view />
</template>

<script setup lang="ts">
import { ConnectionState, api } from '@/plugins/api';
import { onMounted } from 'vue';
import { useTheme } from 'vuetify';
import { store } from '@/plugins/store';
import { i18n } from '@/plugins/i18n';
import router from './plugins/router';
import { sleep } from './helpers/utils';
const theme = useTheme();

const setTheme = function () {
  const themePref = localStorage.getItem('frontend.settings.theme') || 'auto';
  if (themePref == 'dark') {
    // forced dark mode
    theme.global.name.value = 'dark';
  } else if (themePref == 'light') {
    // forced light mode
    theme.global.name.value = 'light';
  } else if (
    window.matchMedia &&
    window.matchMedia('(prefers-color-scheme: dark)').matches
  ) {
    // dark mode is enabled in browser
    theme.global.name.value = 'dark';
  } else {
    // light mode is enabled in browser
    theme.global.name.value = 'light';
  }
};

onMounted(() => {
  // @ts-ignore
  store.isInStandaloneMode = window.navigator.standalone || false;

  // set navigation menu style
  store.navigationMenuStyle =
    localStorage.getItem('frontend.settings.menu_style') || 'horizontal';

  // cache some settings in the store
  store.allowExternalImageRetrieval =
    (localStorage.getItem('frontend.settings.artwork_pref') || 'online') ==
    'online';

  const langPref = localStorage.getItem('frontend.settings.language') || 'auto';
  if (langPref !== 'auto') {
    i18n.global.locale.value = langPref;
  }

  // set color theme (and listen for color scheme changes from browser)
  setTheme();
  window
    .matchMedia('(prefers-color-scheme: dark)')
    .addEventListener('change', setTheme);
  // Initialize API Connection
  // TODO: retrieve serveraddress through discovery and/or user settings ?
  let serverAddress = '';
  if (process.env.NODE_ENV === 'development') {
    serverAddress = localStorage.getItem('mass_debug_address') || '';
    if (!serverAddress) {
      serverAddress =
        prompt(
          'Enter location of the Music Assistant server',
          window.location.origin.replace('3000', '8095'),
        ) || '';
      localStorage.setItem('mass_debug_address', serverAddress);
    }
  } else {
    const loc = window.location;
    serverAddress = loc.origin + loc.pathname;
  }
  api.initialize(serverAddress);

  // very rude way to redirect the user to the settings page if this is a fresh install
  sleep(1000).then(() => {
    if (
      api.state.value === ConnectionState.CONNECTED &&
      !api.setUpCompleted.value
    ) {
      router.push('/settings');
    }
  });
});
</script>
