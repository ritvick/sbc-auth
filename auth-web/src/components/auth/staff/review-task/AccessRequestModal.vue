<template>
  <div>
    <ModalDialog
      max-width="643"
      :isPersistent="true"
      ref="accessRequest"
      icon="mdi-check"
      data-test="dialog-access-request"
      dialog-class="notify-dialog"
    >

      <template v-slot:icon>
        <v-icon large :color="getModalData.color">{{getModalData.icon}}</v-icon>
      </template>
      <template v-slot:title>
        <span class="font-weight-bold text-size mb-1" data-test="dialog-header"> {{ getModalData.title }} </span>
      </template>
      <template v-slot:text>
        <div class="mx-8">
          <p class="mb-4 text-color sub-text-size text-justify" v-html="getModalData.text" data-test="p-modal-text"></p>
          <v-form ref="rejectForm" lazy-validation class="reject-form" data-test="reject-form" v-if="isOnHoldModal">
            <v-row justify="center">
              <v-col cols="6" class="pa-0">
                <v-radio-group
                  v-model="accountToBeOnholdOrRejected"
                  :rules="accountToBeOnholdOrRejectedRules"
                  data-test="radio-group-hold-or-reject"
                  class="mt-0"
                >
                  <v-row dense class="d-flex flex-column align-items-center">
                    <v-col>
                      <v-radio
                        label="Reject Account"
                        :key="OnholdOrRejectCode.REJECTED"
                        :value="OnholdOrRejectCode.REJECTED"
                        data-test="radio-reject"
                      >
                      </v-radio>
                    </v-col>
                    <v-col>
                      <v-radio
                        label="On Hold"
                        :key="OnholdOrRejectCode.ONHOLD"
                        :value="OnholdOrRejectCode.ONHOLD"
                        data-test="radio-on-hold"
                      ></v-radio>
                    </v-col>
                  </v-row>
                  <template v-slot:message="{ message }">
                    <span class="error-size"> {{ message }} </span>
                  </template>
                </v-radio-group>
              </v-col>
            </v-row>
            <v-select
              filled
              label="Reason(s) why account is on hold "
              :items="onholdReasonCodes"
              item-text="desc"
              item-value="desc"
              v-model="onholdReasons"
              data-test="hold-reason-type"
              class="my-0"
              :rules="onholdReasonRules"
              multiple
              v-if="accountToBeOnholdOrRejected === OnholdOrRejectCode.ONHOLD"
            >
              <template v-slot:selection="{ item, index }">
                <span v-if="index === 0">{{ item.desc }}</span>
                <span
                  v-if="index === 1"
                  class="grey--text text-caption"
                >
                  (+{{ onholdReasons.length - 1 }} {{ onholdReasons.length > 2 ? 'others' : 'other' }})
                </span>
              </template>
            </v-select>

          </v-form>
          <v-form ref="rejectForm" lazy-validation class="reject-form" data-test="reject-form" v-if="isMoveToPendingModal">
            <v-row justify="start">
              <v-col cols="6" class="pa-0">
                <v-row dense class="d-flex flex-column align-items-center">
                  <v-checkbox class="mt-0 ml-4" @change="toggleReason('Request was rejected in error')">
                    <template v-slot:label>
                      Request was rejected in error
                    </template>
                  </v-checkbox>
                </v-row>
                <v-row dense class="d-flex flex-column align-items-start">
                  <v-checkbox class="mt-0 ml-4">
                    <template v-slot:label>
                      Other reason
                    </template>
                  </v-checkbox>
                </v-row>
              </v-col>
            </v-row>
            <v-row dense class="d-flex" justify="end">
              <v-col cols="11" class="pa-0">
                <v-text-field
                  filled
                  label="Reason will be displayed in the email sent to user"
                  req
                  persistent-hint
                  full-width
                  :counter="50"
                  v-model="otherReasonText"
                >
                </v-text-field>
              </v-col>
            </v-row>
          </v-form>
        </div>
      </template>
      <template v-slot:actions>
        <v-btn large :color="getModalData.color" @click="callAction()"
          class="font-weight-bold px-4"
          :loading="isSaving"
          data-test="btn-access-request"
        >{{getModalData.btnLabel}}</v-btn>
        <v-btn large outlined color="primary" @click="close()" data-test="btn-close-access-request-dialog">Cancel</v-btn>
      </template>
    </ModalDialog>

    <!-- confirmation modal -->
    <ModalDialog
      ref="accessRequestConfirmationDialog"
      :isPersistent="true"
      :title="getConfirmModalData.title"
      :text="getConfirmModalData.text"
      dialog-class="notify-dialog"
      max-width="640"
    >
      <template v-slot:icon>
        <v-icon large color="primary">mdi-check</v-icon>
      </template>
      <template v-slot:text>
        <p class="mx-5" v-html="getConfirmModalData.text"></p>
      </template>
      <template v-slot:actions>
        <v-btn
          large
          color="primary"
          class="font-weight-bold"
          @click="onConfirmCloseClick"
        >
          OK
        </v-btn>
      </template>
    </ModalDialog>

  </div>
