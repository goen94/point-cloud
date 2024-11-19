<template>
  <footer
    id="page-footer"
  >
    <div class="content py-20 font-size-xs clearfix">
      <div class="float-right">
        <div>
          <div style="position: relative;">
            <a href="button" class="btn btn-success btn-sm" style="height: 32px; border-radius: 20px; position: fixed; bottom: 12px; right:12px;">
              <img src="/whatsapp.png" alt="" style="width: 24px; height: 24px;">
              Contact Us
            </a>
          </div>
          <div>
            Crafted with <i class="fa fa-heart text-pulse" /> by <a
              class="font-w600"
              href="javascript:void(0)"
              target="_blank"
            >RED POINT</a>
          </div>
        </div>
      </div>
      <div class="float-left">
        VERSION {{ version }} | &copy; 2020
      </div>
    </div>
  </footer>
</template>

<script>
export default {
  data () {
    return {
      version: process.env.VUE_APP_VERSION
    }
  }
}
</script>
