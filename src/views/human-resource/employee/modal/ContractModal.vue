<template>
  <div>
    <form
      class="row"
      @submit.prevent="onSubmitContract"
    >
      <sweet-modal
        ref="modal"
        :title="$t('add contract') | uppercase"
        overlay-theme="dark"
        @close="onClose()"
      >
        <p-form-row
          id="contract-date"
          :label="$t('contract begin')"
        >
          <div
            slot="body"
            class="col-lg-9"
          >
            <p-date-picker
              v-model="contract_begin"
              name="contract-date"
            />
          </div>
        </p-form-row>

        <p-form-row
          id="expired-date"
          :label="$t('contract end')"
        >
          <div
            slot="body"
            class="col-lg-9"
          >
            <p-date-picker
              v-model="contract_end"
              name="expired-date"
            />
          </div>
        </p-form-row>

        <p-form-row
          id="due-date"
          :label="$t('due date reminder contract')"
        >
          <div
            slot="body"
            class="col-lg-9"
          >
            <p-date-picker
              v-model="contract_due_date"
              name="due-date"
            />
          </div>
        </p-form-row>

        <p-form-row
          id="notes"
          v-model="notes"
          name="notes"
          :label="$t('notes')"
        />

        <div class="pull-right">
          <button
            type="submit"
            class="btn btn-sm btn-primary"
          >
            {{ $t('save') | uppercase }}
          </button>
        </div>
      </sweet-modal>
    </form>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      default: ''
    },
    id: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      contract_begin: '',
      contract_end: '',
      contract_due_date: '',
      notes: null
    }
  },
  watch: {
    'notes' () {
      this.$emit('value', this.notes)
    }
  },
  methods: {
    open () {
      this.$refs.modal.open()
    },
    close () {
      this.$refs.modal.close()
    },
    onClose () {
      this.$emit('close')
    },
    onSubmitContract () {
      const dueDate = new Date(this.contract_due_date)
      const endDate = new Date(this.contract_end)

      if (dueDate > endDate) {
        this.$notification.error(this.$t('reminder date exceeds contract date'))
        return
      }

      this.$emit('add', {
        contract_begin: this.contract_begin,
        contract_end: this.contract_end,
        contract_due_date: this.contract_due_date,
        notes: this.notes
      })
      this.contract = ''
      this.$refs.modal.close()
    }
  }
}
</script>
