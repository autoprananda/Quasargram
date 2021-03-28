<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="bg-white text-grey-10 " bordered>
      <q-toolbar class="constrain">
        <q-btn flat rounded 
          icon="eva-camera-outline"
          dense 
          size="18px" 
          to="/camera"
          class="large-screen-only q-mr-sm"/>
        <q-separator vertical spaced  class="large-screen-only"/>
        <q-toolbar-title class="text-grand-hotel text-bold">
          Quasargram
        </q-toolbar-title>

        <q-btn flat rounded 
          icon="eva-home-outline"
          dense 
          size="18px" 
          to="/"
          class="large-screen-only"/>
      </q-toolbar>
    </q-header>

    <q-footer class="bg-white" bordered>
        <transition appear enter-active-class="animated fadeInUp" leave-active-class="animated fadeOutDown">
        <div v-if="showInstallBanner" class="banner-container bg-primary">
          <div class="constrain">
            <q-banner dense inline-actions class="bg-primary text-white">
              <b>Install Quasargram?</b>

              <template v-slot:action>
                <q-btn flat label="Yes" dense class="q-px-sm" @click="installApp"/>
                <q-btn flat label="Later" dense class="q-px-sm" @click="later" />
                <q-btn flat label="Never" dense class="q-px-sm" @click="neverShowInstallAppBanner" />
              </template>
            </q-banner>
          </div>
        </div>
        </transition>
        <q-tabs class="text-grey-10 small-screen-only" active-color="primary" indicator-color="transparent">
          <q-route-tab to="/" icon="eva-home-outline" />
          <q-route-tab to="/camera" icon="eva-camera-outline" />
        </q-tabs>
    </q-footer> 

    <q-page-container class="bg-grey-1">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
let deferredPrompt;
export default {
  name: 'MainLayout',
  data () {
    return {
      showInstallBanner: false
    }
  },
  mounted() {
    let neverShowInstallAppBanner = this.$q.localStorage.getItem('neverShowInstallAppBanner')
    if (!neverShowInstallAppBanner) {
      window.addEventListener('beforeinstallprompt', (e) => {
        // Prevent the mini-infobar from appearing on mobile
        e.preventDefault();
        // Stash the event so it can be triggered later.
        deferredPrompt = e;
        // Update UI notify the user they can install the PWA
        setTimeout(() => {

          this.showInstallBanner = true
        }, 2000)
      })
    }
  },
  methods: {
    installApp() {
      // Hide the app provided install promotion
      this.showInstallBanner = false
      // Show the install prompt
      deferredPrompt.prompt();
      deferredPrompt.userChoice.then((choiceResult) => {
        if (choiceResult.outcome === 'accepted') {
          console.log('User accepted the install prompt')
          this.neverShowInstallAppBanner()
        } else {
          console.log('User dismissed the install prompt')
        }
      })
    },
    later() {
      this.showInstallBanner = false
    },
    neverShowInstallAppBanner() {
      this.showInstallBanner = false
      this.$q.localStorage.set('neverShowInstallAppBanner', true)
    }
  }
}
</script>
<style lang="sass">
  .q-toolbar
    @media (min-width: $breakpoint-sm-min)
      height: 77px
  .q-toolbar__title
    font-size: 30px
    @media (max-width: $breakpoint-xs-max)
      text-align: center 
  .q-footer 
    .q-tab__icon
    font-size: 30px
</style>