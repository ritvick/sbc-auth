<template>
  <v-btn
    large
    outlined
    color="bcgovblue"
    class="btn-name-request white--text"
    :class="{'btn-name-request-wide': isWide, 'btn-name-request-inverse': isInverse}"
    @click="goToNameRequest()"
  >
    <span class="btn-text">Request a Name</span>
  </v-btn>
</template>

<script lang="ts">
import ConfigHelper from '@/util/config-helper'
import { LDFlags } from '@/util/constants'
import LaunchDarklyService from 'sbc-common-components/src/services/launchdarkly.services'
import { appendAccountId } from 'sbc-common-components/src/util/common-util'
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
  name: 'NameRequestButton',
  props: {
    isWide: {
      default: false
    },
    isInverse: {
      default: false
    }
  },
  setup (props) {
    // open Name Request in current tab to retain current account and user
    const goToNameRequest = (): void => {
      if (LaunchDarklyService.getFlag(LDFlags.LinkToNewNameRequestApp)) {
        window.location.href = appendAccountId(ConfigHelper.getNameRequestUrl())
      } else {
        window.location.href = appendAccountId(ConfigHelper.getNroUrl())
      }
    }

    return {
      goToNameRequest
    }
  }
})
</script>

<style lang="scss" scoped>
  @import '$assets/scss/theme.scss';

  .btn-name-request {
    font-weight: bold;
    min-height: 2.75rem;
    width: 10rem;
    color: $BCgovBlue5;
    border: 2px solid $BCgovBlue5;
  }

  .btn-name-request-wide {
    margin-bottom: 0.8125rem;
  }

  .v-btn.btn-name-request-inverse {
    color: white !important;
    background-color: $BCgovBlue5;
  }
</style>
