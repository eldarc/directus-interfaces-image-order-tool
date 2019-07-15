<template>
  <span class="no-wrap">{{ images }}</span>
</template>

<script>
import mixin from '@directus/extension-toolkit/mixins/interface'

export default {
  mixins: [mixin],
  computed: {
    images () {
      return this.value.map(image => `"${image.title}"`).join(', ')
    }
  }
}
</script>
