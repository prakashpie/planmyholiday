<template>
  <section
    class="md-layout-rel md-justify-content-center md-layout md-flex-column rx-full-height"
  >
    <section class="md-container md-container--box md-layout-rel">
      <div class="md-flex-column md-layout md-align-items-center md-my-3">
        <div
          id="start"
          class="md-row md-align md-align-items-center"
          style="width: 100%"
        >
          <div
            class="md-col md-d-none md-d-flex-md md-col--6-md md-col--12 md-align-items-center"
          >
            <AuthOnboard />
          </div>
          <div class="md-col md-col--6-md md-col--12">
            <AuthSignin @signinSuccess="onSigninSuccess" />
          </div>
          <div class="md-flex-grow-1"></div>
        </div>
      </div>
    </section>
    <div class="md-flex-grow-1 md-d-flex md-d-none-md" />
    <section class="et-link-block md-container md-container--box md-layout-rel">
      <div class="md-row md-align md-align-items-center">
        <div class="md-col md-col--offset-6-md md-col--12 md-col--6-md">
          <div class="et-login-wrapper">
            <div class="md-border md-d-none md-d-block-md md-mb-2"></div>
          </div>
          <div
            class="et-login-wrapper md-layout md-align-items-center md-justify-content-center"
          >
            Don&apos;t have an account?
            <nuxt-link
              :to="'/signup' + getQueryString"
              class="md-button md-button--accent"
              >Crate an account</nuxt-link
            >
          </div>
        </div>
      </div>
    </section>
  </section>
</template>
<style>
:root {
  --button-text-transform: none;
}
.et-login-button {
  min-width: 128px;
}
.et-full-height {
  height: 100%;
}
.et-link-block {
  width: 100%;
  padding: 0 16px 16px;
}
</style>
<script>
export default {
  middleware: 'guest',
  head() {
    return {
      title: 'Sign in',
      script: [
        {
          async: true,
          defer: true,
          src: 'https://apis.google.com/js/platform.js'
        }
      ]
    }
  },
  layout: 'auth',
  components: {
    AuthOnboard: () => import('@/components/pages/auth/Onboard'),
    AuthSignin: () => import('@/components/pages/auth/Signin')
  },
  data: () => ({
    signup: true,
    actionText: 'OK',
    bannerMsg: '',
    bannerIcon: 'error_outline',
    showIcon: true,
    asyncTab: false,
    currentTab: 'Signup',
    progressId: 'signup-progress',
    banner: '',
    loading: false,
    email: ''
  }),
  computed: {
    getQueryString() {
      return this.$route.fullPath.substring(this.$route.path.length)
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initGoogleAuth()
    })
  },
  beforeDestroy() {
    document.removeEventListener('click', document.getElementById('google-btn'))
    this.$store.commit('core/setAppLoading', false)
  },
  methods: {
    async successHandler(googleUser) {
      const profile = googleUser.getBasicProfile()
      if (this.loading) {
        return false
      }
      this.loading = true
      this.$store.commit('core/setAppLoading', true)
      try {
        await this.$store.dispatch('auth/authGoogle', {
          email: profile.getEmail()
        })
        this.onSigninSuccess()
      } catch (e) {
        // eslint-disable-next-line no-console
        console.log(e)
      }
      this.loading = false
      this.$store.commit('core/setAppLoading', false)
    },
    failedHandler(error) {
      // eslint-disable-next-line no-console
      console.log(error)
    },
    onSigninSuccess() {
      const redirectUrl = this.$route.query.continue
      if (!redirectUrl) {
        this.$router.push('/services')
      } else {
        window.location.href = decodeURIComponent(redirectUrl)
      }
    },
    initGoogleAuth() {
      if (!window.gapi) {
        setTimeout(() => {
          this.initGoogleAuth()
        }, 300)
        return
      }
      const this_ = this
      window.gapi.load('auth2', function() {
        // Retrieve the singleton for the GoogleAuth library and set up the client.
        // eslint-disable-next-line no-undef
        const auth2 = gapi.auth2.init({
          client_id: document.head.querySelector(
            "[name='google-signin-client_id']"
          ).content,
          cookiepolicy: 'single_host_origin'
        })
        const successHandler = this_.successHandler.bind(this)
        const failedHandler = this_.failedHandler.bind(this)
        auth2.attachClickHandler(
          document.getElementById('google-btn'),
          {},
          successHandler,
          failedHandler
        )
      })
    }
  }
}
</script>
