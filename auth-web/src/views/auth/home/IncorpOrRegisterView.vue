<template>
  <v-container id="incorporate-info-container">
    <v-row>
      <!-- Info Column -->
      <v-col cols="12" md="6">
        <h2>Register or Incorporate</h2>
        <v-list class="py-0 mt-4 mb-6" color="transparent">
          <v-list-item class="list-item">
            <v-icon size="8" class="list-item-bullet mt-5">mdi-square</v-icon>
            <v-list-item-content>
              <v-list-item-subtitle class="list-item-text">
                Once your Name Request is approved, visit
                <a class="body-1 font-weight-bold my-registry-text" @click="emitRedirectManage()">
                  My Business Registry</a>
                page and use the Name Request Number to:
              </v-list-item-subtitle>
              <template>
                <v-list-item class="list-item" v-for="(item, index) in bulletPointList" :key="index" >
                  <v-icon size="8" class="list-item-bullet mt-5">mdi-square</v-icon>
                  <v-list-item-content>
                    <v-list-item-subtitle class="list-item-text">
                      {{item.text}}
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </template>
            </v-list-item-content>
          </v-list-item>
          <div class="pb-3">To register or incorporate, you will be asked for the following information:</div>

          <!-- enableBcCccUlc feature flag-->
          <template v-if="enableBcCccUlc">
            <v-expansion-panels flat tile accordion>
              <v-expansion-panel class="incorp-expansion-panels">
                <v-expansion-panel-header class="incorp-expansion-header font-weight-bold">
                  <template v-slot:actions>
                    <v-icon color="primary" class="incorp-icon">
                      $expand
                    </v-icon>
                  </template>
                  Sole Proprietorship, DBA, and General Partnership
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <span id="expPnlContent1" class="list-item-text py-3">
                    The name(s) and address(es) of the proprietor or partner(s).</span>
                </v-expansion-panel-content>
              </v-expansion-panel>
              <v-tooltip top max-width="450px" light content-class="tooltip">
                <template v-slot:activator="{ on }">
                  <v-expansion-panel class="incorp-expansion-panels">
                    <v-expansion-panel-header class="incorp-expansion-header font-weight-bold">
                      <template v-slot:actions>
                        <v-icon color="primary">
                          $expand
                        </v-icon>
                      </template>
                      <span class="tooltip-text" v-on="on">B.C. Based Company</span>
                    </v-expansion-panel-header>
                    <v-expansion-panel-content>
                      <span id="expPnlContent2" class="list-item-text py-3">
                        Office addresses, director names and addresses, share structure and articles.</span>
                    </v-expansion-panel-content>
                  </v-expansion-panel>
                </template>
                <v-card class="tooltip-content">
                  <div>
                    <h3 class="mb-3">B.C. Based Company</h3>
                    <span>You can incorporate the following B.C. based company types: Limited Company, Unlimited
                      Liability Company, Benefit Company, and Community Contribution Company.</span>
                  </div>
                </v-card>
              </v-tooltip>
              <v-expansion-panel class="incorp-expansion-panels">
                <v-expansion-panel-header class="incorp-expansion-header font-weight-bold">
                  <template v-slot:actions>
                    <v-icon color="primary">
                      $expand
                    </v-icon>
                  </template>
                  Cooperative Association
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <span id="expPnlContent3" class="list-item-text py-3">
                    Office addresses, director names and addresses, rules of the association
                    and memorandum.</span>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </template>
          <template v-else>
            <v-expansion-panels flat tile accordion>
              <v-expansion-panel v-for="(item, index) in expansionPanels" :key="index" class="incorp-expansion-panels">
                <v-expansion-panel-header class="incorp-expansion-header font-weight-bold">
                  <template v-slot:actions>
                    <v-icon color="primary">
                      $expand
                    </v-icon>
                  </template>
                  {{ item.text }}
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <v-list-item-content>
                      <v-list-item class="list-item" v-for="(subItem, subIndex) in item.items" :key="subIndex">
                        <v-list-item-content>
                          <v-list-item-subtitle class="list-item-text py-3">
                            {{subItem.text}}
                          </v-list-item-subtitle>
                        </v-list-item-content>
                      </v-list-item>
                    </v-list-item-content>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </template>
        </v-list>
        <!-- Panel Btns -->
        <template>
          <v-btn large color="bcgovblue" class="cta-btn font-weight-bold mr-2 white--text registry-btn"
            @click="emitRedirectManage()">
            Go to My Business Registry
          </v-btn>
        </template>
        <LearnMoreButton isWide=true :redirect-url="learnMoreUrl"/>
        <div class="d-flex mt-8">
          <span class="body-1">New to BC Registries?</span>
          <router-link class="ml-2 body-1 font-weight-bold"
            to="/choose-authentication-method"
          >Create a BC Registries Account
          </router-link>
        </div>
      </v-col>
      <!-- Image Column -->
      <v-col cols="12" md="6">
        <a :href="learnMoreUrl" target="_blank">
          <v-img src="../../../assets/img/Step3_Incorporate_x2.png" aspect-ratio="1.2" contain></v-img>
        </a>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from 'vue-property-decorator'