</template>

<script lang="ts">
import { OnholdOrRejectCode, TaskRelationshipType } from '@/util/constants'
import { Ref, computed, defineComponent, onMounted, ref } from '@vue/composition-api'
import ModalDialog from '@/components/auth/common/ModalDialog.vue'
import { useI18n } from 'vue-i18n-bridge'

export default defineComponent({
  name: 'AccessRequestModal',
  props: {
    isRejectModal: {
      type: Boolean,
      default: false
    },
    isTaskRejected: {
      type: Boolean,
      default: false
    },
    isConfirmationModal: {
      type: Boolean,
      default: false
    },
    isSaving: {
      type: Boolean,
      default: false
    },
    isOnHoldModal: { // for BECID need hold or reject options
      type: Boolean,
      default: false
    },
    isMoveToPendingModal: {
      type: Boolean,
      default: false
    },
    orgName: {
      type: String,
      default: ''
    },
    accountType: {
      type: String,
      default: ''
    },
    taskName: {
      type: String,
      default: ''
    }
  },
  components: {
    ModalDialog
  },
  setup (props, { emit }) {
    const { t } = useI18n()
    const onholdReasons: Ref<string[]> = ref([])
    const accountToBeOnholdOrRejected: Ref<string> = ref('')
    const accessRequest: Ref<ModalDialog> = ref(null)
    const accessRequestConfirmationDialog: Ref<ModalDialog> = ref(null)
    const rejectForm: Ref<HTMLFormElement> = ref(null)
    const onholdReasonCodes: Ref<string[]> = ref([])
    const otherReasonText: Ref<string> = ref('')

    const onholdReasonRules = [
      (v) => v.length > 0 || 'This field is required'
    ]
    const accountToBeOnholdOrRejectedRules = [
      (v) => !!v || 'Choose reject or on hold to proceed.'
    ]

    function openConfirm () {
      accessRequestConfirmationDialog.value.open()
    }

    function closeConfirm () {
      accessRequestConfirmationDialog.value.close()
    }

    function onConfirmCloseClick () {
      emit('after-confirm-action')
      closeConfirm()
    }

    function callAction () {
      let isValidForm = true
      let accountToBeOnholdOrRejected

      if (props.isOnHoldModal) {
        isValidForm = rejectForm.value.validate()
      } else if (props.isMoveToPendingModal) {
        accountToBeOnholdOrRejected = OnholdOrRejectCode.ONHOLD
      }

      if (props.isMoveToPendingModal && otherReasonText.value) {
        onholdReasons.value.push(otherReasonText.value)
      }

      emit('approve-reject-action', {
        isValidForm,
        accountToBeOnholdOrRejected,
        onholdReasons: onholdReasons.value
      })
    }

    const getTitle = (isProductApproval: boolean) => {
      if (isProductApproval) {
        if (props.isTaskRejected) return 'Re-approve Access Request ?'
        else return 'Approve Access Request ?'
      } else {
        return 'Approve Account Creation Request ?'
      }
    }

    const getText = (isProductApproval: boolean) => {
      if (isProductApproval) {
        if (props.isTaskRejected) {
          return `By re-approving the request, this account will have access to  ${props.taskName}`
        } else {
          return `By approving the request, this account will have access to  ${props.taskName}`
        }
      }
      return `Approving the request will activate this account`
    }

    const getIcon = (isRejectModal: boolean) => {
      if (isRejectModal) return 'mdi-alert-circle-outline'
      return 'mdi-help-circle-outline'
    }

    const getColor = (isRejectedModal: boolean) => {
      if (isRejectedModal) return 'error'
      return 'primary'
    }

    const getModalData = computed(() => {
      const isProductApproval =
        props.accountType === TaskRelationshipType.PRODUCT

      let title: string = getTitle(isProductApproval)
      let text: string = getText(isProductApproval)

      let icon = getIcon(props.isRejectModal)
      let color = getColor(props.isRejectModal)
      let btnLabel = props.isTaskRejected ? 'Re-approve' : 'Approve'

      if (props.isRejectModal) {
        title = isProductApproval
          ? 'Reject Access Request ?'
          : 'Reject Account Creation Request ?'

        text = isProductApproval // eslint-disable-next-line no-irregular-whitespace
          ? `By rejecting the request, this account won't have access to ${props.taskName}`
          : 'Rejecting the request will not activate this account'
        btnLabel = isProductApproval ? 'Reject' : 'Yes, Reject Account'
      } else if (props.isOnHoldModal) {
        // if we need to show on hold modal
        title = 'Reject or Hold Account Creation Request'

        text = t('onHoldOrRejectModalText').toString()

        btnLabel = 'Confirm'
      } else if (props.isMoveToPendingModal) {
        title = 'Move to Pending'
        text =
          'To place a rejected access request on hold, please select a reason. An email will be sent to the user to notify the action.'
        btnLabel = 'Confirm'
      }
      return { title, text, icon, color, btnLabel }
    })

    const getConfirmModalData = computed(() => {
      const isProductApproval =
        props.accountType === TaskRelationshipType.PRODUCT

      let title = isProductApproval
        ? `Request has been Approved`
        : `Account has been Approved`

      let text = isProductApproval
        ? `The account <strong>${props.orgName}</strong> has been approved to access ${props.taskName}`
        : `Account creation request has been approved`

      if (props.isRejectModal) {
        title = isProductApproval
          ? `Request has been Rejected`
          : `Account has been Rejected`
        // eslint-disable-next-line no-irregular-whitespace
        text = isProductApproval
          ? `The account <strong>${props.orgName}</strong> has been rejected to access ${props.taskName}`
          : `Account creation request has been rejected`
      } else if (props.isOnHoldModal || props.isMoveToPendingModal) {
        title = 'Request is On Hold'

        text =
          'An email has been sent to the user presenting the reason why the account is on hold, and a link to resolve the issue.'
      }
      return { title, text }
    })

    const open = () => {
      accessRequest.value.open()
    }

    const close = () => {
      accessRequest.value.close()
    }

    const toggleReason = (reason: string) => {
      if (onholdReasons.value.includes(reason)) {
        onholdReasons.value = onholdReasons.value.filter((r) => r !== reason)
      } else {
        onholdReasons.value.push(reason)
      }
    }

    onMounted(() => {
      if (!props.isOnHoldModal) { onholdReasons.value = [] }
    })

    return {
      OnholdOrRejectCode,
      accessRequest,
      accessRequestConfirmationDialog,
      accountToBeOnholdOrRejected,
      accountToBeOnholdOrRejectedRules,
      callAction,
      close,
      closeConfirm,
      getConfirmModalData,
      getModalData,
      onholdReasonCodes,
      onholdReasonRules,
      onholdReasons,
      onConfirmCloseClick,
      open,
      openConfirm,
      otherReasonText,
      rejectForm,
      toggleReason
    }
  }
})
</script>

<style lang="scss" >
  @import '$assets/scss/theme.scss';
  .reject-form{
    margin-bottom: -30px !important;
  }
  .reject-form .v-messages__message{
    color: var(--v-error-darken2) !important;
    caret-color: var(--v-error-darken2) !important;
  }
  .text-color {
    color: $TextColorGray;
  }
  .text-size {
    font-size: 1.7142rem !important;
  }
  .sub-text-size {
    font-size: 1.1428rem !important;
  }
  .align-items-center {
    align-self: center !important;
  }
  .error-size {
    font-size: 16px;
  }
</style>
