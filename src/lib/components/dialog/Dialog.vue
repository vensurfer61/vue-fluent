<template>
  <transition :name="animation">
    <div
      v-if="isActive"
      :class="size"
      class="dialog modal is-active">
      <div 
        class="modal-background" 
        @click="cancel('outside')"/>
      <div class="modal-card animation-content">
        <header 
          v-if="title" 
          class="modal-card-head">
          <p class="modal-card-title">{{ title }}</p>
        </header>

        <section
          :class="{ 'is-titleless': !title, 'is-flex': hasIcon }"
          class="modal-card-body">
          <div class="media">
            <div 
              v-if="hasIcon" 
              class="media-left">
              <b-icon
                :icon="icon ? icon : iconByType"
                :pack="iconPack"
                :type="type"
                :both="!icon"
                size="is-large"/>
            </div>
            <div class="media-content">
              <p v-html="message"/>

              <div 
                v-if="hasInput" 
                class="field">
                <div class="control">
                  <input
                    ref="input"
                    v-model="prompt"
                    :class="{ 'is-danger': validationMessage }"
                    v-bind="inputAttrs"
                    class="input"
                    @keyup.enter="confirm">
                </div>
                <p class="help is-danger">{{ validationMessage }}</p>
              </div>
            </div>
          </div>
        </section>

        <footer class="modal-card-foot">
          <button
            v-if="showCancel"
            ref="cancelButton"
            class="button"
            @click="cancel('button')">
            {{ cancelText }}
          </button>
          <button
            ref="confirmButton"
            :class="type"
            class="button"
            @click="confirm">
            {{ confirmText }}
          </button>
        </footer>
      </div>
    </div>
  </transition>
</template>

<script>
import Icon from '../icon'
import { Modal } from '../modal'
import config from '../../utils/config'
import { removeElement } from '../../utils/helpers'

export default {
  name: 'BDialog',
  components: {
    [Icon.name]: Icon,
  },
  extends: Modal,
  props: {
    title: String,
    message: String,
    icon: String,
    iconPack: String,
    hasIcon: Boolean,
    type: {
      type: String,
      default: 'is-primary',
    },
    size: String,
    confirmText: {
      type: String,
      default: () =>
        config.defaultDialogConfirmText
          ? config.defaultDialogConfirmText
          : 'OK',
    },
    cancelText: {
      type: String,
      default: () =>
        config.defaultDialogCancelText
          ? config.defaultDialogCancelText
          : 'Cancel',
    },
    hasInput: Boolean, // Used internally to know if it's prompt
    inputAttrs: {
      type: Object,
      default: () => ({}),
    },
    onConfirm: {
      type: Function,
      default: () => {},
    },
    focusOn: {
      type: String,
      default: 'confirm',
    },
  },
  data() {
    const prompt = this.hasInput ? this.inputAttrs.value || '' : ''

    return {
      prompt,
      isActive: false,
      validationMessage: '',
    }
  },
  computed: {
    /**
     * Icon name (MDI) based on the type.
     */
    iconByType() {
      switch (this.type) {
        case 'is-info':
          return 'information'
        case 'is-success':
          return 'check-circle'
        case 'is-warning':
          return 'alert'
        case 'is-danger':
          return 'alert-circle'
        default:
          return null
      }
    },
    showCancel() {
      return this.cancelOptions.indexOf('button') >= 0
    },
  },
  beforeMount() {
    // Insert the Dialog component in body tag
    document.body.appendChild(this.$el)
  },
  mounted() {
    this.isActive = true

    if (typeof this.inputAttrs.required === 'undefined') {
      this.$set(this.inputAttrs, 'required', true)
    }

    this.$nextTick(() => {
      // Handle which element receives focus
      if (this.hasInput) {
        this.$refs.input.focus()
      } else if (this.focusOn === 'cancel' && this.showCancel) {
        this.$refs.cancelButton.focus()
      } else {
        this.$refs.confirmButton.focus()
      }
    })
  },
  methods: {
    /**
     * If it's a prompt Dialog, validate the input.
     * Call the onConfirm prop (function) and close the Dialog.
     */
    confirm() {
      if (this.$refs.input !== undefined) {
        if (!this.$refs.input.checkValidity()) {
          this.validationMessage = this.$refs.input.validationMessage
          this.$nextTick(() => this.$refs.input.select())
          return
        }
      }

      this.onConfirm(this.prompt)
      this.close()
    },

    /**
     * Close the Dialog.
     */
    close() {
      this.isActive = false
      // Timeout for the animation complete before destroying
      setTimeout(() => {
        this.$destroy()
        removeElement(this.$el)
      }, 150)
    },
  },
}
</script>
