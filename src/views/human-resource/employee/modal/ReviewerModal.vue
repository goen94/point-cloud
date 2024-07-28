<template>
  <div>
    <form class="row">
      <sweet-modal
        ref="modal"
        :title="$t('add reviewer') | uppercase"
        overlay-theme="dark"
        @close="onClose()"
      >
        <div class="list-group mb-20">
          <template v-for="(user, index) in users">
            <a
              :key="index"
              class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
              href="javascript:void(0)"
              @click="onSubmitReviewer(user, index)"
            >
              <span><i class="fa fa-fw fa-hand-o-right mr-5" /> {{ user.first_name + ' ' + user.last_name }}</span>
            </a>
          </template>
        </div>
      </sweet-modal>
    </form>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

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
      reviewer: null
    }
  },
  computed: {
    ...mapGetters('masterUser', ['users'])
  },
  watch: {
    'reviewer' () {
      this.$emit('value', this.reviewer)
    }
  },
  created () {
    this.get()
  },
  methods: {
    ...mapActions('masterUser', ['get']),
    open () {
      this.$refs.modal.open()
    },
    close () {
      this.$refs.modal.close()
    },
    onClose () {
      this.$emit('close')
    },
    onSubmitReviewer (reviewer) {
      this.$emit('add', {
        id: reviewer.id,
        name: reviewer.first_name + ' ' + reviewer.last_name
      })
      this.reviewer = ''
      this.$refs.modal.close()
    }
  }
}
</script>
