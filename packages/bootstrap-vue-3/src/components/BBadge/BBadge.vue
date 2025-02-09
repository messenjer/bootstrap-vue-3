<template>
  <component :is="computedTag" class="badge" :class="classes" v-bind="props">
    <slot />
  </component>
</template>

<script lang="ts">
import {isLink, omit, pluckProps} from '../../utils'
import {computed, defineComponent, PropType} from 'vue'
import type {ColorVariant} from '../../types'
import {BLINK_PROPS} from '../BLink/BLink.vue'

const linkProps = omit(BLINK_PROPS, ['event', 'routerTag'])

export default defineComponent({
  props: {
    pill: {type: Boolean, default: false},
    tag: {type: String, default: 'span'},
    variant: {type: String as PropType<ColorVariant>, default: 'secondary'},
    textIndicator: {type: Boolean, default: false},
    dotIndicator: {type: Boolean, default: false},
    ...linkProps,
  },
  setup(props) {
    const link = computed<boolean>(() => isLink(props))
    const computedTag = computed<string>(() => (link.value ? 'b-link' : props.tag))

    const classes = computed(() => ({
      [`bg-${props.variant}`]: props.variant,
      'active': props.active,
      'disabled': props.disabled,
      'text-dark': ['warning', 'info', 'light'].includes(props.variant),
      'rounded-pill': props.pill,
      'position-absolute top-0 start-100 translate-middle':
        props.textIndicator || props.dotIndicator,
      'p-2 border border-light rounded-circle': props.dotIndicator,
      'text-decoration-none': link,
    }))

    return {
      classes,
      props: link.value ? pluckProps(linkProps, props) : {},
      computedTag,
    }
  },
})
</script>
