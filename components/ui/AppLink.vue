<template>
  <component v-bind:is="linkProps(to).is" v-bind="linkProps(to)">
    <slot />
  </component>
</template>

<script>
export default {
  props: {
    to: {
      type: String,
      required: true
    }
  },
  methods: {
    linkProps(url) {
      if (url.match(/^(http(s)?|ftp):\/\//)) {
        return {
          is: 'a',
          href: url,
          target: '_blank',
          rel: 'noopener'
        }
      }
      return {
        is: 'nuxt-link',
        to: url
      }
    }
  }
}
</script>
