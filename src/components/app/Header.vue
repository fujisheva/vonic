<template>
  <div von-navbar="active" class="bar bar-header bar-transparent" :class="{'cached': cached}">
    <div class="buttons" :class="{'hide': cached}" v-if="showBack">
      <button class="button button-icon" @click="onBackClick()">
        <span v-html="backText">
        </span>
      </button>
    </div>

    <h1 class="title">
      <span v-text="title"></span>
    </h1>

    <div class="buttons" :class="{'hide': cached}" v-if="showMenu">
      <button class="button button-icon" @click="onMenuClick()">
        <span v-html="menuText">
        </span>
      </button>
    </div>
  </div>
</template>
<style lang="scss" scoped>
  .bar.bar-header {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.15);
    z-index: 2;
    padding: 5px 0px;

    &.bar-transparent {
      background: transparent;
    }

    &.cached {
      box-shadow: none;
      z-index: 1;
    }

    .title {
      -webkit-transition-property: opacity, -webkit-transform;
      transition-property: opacity, transform;
      opacity: 0;
    }

    .hide {
      display: none;
    }
  }
</style>
<script>
  import channel from './channel'

  const TITLE_TRANSITION = () => (document.body.classList.contains('grade-a') && !window.__disable_nav_title_transition__) ?
    '500ms cubic-bezier(.36, .66, .04, 1)' : '0ms'

  const DEFAULT_BACK_TEXT = '<i class="icon ion-ios-arrow-back"></i>'
  const DEFAULT_MENU_TEXT = '<i class="icon ion-navicon"></i>'

  export default {
    props: {
      title: String,
      showBack: {
        type: Boolean,
        default: false
      },
      backText: {
        type: String,
        default: DEFAULT_BACK_TEXT
      },
      onBack: Function,
      showMenu: {
        type: Boolean,
        default: false
      },
      menuText: {
        type: String,
        default: DEFAULT_MENU_TEXT
      },
      onMenu: Function,
      enableTitleTransition: Boolean,
      renderCount: Number
    },

    data() {
      return {
        cached: false
      }
    },

    mounted() {
      this.titleEnter()
    },

    destroyed() {
      let navContainer = document.querySelector('[von-nav] .navbar-container')
      let cached = document.querySelectorAll('[von-navbar="cached"]')

      let i = 0
      while (i < cached.length) {
        navContainer.removeChild(cached[i])
        i++
      }
    },

    methods: {
      onBackClick() {
        if (this.onBack) {
          this.onBack()
          return
        }

        if (window.__push_method__ === 'push') {
          let root = document.querySelector('[von-app]')
          if (root) root.setAttribute('transition-direction', 'back');
          history.go(-1)
        }
      },

      onMenuClick() {
        if (this.onMenu) {
          this.onMenu()
        }
      },

      cache() {
        if (window.__disable_nav_title_transition__) {
          this.$destroy()
          return
        }

        this.cached = true
        this.titleLeave()
      },

      isCached() {
        return this.cached
      },

      titleEnter() {
        let container = this.$el
        let el = container.querySelector('.title')
        let text = container.querySelector('.title > span')

        let style = el.style
        style.webkitTransition
        style.transition = 'none'

        let dist = parseInt((el.offsetWidth - text.offsetWidth) / 2) + 'px'
        let direction = document.querySelector('[von-app]').getAttribute('transition-direction')
        if (direction) {
          style.webkitTransform =
          style.transform = 'translate3d(' + (direction == 'back' ? '-' : '') + dist + ',0,0)'
        }

        setTimeout(() => {
          style.opacity = 1
          style.webkitTransform =
          style.transform = 'translate3d(0,0,0)'

          style.webkitTransition
          style.transition = (direction && this.enableTitleTransition) ? TITLE_TRANSITION() : 'none'

          if (this.renderCount == 1) {
            this.fixNavbar()
          }

          if (this.renderCount == 2) {
            el.addEventListener('transitionEnd', () => {}, false)
            el.addEventListener('webkitTransitionEnd', this._titleEnterTransitionEnd, false)
          }
        }, 0)
      },

      titleLeave() {
        let container = this.$el
        let el = this.$el.querySelector('.title')
        let text = container.querySelector('.title > span')

        let style = el.style
        style.webkitTransition
        style.transition = 'none'

        setTimeout(() => {
          let dist = parseInt((el.offsetWidth - text.offsetWidth) / 2) + 'px'
          let direction = document.querySelector('[von-app]').getAttribute('transition-direction')
          if (direction) {
            style.webkitTransform =
            style.transform = 'translate3d(' + (direction == 'back' ? '' : '-') + dist + ',0,0)'
          }
          style.opacity = 0

          style.webkitTransition
          style.transition = direction && (this.renderCount == 1 || this.enableTitleTransition) ? TITLE_TRANSITION() : 'none'

          el.addEventListener('transitionEnd', () => {}, false)
          el.addEventListener('webkitTransitionEnd', this._titleLeaveTransitionEnd, false)
        }, 0)
      },

      _titleEnterTransitionEnd(e) {
        this.fixNavbar()
        e.target.removeEventListener('webkitTransitionEnd', this._titleEnterTransitionEnd)
      },

      _titleLeaveTransitionEnd(e) {
        this.$destroy()
        this.fixNavbar()
        e.target.removeEventListener('webkitTransitionEnd', this._titleLeaveTransitionEnd)
      },

      fixNavbar() {
        setTimeout(() => {
          document.querySelector('[von-nav]').style.position = 'fixed'
        }, 50)
      }
    }
  }
</script>
