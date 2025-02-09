<template>
  <component :is="computedTag" class="btn" :class="classes" v-bind="attrs" @click="clicked">
    <slot />
  </component>
</template>

<script lang="ts">
import {computed, defineComponent, PropType} from 'vue'
import type {ButtonVariant, InputSize, LinkTarget} from '../../types'
import {BLINK_PROPS} from '../BLink/BLink.vue'

export default defineComponent({
  props: {
    ...BLINK_PROPS,
    active: {type: Boolean, default: false},
    disabled: {type: Boolean, default: false},
    href: {type: String, required: false},
    pill: {type: Boolean, default: false},
    pressed: {type: Boolean, default: null},
    rel: {type: String, default: null},
    size: {type: String as PropType<InputSize>},
    squared: {type: Boolean, default: false},
    tag: {type: String, default: 'button'},
    target: {type: String as PropType<LinkTarget>, default: '_self'},
    type: {type: String, default: 'button'},
    variant: {type: String as PropType<ButtonVariant>, default: 'secondary'},
  },
  emits: ['click', 'update:pressed'],
  setup(props, {emit}) {
    const isToggle = props.pressed !== null
    const isButton = props.tag === 'button' && !props.href && !props.to
    const isLink = !!(props.href || props.to)
    const isBLink = !!props.to
    const nonStandardTag = props.href ? false : !isButton

    const classes = computed(() => ({
      [`btn-${props.variant}`]: props.variant,
      [`btn-${props.size}`]: props.size,
      'active': props.active || props.pressed,
      'rounded-pill': props.pill,
      'rounded-0': props.squared,
      'disabled': props.disabled,
    }))

    const attrs = computed(() => ({
      'aria-disabled': nonStandardTag ? String(props.disabled) : null,
      'aria-pressed': isToggle ? String(props.pressed) : null,
      'autocomplete': isToggle ? 'off' : null,
      'disabled': isButton ? props.disabled : null,
      'href': props.href,
      'rel': isLink ? props.rel : null,
      'role': nonStandardTag || isLink ? 'button' : null,
      'target': isLink ? props.target : null,
      'type': isButton ? props.type : null,
      'to': !isButton ? props.to : null,
      'append': isLink ? props.append : null,
      'activeClass': isBLink ? props.activeClass : null,
      'event': isBLink ? props.event : null,
      'exact': isBLink ? props.exact : null,
      'exactActiveClass': isBLink ? props.exactActiveClass : null,
      'replace': isBLink ? props.replace : null,
      'routerComponentName': isBLink ? props.routerComponentName : null,
      'routerTag': isBLink ? props.routerTag : null,
    }))

    const computedTag = computed<string>(() => {
      if (isBLink) return 'b-link'
      return props.href ? 'a' : props.tag
    })

    const clicked = (e: MouseEvent): void => {
      if (props.disabled) {
        e.preventDefault()
        e.stopPropagation()
        return
      }
      emit('click', e)
      if (isToggle) {
        emit('update:pressed', !props.pressed)
      }
    }

    return {
      classes,
      attrs,
      computedTag,
      clicked,
    }
  },
})
</script>
