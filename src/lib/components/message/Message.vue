<template>
  <transition name="fade">
    <article
      v-if="isActive"
      :class="[type, size]"
      class="message">
      <header 
        v-if="title" 
        class="message-header">
        <p>{{ title }}</p>
        <button
          v-if="closable"
          type="button"
          class="delete"
          @click="close"
        />
      </header>
      <section class="message-body">
        <div class="media">
          <div 
            v-if="icon && hasIcon" 
            class="media-left">
            <b-icon
              :icon="icon"
              :class="type"
              :size="newIconSize"
              both/>
          </div>
          <div class="media-content">
            <slot/>
          </div>
        </div>
      </section>
    </article>
  </transition>
</template>

<script>
import MessageMixin from '../../utils/MessageMixin.js'

export default {
  name: 'BMessage',
  mixins: [MessageMixin],
  data() {
    return {
      newIconSize: this.iconSize || this.size || 'is-large',
    }
  },
}
</script>
