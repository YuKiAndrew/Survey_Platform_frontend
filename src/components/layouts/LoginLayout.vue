<template>
  <div id="dw-layout" style="height: 100vh;" class="dw-layout-body" >
    <el-container>
      <el-container style="padding: 0px;height: calc(100vh - 80px);">
        <el-header height="80" style="padding: 0px;" >
          <dw-header-login></dw-header-login>
        </el-header>
        <el-container style="z-index: 1;">
          <el-container>
            <el-main style="padding: 20px 0px;" >
              <router-view></router-view>
            </el-main>
          </el-container>
        </el-container>
      </el-container>
      <el-footer height="80" class="dw-footer-container"><dw-footer></dw-footer></el-footer>
    </el-container>
  </div>
</template>
<script>

import DwHeader from '@/components/layouts/DwHeader.vue'
import DwHeaderLogin from '@/components/layouts/DwHeaderLogin.vue'
import DwNavMenu from '@/components/layouts/DwNavMenu.vue'
import DwFooter from '@/components/layouts/DwFooter.vue'

export default {
  name: 'LoginLayout',
  components: {
    'dw-nav-menu': DwNavMenu,
    'dw-header': DwHeader,
    'dw-header-login': DwHeaderLogin,
    'dw-footer': DwFooter
  },
  data () {
    return {
      currentHeader: 'dw-header',
      pageH: '600px'
    }
  },
  mounted () {
    this.loginStatus()
    this.pageH = `${window.innerHeight}px`
  },
  methods: {
    loginStatus () {
      const fullPath = this.$route.fullPath
      if (fullPath.indexOf('login') >= 0) {
        this.currentHeader = 'dw-header-login'
      } else {
        this.currentHeader = 'dw-header'
      }
    }
  }
}
</script>
<style scoped>
.dw-footer-container{
  bottom: 0px;
  width: 100%;
  text-align: center;
  background: #F1F1F1;
  padding-top: 20px;
}
.dw-layout-body{
  background-image: url("/static/diaowen/images/style-model/login_bg/1.jpg");
  -moz-background-size: cover;
  background-size: cover;
  background-repeat: no-repeat;
}
</style>
