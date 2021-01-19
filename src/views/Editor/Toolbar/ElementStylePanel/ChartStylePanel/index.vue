<template>
  <div class="chart-style-panel">
    <Button class="full-width-btn" @click="chartDataEditorVisible = true">
      <IconEdit class="btn-icon" /> 编辑图表数据
    </Button>

    <Divider />

    <template v-if="handleElement.chartType === 'line'">
      <div class="row">
        <Checkbox 
          @change="e => updateOptions({ showArea: e.target.checked })" :checked="showArea" 
          style="flex: 1;"
        >面积图样式</Checkbox>
        <Checkbox 
          @change="e => updateOptions({ showLine: !e.target.checked })" :checked="!showLine" 
          style="flex: 1;"
        >散点图样式</Checkbox>
      </div>
      <div class="row">
        <Checkbox 
          @change="e => updateOptions({ lineSmooth: e.target.checked })" 
          :checked="lineSmooth"
        >使用平滑曲线</Checkbox>
      </div>
    </template>
    <div class="row" v-if="handleElement.chartType === 'bar'">
      <Checkbox 
        @change="e => updateOptions({ horizontalBars: e.target.checked })" 
        :checked="horizontalBars"
      >条形图样式</Checkbox>
    </div>
    <div class="row" v-if="handleElement.chartType === 'pie'">
      <Checkbox 
        @change="e => updateOptions({ donut: e.target.checked })" 
        :checked="donut"
      >环形图样式</Checkbox>
    </div>

    <Divider />

    <div class="row">
      <div style="flex: 2;">背景填充：</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker
            :modelValue="fill"
            @update:modelValue="value => updateFill(value)"
          />
        </template>
        <ColorButton :color="fill" style="flex: 3;" />
      </Popover>
    </div>
    <div class="row">
      <div style="flex: 2;">主题配色：</div>
      <Popover trigger="click" v-model:visible="themePoolVisible">
        <template #content>
          <div class="theme-pool">
            <div 
              class="theme-item" 
              v-for="(theme, index) in CHART_THEME_COLORS" 
              :key="index"
              @click="updateTheme(theme)"
            >
              <div 
                class="color-block" 
                v-for="(color, index) in theme" 
                :key="index"
                :style="{ backgroundColor: color }"
              ></div>
            </div>
          </div>
        </template>
        <Button class="theme-color-btn" style="flex: 3;">
          <div class="theme-color-content">
            <div 
              class="color-block" 
              v-for="(color, index) in themeColors" 
              :key="index"
              :style="{ backgroundColor: color }"
            ></div>
          </div>
          <IconPlatte class="theme-color-btn-icon" />
        </Button>
      </Popover>
    </div>
    <div class="row">
      <div style="flex: 2;">网格颜色：</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker
            :modelValue="gridColor"
            @update:modelValue="value => updateGridColor(value)"
          />
        </template>
        <ColorButton :color="gridColor" style="flex: 3;" />
      </Popover>
    </div>

    <Divider />
    <ElementOutline />

    <Modal
      v-model:visible="chartDataEditorVisible" 
      :footer="null" 
      centered
      :closable="false"
      :width="648"
      destroyOnClose
    >
      <ChartDataEditor 
        :data="handleElement.data"
        @close="chartDataEditorVisible = false"
        @save="value => updateData(value)"
      />
    </Modal>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, Ref, ref, watch } from 'vue'
import { IBarChartOptions, ILineChartOptions, IPieChartOptions } from 'chartist'
import { useStore } from 'vuex'
import { MutationTypes, State } from '@/store'
import { ChartData, PPTChartElement } from '@/types/slides'
import { CHART_THEME_COLORS } from '@/configs/chartTheme'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'

import ElementOutline from '../../common/ElementOutline.vue'
import ColorButton from '../../common/ColorButton.vue'
import ChartDataEditor from './ChartDataEditor.vue'

export default defineComponent({
  name: 'chart-style-panel',
  components: {
    ElementOutline,
    ChartDataEditor,
    ColorButton,
  },
  setup() {
    const store = useStore<State>()
    const handleElement: Ref<PPTChartElement> = computed(() => store.getters.handleElement)

    const chartDataEditorVisible = ref(false)
    const themePoolVisible = ref(false)

    const { addHistorySnapshot } = useHistorySnapshot()

    const fill = ref<string>()

    const themeColors = ref<string[]>([])
    const gridColor = ref('')

    const lineSmooth = ref<boolean | Function>(true)
    const showLine = ref(true)
    const showArea = ref(false)
    const horizontalBars = ref(false)
    const donut = ref(false)

    watch(handleElement, () => {
      if(!handleElement.value) return
      fill.value = handleElement.value.fill || '#000'

      if(handleElement.value.options) {
        const {
          lineSmooth: _lineSmooth,
          showLine: _showLine,
          showArea: _showArea,
          horizontalBars: _horizontalBars,
          donut: _donut,
        } = handleElement.value.options

        if(_lineSmooth !== undefined) lineSmooth.value = _lineSmooth
        if(_showLine !== undefined) showLine.value = _showLine
        if(_showArea !== undefined) showArea.value = _showArea
        if(_horizontalBars !== undefined) horizontalBars.value = _horizontalBars
        if(_donut !== undefined) donut.value = _donut
      }

      themeColors.value = handleElement.value.themeColors || CHART_THEME_COLORS[0]
      gridColor.value = handleElement.value.gridColor || 'rgba(0, 0, 0, 0.4)'
    }, { deep: true, immediate: true })

    const updateData = (data: ChartData) => {
      chartDataEditorVisible.value = false
      const props = { data }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    const updateFill = (value: string) => {
      const props = { fill: value }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    const updateOptions = (optionProps: ILineChartOptions & IBarChartOptions & IPieChartOptions) => {
      const options = handleElement.value.options || {}
      const newOptions = { ...options, ...optionProps }
      const props = { options: newOptions }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    const updateTheme = (themeColors: string[]) => {
      themePoolVisible.value = false
      const props = { themeColors }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    const updateGridColor = (gridColor: string) => {
      const props = { gridColor }
      store.commit(MutationTypes.UPDATE_ELEMENT, { id: handleElement.value.id, props })
      addHistorySnapshot()
    }

    return {
      chartDataEditorVisible,
      themePoolVisible,
      handleElement,
      updateData,
      fill,
      updateFill,
      lineSmooth,
      showLine,
      showArea,
      horizontalBars,
      donut,
      updateOptions,
      themeColors,
      gridColor,
      CHART_THEME_COLORS,
      updateTheme,
      updateGridColor,
    }
  },
})
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.full-width-btn {
  width: 100%;
}
.btn-icon {
  margin-right: 3px;
}

.theme-item {
  border-radius: $borderRadius;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px 20px;
  transition: background-color .1s;

  & + .theme-item {
    margin-top: 8px;
  }

  &:hover {
    background-color: #e1e1e1;
    cursor: pointer;
  }
}
.theme-color-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 !important;
}
.theme-color-content {
  height: 20px;
  margin-left: 8px;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}
.color-block {
  width: 18px;
  height: 18px;
  border-radius: 50%;

  & + .color-block {
    margin-left: -3px;
  }
}
.theme-color-btn-icon {
  width: 30px;
  font-size: 12px;
  margin-top: 2px;
  color: #bfbfbf;
}
</style>