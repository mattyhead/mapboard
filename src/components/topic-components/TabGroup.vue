<template>
  <div>
    <ul class="tabs" data-tabs>
      <li class="tabs-title"
          v-for="item in items"
          v-bind:class="{'is-active': itemIsActive(item)}"
          :key="keyForItem(item)"
      >
        <a :href="'#parcel-' + keyForItem(item)"
           @click.prevent="clickedItem(item)"
        >
        <!-- @click.prevent="activeItem = keyForItem(item)" -->
          {{ titleForItem(item) }}
        </a>
      </li>
    </ul>

    <div class="tabs-content">
      <div class="tabs-panel"
           v-for="item in items"
           v-bind:class="{'is-active': itemIsActive(item)}"
           v-bind:id="'parcel-' + keyForItem(item)"
      >
        <topic-component-group :topic-components="comps" :item="item"/>
      </div>
    </div>
  </div>
</template>

<script>
  import TopicComponent from './TopicComponent.vue';
  import TopicComponentGroup from '../TopicComponentGroup.vue';

  export default {
    mixins: [TopicComponent],
    components: {
      TopicComponentGroup
    },
    // some internal state for things local enough that they shouldn't be in
    // vuex if we can avoid it.
    data() {
      // computed props aren't accessible here, so evaluate slot separately
      const items = this.evaluateSlot(this.slots.items);
      return {
        activeItem: this.keyForItem(items[0]),
      };
    },
    // mounted() {
    //   // REVIEW globals. also is this still needed?
    //   // $(document).foundation();
    // },
    // props: [],
    computed: {
      items() {
        const items = this.evaluateSlot(this.slots.items);

        // sort
        const sortFn = this.options.sort;
        let itemsSorted = items;
        if (sortFn) {
          itemsSorted = sortFn(items);
        }

        return itemsSorted;
      },
      comps() {
        return this.options.components;
      }
    },
    watch: {
      // when items change, update the activeItem
      items(items) {
        const nextFirstItem = items[0];
        const nextActiveKey = this.keyForItem(nextFirstItem);
        this.activeItem = nextActiveKey;
      }
    },
    methods: {
      clickedItem(item) {
        this.$data.activeItem = this.keyForItem(item)
        // console.log('clickedItem is firing');
        this.$store.commit('setActiveDorParcel', this.$data.activeItem);
      },
      keyForItem(item) {
        try {
          return this.options.getKey(item);
        } catch (e) {
          return null;
        }
      },
      titleForItem(item) {
        try {
          return this.options.getTitle(item);
        } catch (e) {
          return null;
        }
      },
      itemIsActive(item) {
        const isActive = (this.activeItem === this.keyForItem(item));
        return isActive;
      }
    }
  };
</script>

<style scoped>
  .tabs-panel {
    padding: 20px;
    padding-bottom: 0px;
  }
</style>
