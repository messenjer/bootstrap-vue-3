<template>
  <div
    :id="id"
    ref="element"
    class="popover b-popover"
    :class="classes"
    role="tooltip"
    tabindex="-1"
  >
    <div ref="titleRef">
      <slot name="title">
        {{ title }}
      </slot>
    </div>
    <div ref="contentRef">
      <slot>
        {{ content }}
      </slot>
    </div>
  </div>
</template>

<script lang="ts">
import {
  ComponentPublicInstance,
  computed,
  defineComponent,
  nextTick,
  onBeforeUnmount,
  onMounted,
  PropType,
  ref,
  watch,
} from 'vue'
import Popover from 'bootstrap/js/dist/popover'
import {useEventListener} from '../composables'
import type {BPopoverDelayObject} from '../types/components'
import type {ColorVariant} from '../types'

export default defineComponent({
  props: {
    container: {
      type: [String, Object] as PropType<
        string | ComponentPublicInstance<HTMLElement> | HTMLElement
      >,
      default: 'body',
    },
    content: {type: String},
    id: {type: String},
    noninteractive: {type: Boolean, default: false},
    placement: {type: String as PropType<Popover.Options['placement']>, default: 'right'},
    target: {
      type: [String, Object] as PropType<
        string | ComponentPublicInstance<HTMLElement> | HTMLElement | undefined
      >,
      default: undefined,
    },
    title: {type: String},
    delay: {type: [Number, Object] as PropType<number | BPopoverDelayObject>, default: 0},
    triggers: {type: String as PropType<Popover.Options['trigger']>, default: 'click'},
    show: {type: Boolean, default: false},
    variant: {type: String as PropType<ColorVariant>, default: undefined},
    html: {type: Boolean, default: true},
    sanitize: {type: Boolean, default: false},
    offset: {type: String as PropType<Popover.Options['offset']>, default: '0'},
  },
  emits: ['show', 'shown', 'hide', 'hidden', 'inserted'],
  setup(props, {emit, slots}) {
    const element = ref<HTMLElement>()
    const target = ref<HTMLElement | undefined>()
    const instance = ref<Popover>()
    const titleRef = ref<HTMLElement>()
    const contentRef = ref<HTMLElement>()
    const classes = computed(() => ({
      [`b-popover-${props.variant}`]: props.variant,
    }))

    const cleanElementProp = (
      target: string | ComponentPublicInstance<HTMLElement> | HTMLElement | undefined
    ): HTMLElement | string | undefined => {
      if (typeof target === 'string') {
        return target
      } else if (target instanceof HTMLElement) return target
      else if (typeof target !== 'undefined')
        return (target as ComponentPublicInstance<HTMLElement>).$el as HTMLElement
      return undefined
    }

    const getElement = (element: HTMLElement | string | undefined): HTMLElement | undefined => {
      if (!element) return undefined
      if (typeof element === 'string') {
        const idElement = document.getElementById(element)
        return idElement ? idElement : undefined
      }
      return element
    }

    const generatePopoverInstance = (
      targetValue: string | ComponentPublicInstance<HTMLElement> | HTMLElement | undefined
    ) => {
      target.value = getElement(cleanElementProp(targetValue))

      if (!target.value) return

      instance.value = new Popover(target.value, {
        container: cleanElementProp(props.container),
        trigger: props.triggers,
        placement: props.placement,
        title: props.title || slots.title ? titleRef.value : '',
        content: contentRef.value,
        html: props.html,
        delay: props.delay,
        sanitize: props.sanitize,
        offset: props.offset,
      })
    }

    onMounted(() => {
      nextTick(() => {
        generatePopoverInstance(props.target)
      })

      element.value?.parentNode?.removeChild(element.value)

      if (props.show) {
        instance.value?.show()
      }
    })

    onBeforeUnmount(() => {
      instance.value?.dispose()
    })

    watch(
      () => props.target,
      (newValue) => {
        instance.value?.dispose()
        generatePopoverInstance(newValue)
      }
    )

    watch(
      () => props.show,
      (show, oldVal) => {
        if (show !== oldVal) {
          if (show) {
            instance.value?.show()
          } else {
            instance.value?.hide()
          }
        }
      }
    )

    useEventListener(target, 'show.bs.popover', () => emit('show'))
    useEventListener(target, 'shown.bs.popover', () => emit('shown'))
    useEventListener(target, 'hide.bs.popover', () => emit('hide'))
    useEventListener(target, 'hidden.bs.popover', () => emit('hidden'))
    useEventListener(target, 'inserted.bs.popover', () => emit('inserted'))

    return {
      element,
      titleRef,
      contentRef,
      classes,
    }
  },
})
/*
// import type {BPopoverEmits, BPopoverProps} from '../types/components'
import type {BPopoverDelayObject} from '../types/components'
import Popover from 'bootstrap/js/dist/popover'
import {
  ComponentPublicInstance,
  computed,
  defineComponent,
  nextTick,
  onBeforeUnmount,
  onMounted,
  ref,
  useSlots,
  watch,
} from 'vue'
import {useEventListener} from '../composables'
import type {ColorVariant} from '../types'

interface BPopoverProps {
  id?: string
  title?: string
  content?: string
  container?: string | ComponentPublicInstance<HTMLElement> | HTMLElement
  noninteractive?: boolean
  placement?: Popover.Options['placement']
  target?: string | ComponentPublicInstance<HTMLElement> | HTMLElement
  delay?: number | BPopoverDelayObject
  triggers?: Popover.Options['trigger']
  show?: boolean
  variant?: ColorVariant
  html?: boolean
  sanitize?: boolean
  offset?: Popover.Options['offset']
}

const props = withDefaults(defineProps<BPopoverProps>(), {
  container: 'body',
  noninteractive: false,
  placement: 'right',
  target: undefined,
  delay: 0,
  triggers: 'click',
  show: false,
  variant: undefined,
  html: true,
  sanitize: false,
  offset: '0',
})

interface BPopoverEmits {
  (e: 'show'): void
  (e: 'shown'): void
  (e: 'hide'): void
  (e: 'hidden'): void
  (e: 'inserted'): void
}

const emit = defineEmits<BPopoverEmits>()

const slots = useSlots()

const element = ref<HTMLElement>()
const target = ref<HTMLElement | undefined>()
const instance = ref<Popover>()
const titleRef = ref<HTMLElement>()
const contentRef = ref<HTMLElement>()
const classes = computed(() => ({
  [`b-popover-${props.variant}`]: props.variant,
}))

const cleanElementProp = (
  target: string | ComponentPublicInstance<HTMLElement> | HTMLElement | undefined
): HTMLElement | string | undefined => {
  if (typeof target === 'string') {
    return target
  } else if (target instanceof HTMLElement) return target
  else if (typeof target !== 'undefined')
    return (target as ComponentPublicInstance<HTMLElement>).$el as HTMLElement
  return undefined
}

const getElement = (element: HTMLElement | string | undefined): HTMLElement | undefined => {
  if (!element) return undefined
  if (typeof element === 'string') {
    const idElement = document.getElementById(element)
    return idElement ? idElement : undefined
  }
  return element
}

const generatePopoverInstance = (
  targetValue: string | ComponentPublicInstance<HTMLElement> | HTMLElement | undefined
) => {
  target.value = getElement(cleanElementProp(targetValue))

  if (!target.value) return

  instance.value = new Popover(target.value, {
    container: cleanElementProp(props.container),
    trigger: props.triggers,
    placement: props.placement,
    title: props.title || slots.title ? titleRef.value : '',
    content: contentRef.value,
    html: props.html,
    delay: props.delay,
    sanitize: props.sanitize,
    offset: props.offset,
  })
}

onMounted(() => {
  nextTick(() => {
    generatePopoverInstance(props.target)
  })

  element.value?.parentNode?.removeChild(element.value)

  if (props.show) {
    instance.value?.show()
  }
})

onBeforeUnmount(() => {
  instance.value?.dispose()
})

watch(
  () => props.target,
  (newValue) => {
    instance.value?.dispose()
    generatePopoverInstance(newValue)
  }
)

watch(
  () => props.show,
  (show, oldVal) => {
    if (show !== oldVal) {
      if (show) {
        instance.value?.show()
      } else {
        instance.value?.hide()
      }
    }
  }
)

useEventListener(target, 'show.bs.popover', () => emit('show'))
useEventListener(target, 'shown.bs.popover', () => emit('shown'))
useEventListener(target, 'hide.bs.popover', () => emit('hide'))
useEventListener(target, 'hidden.bs.popover', () => emit('hidden'))
useEventListener(target, 'inserted.bs.popover', () => emit('inserted'))
*/
</script>
