<template>
  <pf-config-view
    :form-store-name="formStoreName"
    :isLoading="isLoading"
    :disabled="isLoading"
    :isNew="isNew"
    :isClone="isClone"
    :view="view"
    @close="close"
    @create="create"
    @save="save"
    @remove="remove"
  >
    <template v-slot:header>
      <b-button-close @click="close" v-b-tooltip.hover.left.d300 :title="$t('Close [ESC]')"><icon name="times"></icon></b-button-close>
      <h4 class="mb-0">
        <span v-if="!isNew && !isClone" v-html="$t('Routed Network {id}', { id: $strong(id) })"></span>
        <span v-else-if="isClone" v-html="$t('Clone Routed Network {id}', { id: $strong(id) })"></span>
        <span v-else>{{ $t('New Routed Network') }}</span>
      </h4>
    </template>
    <template v-slot:footer>
      <b-card-footer>
        <pf-button-save :disabled="isDisabled" :isLoading="isLoading" class="mr-1">
          <template v-if="isNew">{{ $t('Create') }}</template>
          <template v-else-if="isClone">{{ $t('Clone') }}</template>
          <template v-else-if="actionKey">{{ $t('Save & Close') }}</template>
          <template v-else>{{ $t('Save') }}</template>
        </pf-button-save>
        <b-button :disabled="isLoading" class="mr-1" variant="outline-secondary" @click="init()">{{ $t('Reset') }}</b-button>
        <template v-if="!isNew && !isClone">
          <b-button :disabled="isLoading" class="mr-1" variant="outline-primary" @click="clone()">{{ $t('Clone') }}</b-button>
          <pf-button-service service="pfdhcp" class="mr-1" restart start stop></pf-button-service>
          <pf-button-service service="pfdns" class="mr-1" restart start stop></pf-button-service>
          <pf-button-service service="keepalived" class="mr-1" restart start stop></pf-button-service>
          <pf-button-delete :disabled="isLoading" class="mr-1" :confirm="$t('Delete Routed Network?')" @on-delete="remove()"/>
        </template>
      </b-card-footer>
    </template>
  </pf-config-view>
</template>

<script>
import pfConfigView from '@/components/pfConfigView'
import pfButtonSave from '@/components/pfButtonSave'
import pfButtonDelete from '@/components/pfButtonDelete'
import pfButtonService from '@/components/pfButtonService'
import {
  defaultsFromMeta as defaults
} from '../_config/'
import {
  view,
  validators
} from '../_config/routedNetwork'

export default {
  name: 'routed-network-view',
  components: {
    pfConfigView,
    pfButtonSave,
    pfButtonDelete,
    pfButtonService
  },
  props: {
    formStoreName: { // from router
      type: String,
      default: null,
      required: true
    },
    isNew: { // from router
      type: Boolean,
      default: false
    },
    isClone: { // from router
      type: Boolean,
      default: false
    },
    id: { // from router
      type: String,
      default: null
    }
  },
  computed: {
    meta () {
      return this.$store.getters[`${this.formStoreName}/$meta`]
    },
    form () {
      return this.$store.getters[`${this.formStoreName}/$form`]
    },
    view () {
      return view(this.form, this.meta) // ../_config/routedNetwork
    },
    invalidForm () {
      return this.$store.getters[`${this.formStoreName}/$formInvalid`]
    },
    isLoading () {
      return this.$store.getters['$_routed_networks/isLoading']
    },
    isDisabled () {
      return this.invalidForm || this.isLoading
    },
    actionKey () {
      return this.$store.getters['events/actionKey']
    },
    escapeKey () {
      return this.$store.getters['events/escapeKey']
    }
  },
  methods: {
    init () {
      this.$store.dispatch('$_routed_networks/options', this.id).then(options => {
        const { meta = {} } = options
        const { isNew, isClone } = this
        this.$store.dispatch(`${this.formStoreName}/setMeta`, { ...meta, ...{ isNew, isClone } })
        if (this.id) {
          // existing
          this.$store.dispatch('$_routed_networks/getRoutedNetwork', this.id).then(form => {
            this.$store.dispatch(`${this.formStoreName}/setForm`, form)
          })
        } else {
          // new
          this.form = defaults(meta) // set defaults
        }
      })
      this.$store.dispatch(`${this.formStoreName}/setFormValidations`, validators)
    },
    close () {
      this.$router.push({ name: 'interfaces' })
    },
    clone () {
      this.$router.push({ name: 'cloneRoutedNetwork' })
    },
    create () {
      const actionKey = this.actionKey
      this.$store.dispatch('$_routed_networks/createRoutedNetwork', this.form).then(response => {
        if (actionKey) { // [CTRL] key pressed
          this.close()
        } else {
          this.$router.push({ name: 'routed_network', params: { id: this.form.id } })
        }
      })
    },
    save () {
      const actionKey = this.actionKey
      this.$store.dispatch('$_routed_networks/updateRoutedNetwork', this.form).then(response => {
        if (actionKey) { // [CTRL] key pressed
          this.close()
        }
      })
    },
    remove () {
      this.$store.dispatch('$_routed_networks/deleteRoutedNetwork', this.id).then(response => {
        this.close()
      })
    }
  },
  created () {
    this.init()
  },
  watch: {
    isClone: {
      handler: function (a, b) {
        this.init()
      }
    },
    escapeKey (pressed) {
      if (pressed) this.close()
    }
  }
}
</script>
