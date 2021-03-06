<template>
  <div :class="{'show':show}" class="header-search">
    <svg-icon class-name="search-icon" icon-class="search" @click.stop="click" />
    <el-select
      ref="headerSearchSelect"
      v-model="search"
      :remote-method="querySearch"
      filterable
      default-first-option
      remote
      placeholder="Search"
      class="header-search-select"
      @change="change"
    >
      <el-option v-for="item in options" :key="item.path" :value="item" :label="item.title" />
    </el-select>
  </div>
</template>

<script>
// fuse is a lightweight fuzzy-search module
// make search results more in line with expectations
import Fuse from 'fuse.js'
// import path from 'path'
import routing from '@/utils/routing'

export default {
  name: 'HeaderSearch',
  data() {
    return {
      search: '',
      options: [],
      searchable: [],
      show: false,
      fuse: undefined
    }
  },
  computed: {
    routes() {
      return this.$store.getters.feature_routes
    }
  },
  watch: {
    routes() {
      this.initSearchable()
    },
    searchable(list) {
      this.initFuse(list)
    },
    show(value) {
      if (value) {
        document.body.addEventListener('click', this.close)
      } else {
        document.body.removeEventListener('click', this.close)
      }
    }
  },
  mounted() {
    this.initSearchable()
  },
  methods: {
    initSearchable() {
      const menu = routing.navigation(this.$store.state.routing.routes)
      const items = []
      const flatten_menus = function(menus) {
        menus.forEach((menu) => {
          items.push({ title: menu.title, icon: menu.icon, path: menu.path })
          if (menu.children) {
            flatten_menus(menu.children)
          }
        })
      }

      flatten_menus(menu)
      this.searchable = items
      // console.log('searchable: %o', this.searchable)
    },
    click() {
      this.show = !this.show
      if (this.show) {
        this.$refs.headerSearchSelect && this.$refs.headerSearchSelect.focus()
      }
    },
    close() {
      this.$refs.headerSearchSelect && this.$refs.headerSearchSelect.blur()
      this.options = []
      this.show = false
    },
    change(val) {
      this.$router.push(val.path)
      this.search = ''
      this.options = []
      this.$nextTick(() => {
        this.show = false
      })
    },
    initFuse(list) {
      this.fuse = new Fuse(list, {
        shouldSort: true,
        threshold: 0.4,
        location: 0,
        distance: 10,
        maxPatternLength: 32,
        minMatchCharLength: 1,
        keys: [{
          name: 'title',
          weight: 0.7
        }, {
          name: 'path',
          weight: 0.3
        }]
      })
    },
    // Filter out the routes that can be displayed in the sidebar
    // // And generate the internationalized title
    // generateRoutes(routes, basePath = '/', prefixTitle = []) {
    //   let res = []

    //   for (const router of routes) {
    //     // skip hidden router
    //     if (router.hidden) { continue }

    //     // console.log('generateRoutes: %o -> %o', basePath, router.path)
    //     const data = {
    //       path: path.resolve(basePath, router.path),
    //       title: [...prefixTitle]
    //     }

    //     if (router.meta && router.meta.title) {
    //       data.title = [...data.title, router.meta.title]

    //       if (router.redirect !== 'noRedirect') {
    //         // only push the routes with title
    //         // special case: need to exclude parent router without redirect
    //         res.push(data)
    //       }
    //     }

    //     // recursive child routes
    //     if (router.children) {
    //       const tempRoutes = this.generateRoutes(router.children, data.path, data.title)
    //       if (tempRoutes.length >= 1) {
    //         res = [...res, ...tempRoutes]
    //       }
    //     }
    //   }
    //   return res
    // },
    querySearch(query) {
      if (query !== '') {
        this.options = this.fuse.search(query)
      } else {
        this.options = []
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.header-search {
  font-size: 0 !important;

  .search-icon {
    cursor: pointer;
    font-size: 18px;
    vertical-align: middle;
  }

  .header-search-select {
    font-size: 18px;
    transition: width 0.2s;
    width: 0;
    overflow: hidden;
    background: transparent;
    border-radius: 0;
    display: inline-block;
    vertical-align: middle;

    /deep/ .el-input__inner {
      border-radius: 0;
      border: 0;
      padding-left: 0;
      padding-right: 0;
      box-shadow: none !important;
      border-bottom: 1px solid #d9d9d9;
      vertical-align: middle;
    }
  }

  &.show {
    .header-search-select {
      width: 210px;
      margin-left: 10px;
    }
  }
}
</style>
