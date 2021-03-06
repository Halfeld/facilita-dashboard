<template>
  <div class="layout" :class="classObject" v-resize>
    <navbar></navbar>
    <sidebar></sidebar>
    <div class="content-wrap" id="content-wrap">
      <main id="content" class="content" role="main">
        <vuestic-breadcrumbs v-if="!cannotShowRouter" :breadcrumbs="breadcrumbs"/>
        <vuestic-pre-loader v-show="isLoading" ref="preLoader" class="pre-loader"></vuestic-pre-loader>
        <router-view v-if="!cannotShowRouter"></router-view>
        <Profile v-if="cannotShowRouter" :profiles="profilesSorted" />
      </main>
    </div>
    <div class="made-by-footer">
      ©2018. copyright - facilita
    </div>
  </div>
</template>

<script>
  import {
    mapGetters,
    mapState
  } from 'vuex'

  import {filter} from 'lodash'
  import Navbar from './navbar/Navbar'
  import Sidebar from './sidebar/Sidebar'
  import Resize from 'directives/ResizeHandler'
  import Profile from 'components/profile/Profile.vue'

  export default {
    name: 'layout',
    components: {
      Navbar,
      Sidebar,
      Profile
    },
    directives: {
      resize: Resize
    },
    props: {
      fixed: {
        type: Boolean,
        default: false,
      }
    },
    data: () => ({
      profiles: [],
      profilesSorted: []
    }),
    created () {
      if (!this.profiles.length) {
        fetch('https://facilitafn.azurewebsites.net/api/FacilitaListarEmpresas?code=5YpoqVzrIMNKICdD50CdlMG5IxLmqXBSXif2BFXW4MAK/Axwc1nRuA==')
          .then(res => res.text())
          .then(JSON.parse)
          .then(JSON.parse)
          .then(res => {
            console.log(res)
            this.profiles = res
            this.profilesSorted = res
          })
      }
    },
    watch: {
      searchWord () {
        console.log('Search word called', this.searchWord)
        this.profilesSorted = filter(this.profiles, (p) => {
          if (!this.searchWord) return p
          const regexp = new RegExp(`${this.searchWord}*`, 'ig')
          if (regexp.test(p.name)) {
            return p
          }
        })
      }
    },
    computed: {
      ...mapState({
        searchBarStatus: (state) => state.ui.searchBarStatus,
        searchWord: (state) => state.ui.searchWord
      }),
      ...mapGetters([
        'sidebarOpened',
        'toggleWithoutAnimation',
        'isLoading'
      ]),
      cannotShowRouter () {
        return !this.isLoading && this.searchBarStatus
      },
      classObject: function () {
        return {
          'layout-fixed': this.fixed,
          'sidebar-hidden': !this.toggleWithoutAnimation && !this.sidebarOpened,
          'sidebar-hidden sidebar-hidden_without-animation': this.toggleWithoutAnimation && !this.sidebarOpened
        }
      },
      breadcrumbs () {
        return this.$store.getters.breadcrumbs(this.$route.name)
      }
    }
  }
</script>

<style lang="scss">
  @import "../../sass/_variables.scss";
  @import "~bootstrap/scss/mixins/breakpoints";
  @import "~bootstrap/scss/functions";
  @import "~bootstrap/scss/variables";

  .layout {
    &-fixed {
      .content-wrap {
        padding-right: $layout-padding-right;
        padding-top: $sidebar-top;

        @include media-breakpoint-down(md) {
          padding: $content-mobile-wrap-fixed-layout;
          margin-left: 0;

        }
      }
    }
  }

  .content-wrap {
    margin-left: $content-wrap-ml;
    transition: margin-left 0.3s ease;
    padding-right: $layout-padding-right;
    padding-top: $layout-padding;

    padding-bottom: $content-wrap-pb;

    .pre-loader {
      position: absolute;
      left: $vuestic-preloader-left;
      top: $vuestic-preloader-top;
    }

    .sidebar-hidden & {
      margin-left: $sidebar-left;
    }

    @include media-breakpoint-down(md) {
      padding: $content-mobile-wrap;
      margin-left: 0;

      .sidebar-hidden & {
        margin-left: 0;
        padding-top: $content-mobile-wrap-sb-top;
      }
    }
  }

  .made-by-footer {
    padding-top:25px;
    position: absolute;
    bottom: 0;
    padding-bottom: $made-by-footer-pb;
    height: calc(#{$layout-padding} + #{$widget-mb});
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>
