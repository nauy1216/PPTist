<template>
  <div class="text-element-operate">
    <template v-if="!elementInfo.lock && (isActiveGroupElement || !isMultiSelect)">
      <ResizeHandler
        class="operate-resize-handler" 
        v-for="point in resizeHandlers"
        :key="point.direction"
        :type="point.direction"
        :style="point.style"
        @mousedown.stop="$event => dragLineElement($event, elementInfo, point.handler)"
      />
    </template>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, PropType } from 'vue'
import { useStore } from 'vuex'
import { State } from '@/store'

import { PPTLineElement } from '@/types/slides'
import { OperateLineHandler, OperateLineHandlers } from '@/types/edit'

import ResizeHandler from './ResizeHandler.vue'

export default defineComponent({
  name: 'text-element-operate',
  components: {
    ResizeHandler,
  },
  props: {
    elementInfo: {
      type: Object as PropType<PPTLineElement>,
      required: true,
    },
    isActiveGroupElement: {
      type: Boolean,
      required: true,
    },
    isMultiSelect: {
      type: Boolean,
      required: true,
    },
    dragLineElement: {
      type: Function as PropType<(e: MouseEvent, element: PPTLineElement, command: OperateLineHandler) => void>,
      required: true,
    },
  },
  setup(props) {
    const store = useStore<State>()
    const canvasScale = computed(() => store.state.canvasScale)

    const resizeHandlers = computed(() => {
      return [
        {
          handler: OperateLineHandlers.START,
          style: {
            left: props.elementInfo.start[0] * canvasScale.value + 'px',
            top: props.elementInfo.start[1] * canvasScale.value + 'px',
          }
        },
        {
          handler: OperateLineHandlers.END,
          style: {
            left: props.elementInfo.end[0] * canvasScale.value + 'px',
            top: props.elementInfo.end[1] * canvasScale.value + 'px',
          }
        },
      ]
    })

    return {
      resizeHandlers,
    }
  },
})
</script>