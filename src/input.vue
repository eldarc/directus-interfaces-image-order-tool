<template>
  <div class="image-order">
    <sortable-list
      v-if="items && items.length > 0"
      v-model="items"
      lock-axis="y">
      <sortable-item
        v-for="(item, index) in items"
        :index="index"
        :key="item.id">
        <div class="list-item-wrapper">
          <div class="list-item-descriptor">
            <img
              :src="item.fullUrl"
              class="list-item-image">
            <span class="list-item-title">
              {{ item.title.substring(0, 15) }}...
            </span>
          </div>
          <div class="list-item-position-selector">
            <span>
              <span>Horizontally</span>
              <v-select
                :value="item.xPosition"
                :options="xPositionOptions"
                @input="(newValue) => updateItemPosition('x', item, newValue)"
              />
            </span>
            <span>
              <span>Vertically</span>
              <v-select
                :value="item.yPosition"
                :options="yPositionOptions"
                @input="(newValue) => updateItemPosition('y', item, newValue)"
              />
            </span>
          </div>
        </div>
      </sortable-item>
    </sortable-list>
    <span v-else>
      {{ $t('interfaces-image-order-tool-no_images_selected') }}
    </span>
  </div>
</template>

<script>
import mixin from '@directus/extension-toolkit/mixins/interface'
import { ContainerMixin, ElementMixin, HandleDirective } from 'vue-slicksort'

const SortableList = {
  mixins: [ContainerMixin],
  render (createElement) {
    return createElement(
      'ul', {
        attrs: {
          'class': 'list'
        }
      },
      this.$slots.default
    )
  }
}

const SortableItem = {
  mixins: [ElementMixin],
  props: ['item'],
  directives: { handle: HandleDirective },
  render (createElement) {
    return createElement(
      'li', {
        attrs: {
          'class': 'list-item'
        }
      },
      [
        createElement('span', {
          attrs: {
            'class': 'list-handle'
          },
          directives: [
            {
              name: 'handle'
            }
          ]
        }),
        ...this.$slots.default
      ]

    )
  }
}

const imageItem = (id, title, fullUrl) => {
  return {
    id,
    title,
    fullUrl,
    xPosition: 'center',
    yPosition: 'center'
  }
}

const xPositionOptions = {
  center: 'Center',
  right: 'Right',
  left: 'Left',
  '0%': '0%',
  '5%': '5%',
  '10%': '10%',
  '15%': '15%',
  '20%': '20%',
  '25%': '25%',
  '30%': '30%',
  '35%': '35%',
  '40%': '40%',
  '45%': '45%',
  '50%': '50%',
  '55%': '55%',
  '60%': '60%',
  '65%': '65%',
  '70%': '70%',
  '75%': '75%',
  '80%': '80%',
  '85%': '85%',
  '90%': '90%',
  '95%': '95%',
  '100%': '100%'
}

const yPositionOptions = {
  center: 'Center',
  top: 'Top',
  bottom: 'Bottom',
  '0%': '0%',
  '5%': '5%',
  '10%': '10%',
  '15%': '15%',
  '20%': '20%',
  '25%': '25%',
  '30%': '30%',
  '35%': '35%',
  '40%': '40%',
  '45%': '45%',
  '50%': '50%',
  '55%': '55%',
  '60%': '60%',
  '65%': '65%',
  '70%': '70%',
  '75%': '75%',
  '80%': '80%',
  '85%': '85%',
  '90%': '90%',
  '95%': '95%',
  '100%': '100%'
}

export default {
  components: {
    SortableList,
    SortableItem
  },
  mixins: [mixin],
  data () {
    return {
      xPositionOptions,
      yPositionOptions,
      blockMirrorTrigger: false
    }
  },
  computed: {
    mirror () {
      const { mirroredField } = this.options

      return this.values[mirroredField]
    },
    items: {
      get () {
        return this.value || []
      },
      set (newValue) {
        this.updateValue(newValue)
      }
    }
  },
  watch: {
    mirror (newItemsRaw) {
      // Fix infinite loop updates.
      if (this.blockMirrorTrigger) {
        this.blockMirrorTrigger = false
        return
      }

      // Prepare items
      this.items = this.prepareItems(newItemsRaw)
    }
  },
  created () {
    this.items = this.prepareItems(this.mirror)
  },
  methods: {
    prepareItems (newItemsRaw) {
      if (newItemsRaw === null) {
        return []
      }

      // Remove deleted items from newItems.
      const newItems = newItemsRaw.filter(item => !item.hasOwnProperty('$delete'))

      // Remove unselected images from the list.
      const existingItems = this.items
        .filter(existingItem => newItems.find(newItem => newItem.directus_files_id.id === existingItem.id))

      // Select only items that are not logged in the existing items.
      const preparedItems = newItems
        .filter(newItem => !existingItems.find(existingItem => existingItem.id === newItem.directus_files_id.id))
        .map(newItem => imageItem(newItem.directus_files_id.id, newItem.directus_files_id.title, newItem.directus_files_id.data.full_url))

      return [
        ...existingItems,
        ...preparedItems
      ]
    },
    updateItemPosition (direction, itemRef, newValue) {
      // This method bypasses vuex warnings and limitations.
      const value = JSON.parse(JSON.stringify(this.value))
      const item = value.find(i => i.id === itemRef.id)
      item[`${direction}Position`] = newValue

      this.updateValue(value)
    },
    updateValue (value) {
      this.blockMirrorTrigger = true
      this.$nextTick(() => {
        this.blockMirrorTrigger = false
      })

      this.$emit(
        'input',
        value
      )
    }
  }
}
</script>

<style lang="scss" scoped>
.image-order {
  max-width: var(--width-medium);
}

.list {
  list-style-type: none;
  margin: 0 auto;
  padding: 0;
  overflow: auto;
  background-color: var(--lightest-gray);
  border: var(--input-border-width) solid var(--lighter-gray);
  border-radius: var(--border-radius);
  width: 100%;
}

.list-item {
  display: flex;
  align-items: center;
  width: 100%;
  padding: 20px;
  background-color: var(--white);
  border-bottom: var(--input-border-width) solid var(--lighter-gray);
  box-sizing: border-box;
  user-select: none;
  -moz-user-select: none;
  color: var(--gray);

  &:last-of-type {
    border-bottom: 0;
  }
}

.list-item-image {
  width: 100%;
  max-width: 80px;
  margin-right: 10px;
}

.list-item-title {
  margin-right: 10px;
}

.list-item-wrapper {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;

  @media (max-width: 580px) {
    flex-direction: column;
  }
}

.list-item-descriptor {
  display: flex;
  margin-right: auto;
  align-items: center;
  justify-content: center;
}

.list-item-position-selector {
  display: flex;
  margin-left: auto;
  text-align: center;
  justify-content: end;

  span {
    margin-right: 10px;

    &:last-of-type {
      margin-right: 0;
    }
  }

  @media (max-width: 580px) {
    margin-top: 15px;
    margin-left: 0;
    margin-right: auto;
  }
}
</style>

<style lang="scss">
.list-item .list-handle {
  display: block;
  width: 18px;
  min-width: 18px;
  height: 18px;
  background-image: url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="50" height="50" viewBox="0 0 50 50"><path d="M0 7.5v5h50v-5H0zm0 15v5h50v-5H0zm0 15v5h50v-5H0z" color="%23000"/></svg>');
  background-size: contain;
  background-repeat: no-repeat;
  opacity: .25;
  margin-right: 20px;
  cursor: row-resize;
}
</style>
