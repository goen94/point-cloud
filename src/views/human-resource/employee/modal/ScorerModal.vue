<template>
  <div>
    <form class="row">
      <sweet-modal
        ref="modal"
        :title="$t('add scorer') | uppercase"
        overlay-theme="dark"
        @close="onClose()"
      >
        <input
          ref="searchText"
          v-model="searchText"
          type="text"
          class="form-control"
          placeholder="Search..."
          @keydown.enter.prevent=""
        >
        <hr>
        <div v-if="isLoading">
          <h3 class="text-center">
            Loading ...
          </h3>
        </div>
        <div v-else class="list-group mb-20">
          <template v-for="(user, index) in options" :key="index">
            <a
              class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
              href="javascript:void(0)"
              @click="onSubmitScorer(user, index)"
            >
              <span><i class="fa fa-fw fa-hand-o-right mr-5" /> {{ user.first_name + ' ' + user.last_name }}</span>
            </a>
          </template>
        </div>
        <div
          v-if="searchText && options.length == 0 && !isLoading"
          class="alert alert-info text-center"
        >
          {{ $t('searching not found', [searchText]) | capitalize }} <br>
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
      scorer: null,
      searchText: '',
      options: [],
      isLoading: false
    }
  },
  computed: {
    ...mapGetters('masterUser', ['users'])
  },
  watch: {
    'scorer' () {
      this.$emit('value', this.scorer)
    },
    searchText: debounce(function () {
      this.search()
    }, 300),
  },
  created () {
    this.get({ params: { limit: 99999 }})
  },
  methods: {
    ...mapActions('masterUser', ['get']),
    search () {
      this.isLoading = true
      this.get({
        params: {
          sort_by: 'name',
          limit: 50,
          filter_like: {
            first_name: this.searchText,
            last_name: this.searchText
          }
        }
      }).then(response => {
        this.options = []
        response.data.map((key, value) => {
          const obj = {
            id: key.id,
            first_name: key.first_name,
            last_name: key.last_name
          }
          this.options.push({
            ...obj
          })
        })
        this.isLoading = false
      }).catch(error => {
        this.isLoading = false
      })
    },
    open () {
      this.$refs.modal.open()
    },
    close () {
      this.$refs.modal.close()
    },
    onClose () {
      this.$emit('close')
    },
    onSubmitScorer (scorer) {
      this.$emit('add', {
        id: scorer.id,
        name: scorer.first_name + ' ' + scorer.last_name
      })
      this.scorer = ''
      this.$refs.modal.close()
    }
  }
}
</script>
