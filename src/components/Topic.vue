<template>
  <div>
    <a href="#"
       class="topic-header"
       @click.prevent="handleTopicHeaderClick"
       v-if="shouldShowHeader"
    >
      <span v-show="status === 'waiting'" class="loading">
        <i class="fa fa-spinner fa-lg spin"></i>
      </span>
      <i :class="['fa', 'fa-' + icon, 'topic-header-icon']"
         aria-hidden="true"
      />
      {{ topic.label }}
    </a>

    <!-- success -->
    <transition name="topic-body">
      <div class="topic-body" v-if="shouldShowBody">
        <topic-component-group :topic-components="topic.components" />
      </div>
    </transition>

    <!-- error -->
    <div class="topic-body" v-show="shouldShowError">
      Could not locate records for that address.
    </div>
  </div>
</template>

<script>
  // import { mapMutations } from 'vuex';

  import TopicComponentGroup from './TopicComponentGroup.vue';

  export default {
    props: ['topicKey'],
    components: {
      TopicComponentGroup
    },
    computed: {
      // returns the full config object for the topic
      topic() {
        const topicKey = this.$props.topicKey;
        const topicsFiltered = this.$config.topics.filter((topic) => {
          return topic.key === this.$props.topicKey;
        });
        if (topicsFiltered.length !== 1) {
          throw `Could not get single config object for topic '${topicKey}'.`;
        }
        const config = topicsFiltered[0];
        return config;
      },
      icon() {
        return this.topic.icon;
      },
      isActive() {
        const key = this.topic.key;
        const activeTopic = this.$store.state.activeTopic;
        return activeTopic === key;
      },
      shouldShowHeader() {
        return this.$config.topics.length > 1;
      },
      dataSources() {
        return this.topic.dataSources || [];
      },
      hasData() {
        if (this.dataSources.includes('condoList') && this.condoListExists) {
          return true;
        } else {
          return this.dataSources.every(dataSource => {
            const targetsFn = this.$config.dataSources[dataSource].targets
            if (targetsFn) {
              const targetsMap = this.$store.state.sources[dataSource].targets;
              const targets = Object.values(targetsMap);
              return targets.every(target => target.status !== 'waiting');
            } else {
              return this.$store.state.sources[dataSource].data;
            }
          });
        }
      },
      shouldShowBody() {
        const succeeded = this.status === 'success';
        const hasData = this.hasData;
        const condoListExists = this.condoListExists;
        const should = succeeded && hasData && this.isActive;
        return should;
      },
      condoListExists() {
        const condoList = this.$store.state.sources.condoList.data;
        // let status;
        if (!condoList) {
          return false;
        } else {
          if (condoList.features.length === 1) {
            return false;
          } else {
            return true;
          }
        }
      },
      shouldShowError() {
        // console.log('Topic.vue shouldShowError this.hasData', this.topicKey, this.hasData, this.condoListExists);
        return (
          // topic must be active and
          this.isActive && (
            // there either has to be an error or
            this.status === 'error' ||
            // we got the response and it's empty
            (this.status !== 'waiting' && !this.hasData)
          )
        );
      },
      // REVIEW this is getting cached and not updating when the deps update
      status: {
        cache: false,
        get() {
          // get the status of each source
          const dataSources = this.topic.dataSources || [];

          // if no sources, return success
          if (dataSources.length === 0) {
            return 'success';
          }

          let topicStatus;

          const sourceStatuses = dataSources.map(dataSource => {
            // this is what should be observed. when it changes,
            // it's not causing this to re-evaluate.
            return this.$store.state.sources[dataSource].status;
          });

          // if any sources are still waiting, return waiting
          if (sourceStatuses.some(x => x === 'waiting')) {
            topicStatus = 'waiting';
          }

          // if any sources have errors, return error
          else if (sourceStatuses.some(x => x === 'error')) {
            topicStatus = 'error';
          }

          else {
            topicStatus = 'success';
          }

          return topicStatus;
        }
      },
    },
    methods: {
      configForBasemap(key) {
        return this.$config.map.basemaps[key];
      },

      handleTopicHeaderClick(e) {
        const topic = this.$props.topicKey;

        let nextTopic;

        if (topic === this.$store.state.activeTopic) {
          nextTopic = null;
        } else {
          nextTopic = topic;
        }

        // notify controller (which will handle routing)
        this.$controller.handleTopicHeaderClick(nextTopic);
      },
    }
  };
</script>

<style scoped>
  /*REVIEW these aren't prefixed `mb-`because they're scoped, but it feels
  inconsistent?*/
  .topic-header {
    background: #f5f5f5;
    border: 1px solid #ddd;
    display: block;
    font-size: 18px;
    font-weight: normal;
    height: 70px;
    line-height: 45px;
    padding: 10px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
    margin-bottom: 8px;
  }

  .topic-header:hover {
    background: #fff;
    color: inherit;
  }

  .topic-header-icon {
    padding-left: 10px;
    padding-right: 10px;
  }

  .topic-body {
    padding: 5px;
    margin-bottom: 10px;
  }

  .loading {
    float: right;
  }

  .topic-body-enter-active, .topic-body-leave-active {
    transition: opacity 0.18s;
  }

  .topic-body-enter, .topic-body-leave-to {
    opacity: 0;
  }
</style>