import ConfigHelper from '@/util/config-helper'
import { LDFlags } from '@/util/constants'
import LaunchDarklyService from 'sbc-common-components/src/services/launchdarkly.services'
import LearnMoreButton from '@/components/auth/common/LearnMoreButton.vue'
import NumberedCompanyTooltip from '@/components/auth/common/NumberedCompanyTooltip.vue'
import { User } from '@/models/user'
import { appendAccountId } from 'sbc-common-components/src/util/common-util'

@Component({
  components: {
    LearnMoreButton,
    NumberedCompanyTooltip
  }
})
export default class IncorpOrRegisterView extends Vue {
  protected readonly learnMoreUrl = 'https://www2.gov.bc.ca/gov/content/governments/organizational-structure/ministries-organizations/ministries/citizens-services/bc-registries-online-services'

  // For BEN only as feature flag 'EnableBcCccUlc' enabled
  readonly bulletPointsBEN: Array<any> = [
    { text: 'Register a firm such as a sole proprietorship, a Doing Business As name (DBA), or a general partnership.' },
    { text: 'Incorporate a benefit company or a cooperative association.' }
  ]

  // Use this when feature flag 'EnableBcCccUlc' no longer used. Change name and refrences
  readonly bulletPointsIA: Array<any> = [
    { text: 'Register a firm such as a sole proprietorship, a Doing Business As name (DBA), or a general partnership.' },
    { text: 'Incorporate a B.C. based company or a cooperative association.' }
  ]

  // For BEN only as feature flag 'EnableBcCccUlc' enabled
  private readonly expansionPanels: Array<any> = [
    { text: 'Sole Proprietorship, DBA, and General Partnership',
      items: [
        { text: 'The name(s) and address(es) of the proprietor or partner(s).' }
      ]
    },
    { text: 'Benefit Company',
      items: [
        { text: 'Office addresses, director names and addresses, share structure and articles.' }
      ]
    },
    { text: 'Cooperative Association',
      items: [
        { text: 'Office addresses, director names and addresses, rules of the association and memorandum.' }
      ]
    }
  ]

  @Prop()
  private userProfile: User

  private emitRedirectManage () {
    if (this.userProfile) {
      this.emitManageBusinesses()
    } else {
      window.location.assign(appendAccountId(`${ConfigHelper.getRegistryHomeURL()}dashboard`))
    }
  }

  @Emit('login')
  private emitLogin (): void {}

  @Emit('account-dialog')
  private emitAccountDialog (): void {}

  @Emit('manage-businesses')
  private emitManageBusinesses (): void {}

  get enableBcCccUlc (): boolean {
    return LaunchDarklyService.getFlag(LDFlags.EnableBcCccUlc) || false
  }

  get bulletPointList (): Array<any> {
    if (this.enableBcCccUlc) {
      return this.bulletPointsIA
    } else {
      return this.bulletPointsBEN
    }
  }
}
</script>

<style lang="scss" scoped>
  @import '$assets/scss/theme.scss';

  #incorporate-info-container {
    padding-top: 0 !important;
    flex-wrap: wrap;

    a:hover {
      color: $BCgoveBueText2;
    }

    .list-item {
      align-items: flex-start;
      padding-left: 0;
    }

    .list-item-bullet {
      color: $BCgovBullet;
      margin-right: 1rem;
    }

    .list-item-text {
      white-space: initial;
      color: $gray7;
      font-size: 1rem;
      letter-spacing: 0;
      line-height: 1.5rem;
    }

    .v-btn:hover {
      opacity: .8;
    }

    .incorporate-btns {
      flex-direction: column;

      .incorporate-btn {
        align-self: flex-start;
      }
    }

    .incorp-expansion-panels {
      background-color: inherit;
    }

    .incorp-expansion-header {
      color: $BCgoveBueText1;
    }

    .v-expansion-panel-content .v-list-item__content {
      padding: 0;
    }

    .registry-btn:hover {
      color: white !important;
      opacity: .8;
    }

    .my-registry-text {
      text-decoration: underline;
    }
  }

  .tooltip {
    background-color: transparent;
    opacity: 1 !important;

    .tooltip-content {
      min-width: 30rem;
      padding: 2rem;
      font-size: $px-12;
    }
  }

  .tooltip-text {
    text-decoration: underline dotted;
    text-underline-offset: 2px;
  }

  .tooltip-text:hover {
    cursor: pointer;
  }
</style>
