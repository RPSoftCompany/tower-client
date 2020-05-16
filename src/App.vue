<template>
  <v-app>
    <v-dialog
      v-model="errorDialog"
      persistent
      max-width="500px"
    >
      <v-card>
        <v-card-title
          class="error"
          primary-title
        >
          Error
        </v-card-title>
        <v-divider />
        <v-card-text class="pt-4">
          {{ errorText }}
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            text
            @click="$store.commit('closeError')"
          >
            OK
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <drawer v-if="loggedIn" />
    <toolbar v-if="loggedIn" />
    <v-content>
      <v-container
        fluid
        style="padding-left: 70px; height: 100%"
      >
        <transition
          name="fade"
          mode="out-in"
        >
          <router-view
            :key="$route.fullPath"
            class="mx-3 mt-4"
          />
        </transition>
      </v-container>
    </v-content>
    <footerElement />
  </v-app>
</template>

<script>
  import drawer from './components/core/drawer'
  import toolbar from './components/core/toolbar'
  import footerElement from './components/core/footerElement'

  export default {
    name: 'App',
    components: {
      drawer,
      toolbar,
      footerElement,
    },
    data: () => ({}),
    computed: {
      loggedIn () {
        return (
          this.$store.state.user !== null &&
          this.$route.name !== 'noPermissions' &&
          this.$route.name !== 'changePassword' &&
          this.$route.name !== 'Login'
        )
      },
      errorDialog () {
        return this.$store.state.error.show
      },
      errorText () {
        return this.$store.state.error.text
      },
    },
    async created () {
      const response = await this.axios.get(
        `${this.$store.state.mainUrl}/configurations/initialized`
      )

      if (response.status === 200) {
        this.$store.state.initialized = response.data
        if (!this.$store.state.initialized) {
          this.$router.push('/initialize')
        }
      }
    },
    async mounted () {
      this.axios.interceptors.response.use(
        response => response,
        e => {
          if (
            this.$route.name !== 'Login' &&
            e.response !== undefined &&
            e.response.data !== undefined &&
            e.response.data.error !== undefined &&
            e.response.data.error.message !== undefined
          ) {
            if (e.response.status === 401) {
              this.$store.commit('setUserData', null)
              this.$store.commit('setUserRoles', [])

              this.$router.push('/login')
            }
            this.$store.commit('setError', e.response.data.error.message)
          }
        }
      )
    },
  }
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition-duration: 0.3s;
  transition-property: opacity;
  transition-timing-function: ease;
}

.fade-enter,
.fade-leave-active {
  opacity: 0;
}

.slowfade-enter-active,
.slowfade-leave-active {
  transition-duration: 0.4s;
  transition-property: opacity;
  transition-timing-function: ease;
}

.slowfade-enter,
.slowfade-leave-active {
  opacity: 0;
}
</style>
