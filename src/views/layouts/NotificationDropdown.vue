<template>
  <div
    v-click-outside="close"
    class="btn-group"
    role="group"
  >
    <a
      href="javascript:void(0)"
      class="btn text-black"
      style="margin-top: -4px; padding: 8px !important"
      @click="toggle"
    >
      <i class="fa fa-bell" style="font-size: 18px;" />
    </a>
    <div
      class="dropdown-menu dropdown-menu-right min-width-300"
      style="right: -130px;"
      :class="{'show': show}"
    >
      <h5 class="h6 text-center py-10 mb-0 border-b text-uppercase">
        Notifications
      </h5>
      <ul class="list-unstyled my-20">
        <li
          v-for="(notification, index) in notifications"
          :key="index"
        >
          <a
            class="text-body-color-dark media mb-15"
            :href="'//'+notification.clickAction"
          >
            <div class="ml-5 mr-15">
              <i
                v-if="notification.type == 'info'"
                class="fa fa-fw fa-info-circle text-primary"
              />
              <i
                v-if="notification.type == 'warning'"
                class="fa fa-fw fa-warning text-warning"
              />
              <i
                v-if="notification.type == 'reminder'"
                class="fa fa-fw fa-calendar text-info"
              />
            </div>
            <div class="media-body pr-10">
              <p class="mb-0">{{ notification.message }}</p>
              <div class="text-muted font-size-sm font-italic">{{ notification.createdAt | fromNow }}</div>
            </div>
          </a>
        </li>
      </ul>
      <div class="dropdown-divider" />
      <a
        class="dropdown-item text-center mb-0"
        href="javascript:void(0)"
      >
        <i class="fa fa-bell mr-5" /> View All
      </a>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase/app'
import 'firebase/firestore'
import { mapGetters } from 'vuex'

export default {
  data () {
    return {
      show: false,
      notifications: []
    }
  },
  computed: {
    ...mapGetters('auth', ['authUser'])
  },
  watch: {
    authUser () {
      var self = this
      if (this.authUser) {
        firebase.firestore().collection('notifications')
          .orderBy('createdAt', 'desc')
          .where('userId', '==', this.authUser.id)
          .limit(4)
          .onSnapshot(function (querySnapshot) {
            self.notifications = []
            querySnapshot.forEach(function (doc) {
              const data = {
                type: 'info',
                message: doc.data().message,
                clickAction: doc.data().clickAction,
                createdAt: doc.data().createdAt
              }
              self.notifications.push(data)
            })
          })
      }
    }
  },
  created () {

  },
  methods: {
    toggle () {
      this.show = !this.show
    },
    close () {
      this.show = false
    }
  }
}
</script>
