<template>
  <b-card no-body>
    <pf-config-list
      ref="pfConfigList"
      :config="config"
    >
      <template v-slot:pageHeader>
        <b-card-header>
          <h4 class="mb-0">{{ $t('Fast Profiles') }}</h4>
        </b-card-header>
      </template>
      <template v-slot:buttonAdd>
        <b-button variant="outline-primary" :to="{ name: 'newRadiusFast' }">{{ $t('New Fast Profile') }}</b-button>
        <pf-button-service service="radiusd-acct" class="ml-1" restart start stop :disabled="isLoading" @start="init" @restart="init"></pf-button-service>
        <pf-button-service service="radiusd-auth" class="ml-1" restart start stop :disabled="isLoading" @start="init" @restart="init"></pf-button-service>
      </template>
      <template v-slot:emptySearch="state">
        <pf-empty-table :isLoading="state.isLoading">{{ $t('No Fast profiles found') }}</pf-empty-table>
      </template>
      <template v-slot:cell(buttons)="item">
        <span class="float-right">
          <pf-button-delete size="sm" v-if="!item.not_deletable" variant="outline-danger" class="mr-1" :disabled="isLoading" :confirm="$t('Delete Profile?')" @on-delete="remove(item)" reverse/>
          <b-button size="sm" variant="outline-primary" class="mr-1" @click.stop.prevent="clone(item)">{{ $t('Clone') }}</b-button>
        </span>
      </template>
    </pf-config-list>
  </b-card>
</template>

<script>
import pfButtonDelete from '@/components/pfButtonDelete'
import pfButtonService from '@/components/pfButtonService'
import pfConfigList from '@/components/pfConfigList'
import pfEmptyTable from '@/components/pfEmptyTable'
import {
  config
} from '../_config/radius/fast'

export default {
  name: 'radius-fast-list',
  components: {
    pfButtonDelete,
    pfButtonService,
    pfConfigList,
    pfEmptyTable
  },
  data () {
    return {
      config: config(this)
    }
  },
  methods: {
    clone (item) {
      this.$router.push({ name: 'cloneRadiusFast', params: { id: item.id } })
    },
    remove (item) {
      this.$store.dispatch('$_radius_fast/deleteRadiusFast', item.id).then(response => {
        const { $refs: { pfConfigList: { refreshList = () => {} } = {} } = {} } = this
        refreshList() // soft reload
      })
    }
  }
}
</script>
