<template>
  <v-app>
    <v-content v-bind:style="{paddingBottom: (footer ? 56 : 0) + 'px'}" style="padding-top:0">
      <keep-alive>
        <router-view />
      </keep-alive>
    </v-content>
    <mobile-footer v-show="footer"></mobile-footer>
    <v-dialog v-model="loading" hide-overlay persistent fullscreen>
      <v-card height="92" min-height="92" class="loading">
        <v-card-text class="loading-progress">
          <v-progress-circular indeterminate size="60" color="primary"></v-progress-circular>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-snackbar v-model="snackbar.display" :timeout="snackbar.timeout">
      {{snackbar.text}}
      <v-btn color="primary" flat @click="snackbar.display = false">
        关闭
      </v-btn>
    </v-snackbar>
    <!--this is tricky try fix for bottom-nav display ugly on iphone X  -->
    <div style="z-index: 5;width: 100%;height: 45px;bottom: -45px;background: white;position: fixed;">
    </div>
  </v-app>
</template>

<script>
import _ from "lodash";

export default {
  name: "App",
  data() {
    return {
      title: "铁塔测量",
      loading: false,
      snackbar: {
        display: false,
        text: null,
        timeout: 5000
      },
      allNavs: [
        
      ]
    };
  },
  computed: {
    footer: function() {
      const to = this.$route;
      return _.isNil(to.meta) || _.isNil(to.meta.footer) || to.meta.footer;
    }   
  },
  created: function() {
    this.$bus.$on("show-loading", () => {
      this.loading = true;
    });
    this.$bus.$on("hide-loading", () => {
      this.loading = false;
    });
    this.$bus.$on("toast", ({ text, timeout }) => {
      this.snackbar = {
        text: text,
        timeout: timeout,
        display: true
      };
    });
  },
  mounted: function() {
    if (window.cordova) {
      navigator.splashscreen && navigator.splashscreen.hide();
    }
  }
};
</script>
<style scoped>
.loading {
  background: #000;
  opacity: 0.6;
  display: flex;
  justify-content: center;
  align-items: center;
}
.loading-progress {
  text-align: center;
}
</style>
