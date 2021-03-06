<template>
  <f7-card :no-border="config.noBorder" :no-shadow="config.noShadow" :outline="config.outline">
    <f7-card-header v-if="config.title">
      <div>{{config.title}}</div>
    </f7-card-header>
    <f7-card-content ref="cardContent" @click.native="performAction" class="label-card-content" :style="{ background: config.background }" :class="{ 'vertical-arrangement': config.vertical }">
      <trend :key="'trend' + config.item" v-if="showTrend" :width="trendWidth" class="trend" :data="trendData" :gradient="trendGradient" :stroke-width="trendStrokeWidth" auto-draw smooth />
      <f7-list>
        <f7-list-item :link="config.action ? true : false" no-chevron>
          <oh-icon slot="media" v-if="config.icon && config.icon.indexOf('oh:') === 0" :icon="config.icon.substring(3)" :height="config.iconSize || 32" :width="config.iconSize || 32" />
          <f7-icon slot="media" v-else-if="config.icon" :ios="config.icon" :md="config.icon" :aurora="config.icon" :size="config.iconSize || 32" :color="config.iconColor" />
          <div :class="config.class">
            <span :style="{ 'font-size': config.fontSize || '24px', 'font-weight': config.fontWeight || 'normal' }">
              {{config.label || context.store[config.item].displayState || context.store[config.item].state}}
            </span>
          </div>
        </f7-list-item>
      </f7-list>
      <!-- <f7-link class="label-link" v-if="config.action">{{context.store[config.item].displayState || context.store[config.item].state}}</f7-link> -->
      <!-- <h2>{{context.store[config.item].displayState || context.store[config.item].state}}</h2> -->
    </f7-card-content>
    <f7-card-footer v-if="config.footer">
      {{config.footer}}
    </f7-card-footer>
  </f7-card>
</template>

<style lang="stylus">
.label-card-content
  .item-content
    padding 1rem
  &.vertical-arrangement
    .item-content
      flex-direction column
      padding-left 0
      padding-right 0
  .item-inner
    padding 0 !important
    justify-content center !important
    text-align center
    --f7-list-item-media-margin 0
  .trend
    position absolute
    left 0
    top 0
    width 100%
    height 100%
    opacity 0.2
</style>

<script>
import mixin from '../widget-mixin'
import { actionGroup, actionProps, actionsMixin } from '../widget-actions'

export default {
  mixins: [mixin, actionsMixin],
  widget: {
    name: 'oh-label-card',
    label: 'Label Card',
    description: 'Display the state of an item in a card',
    props: {
      parameterGroups: [
        actionGroup(null, 'Action to perform when the element is clicked')
      ],
      parameters: [
        ...actionProps(),
        {
          name: 'title',
          label: 'Title',
          type: 'TEXT',
          description: 'Title of the card'
        },
        {
          name: 'item',
          label: 'Item',
          type: 'TEXT',
          context: 'item',
          description: 'Item to display'
        },
        {
          name: 'footer',
          label: 'Footer text',
          type: 'TEXT',
          description: 'Footer of the card'
        },
        {
          name: 'label',
          label: 'Label',
          type: 'TEXT',
          description: 'Display this text (or expression result) instead of the item\'s state'
        },
        {
          name: 'background',
          label: 'Background style',
          type: 'TEXT',
          description: 'Background style (in CSS "background" attribute format)'
        },
        {
          name: 'fontSize',
          label: 'Font Size',
          type: 'TEXT',
          description: 'Font size (e.g. "34px")'
        },
        {
          name: 'fontWeight',
          label: 'Font Size',
          type: 'TEXT',
          description: 'Font size (e.g. "normal" or "bold")'
        },
        {
          name: 'icon',
          label: 'Icon',
          type: 'TEXT',
          description: 'Use <code>oh:iconName</code> (<a class="external text-color-blue" target="_blank" href="https://www.openhab.org/docs/configuration/iconsets/classic/">openHAB icon</a>) or <code>f7:iconName</code> (<a class="external text-color-blue" target="_blank" href="https://framework7.io/icons/">Framework7 icon</a>)'
        },
        {
          name: 'iconColor',
          label: 'Icon Color',
          type: 'TEXT',
          description: 'Not applicable to openHAB icons'
        },
        {
          name: 'iconSize',
          label: 'Icon Size',
          type: 'NUMBER',
          description: 'Size of the icon in px'
        },
        {
          name: 'vertical',
          label: 'Vertical arrangement',
          type: 'BOOLEAN',
          description: 'Display label below icon'
        }
      ]
    }
  },
  data () {
    return {
      trendData: [],
      showTrend: false
    }
  },
  computed: {
    trendItem () {
      return this.config.trendItem
    },
    trendWidth () {
      return this.$refs.cardContent.$el.clientWidth
    },
    trendGradient () {
      return this.config.trendGradient || ['#2196f3', '#5ac8fa']
    },
    trendStrokeWidth () {
      return this.config.trendStrokeWidth || 3
    }
  },
  mounted () {
    this.buildTrend()
  },
  watch: {
    trendItem (item) {
      this.buildTrend()
    }
  },
  methods: {
    buildTrend () {
      this.trendData = []
      this.showTrend = false
      if (!this.trendItem) return []
      const sampling = this.config.trendSampling || 60
      return this.$oh.api.get('/rest/persistence/items/' + this.config.trendItem).then((resp) => {
        if (resp.data && resp.data.length) {
          let data = []
          for (let i = 0; i < resp.data.length; i += sampling) {
            data.push(parseFloat(resp.data[i].state))
          }
          this.trendData = data
          this.showTrend = true
        }
      })
    }
  }
}
</script>
