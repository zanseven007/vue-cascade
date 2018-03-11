<template>
  <div :class="cascadeClasses">
    <div class="cascade__mask" v-show="visible" @click="close()"></div>
    <div class="cascade__content" v-show="visible">
      <div class="cascade__header">
        <div class="cascade__back" @click="back()">
          <i class="iconfont icon-back"></i>
        </div>
        <div class="cascade__title">
          {{titleName}}
        </div>
      </div>
      <div class="cascade__search" v-if="filterable">
        <input
          type="text"
          class="cascade__search--input"
          v-model="query"
          placeholder="请输入您想搜索的内容"
          autocomplete="off"
          spellcheck="false"
          ref="input"
          v-focus="autoFocus"
        >
        <div class="cascade__search--comfirm" v-show="showComfirm" @click="comfirm">
          确认
        </div>
      </div>
      <div class="cascade__list">
        <div class="cascade__item" v-if="isMatch(item.label)" :key="item.key" v-for="item in casData" @click= "selectItem(item,casData)">
            <p>{{item.label}}</p>
        </div>
        <div class="cascade__loading" v-show="remote && loading">
          <img src="../assets/loading.gif" alt="loading">
        </div>
        <div class="cascade__noSearchResultTip" v-show="showComfirm && query">
          没有找到相关内容,点击确定使用自定义内容~
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'v-cascade',
  props: {
    title: {
      type: Array
    },
    value: {
      type: Boolean,
      default: false
    },
    addClass: {
      type: String
    },
    data: {
      type: Array,
      default: () => []
    },
    autoFocus: {
      type: Boolean,
      default: true
    },
    filterable: {
      type: Boolean,
      default: false
    },
    remote: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    },
    remoteMethod: {
      type: Function
    }
  },
  data () {
    return {
      level: 1,
      casData: this.data,
      casTitle: this.title,
      visible: this.value,
      path: [],
      pathName: [],
      query: ''
    }
  },
  methods: {
    selectItem (item, casData) {
      // 删除上一次的选择
      this.path.length = this.level - 1
      this.pathName.length = this.level - 1

      this.path.push(item.key)
      this.pathName.push(item.label)

      if (item.children) {
        this.query = ''
        this[`data_${this.level}`] = casData
        this.level++
        this.casData = item.children
      } else {
        this.$emit('input', false)
        this.$emit('success', { path: this.path, pathName: this.pathName })
      }
    },
    comfirm () {
      // 删除上一次的选择
      this.path.length = this.level - 1
      this.pathName.length = this.level - 1

      this.path.push('-1')
      this.pathName.push(this.query)
      this.$emit('input', false)
      this.$emit('success', { path: this.path, pathName: this.pathName })
    },
    close () {
      this.visible = false
      this.$emit('input', false)
      this.$emit('cancel')
    },
    back () {
      if (this.level === 1) {
        this.close()
        return false
      }
      this.level--
      this.casData = this[`data_${this.level}`]
    },
    isMatch (name) {
      if (!this.filterable || this.remote) {
        return true
      } else if (name.indexOf(this.query) > -1) {
        return true
      } else {
        return false
      }
    }
  },
  created () {},
  computed: {
    titleName () {
      return this.title[this.level - 1] ? this.title[this.level - 1] : '请选择'
    },
    cascadeClasses () {
      return [
        'cascade',
        {
          [this.addClass]: this.addClass,
          'filterable': this.filterable
        }
      ]
    },
    showComfirm () {
      if (!this.remote) {
        return this.casData.find((item) => item.label.indexOf(this.query) > -1) === undefined
      }
    }
  },
  watch: {
    value (val) {
      this.visible = val
      if (val) {
        if (this.remote && !this.casData.length) {
          this.remoteData(this.query)
        }
      }
    },
    data (val) {
      this.casData = val
    },
    loading (val) {
      if (val) {
        this.casData = []
      }
    },
    query (val) {
      if (this.remote && this.remoteMethod) {
        this.remoteMethod(val)
        this.$emit('on-query-change', val)
      }
    }
  },
  directives: {
    focus: {
      inserted: function (el, binding) {
        if (binding.value) {
          el.focus()
        }
      },
      componentUpdated: function (el, binding) {
        if (binding.value) {
          el.focus()
        }
      }
    }
  },
  components: {}
}
</script>

<style lang="scss">
.cascade {
  &__mask {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.3);
  }
  &__content {
    width: 80%;
    max-width: 350px;
    position: fixed;
    top: 0;
    right: 0;
    height: 100%;
    background: #fff;
    border-left: 1px solid #efefef;
  }
  &__header {
    display: flex;
    height: 40px;
    align-items: center;
    border-bottom: 1px dashed #efefef;
    .icon-back {
      font-size: 22px;
      cursor: pointer;
    }
  }
  &__title {
    width: 100%;
    display: flex;
    justify-content: center;
    text-indent: -20px;
    font-weight: bold;
  }
  &__search {
    height: 40px;
    padding: 0 10px;
    display: flex;
    &--input {
      border: none;
      width: 100%;
      text-align: center;
      -webkit-appearance: none;
      outline: none;
    }
    &--comfirm {
      position: absolute;
      width: 60px;
      height: 40px;
      background: #495060;
      right: 0;
      line-height: 40px;
      text-align: center;
      font-size: 13px;
      color: #fff;
      cursor: pointer;
      border-top-left-radius: 4px;
      border-bottom-left-radius: 4px;
      z-index: 9;
    }
  }
  &__list {
    height: calc(100vh - 41px);
    overflow-y: scroll;
    padding-right: 17px;
    overflow-x: hidden;
    box-sizing: content-box;
    width: 100%;
  }
  &__item {
    display: flex;
    justify-content: center;
    align-items: center;
    width: calc(106%);
    padding: 12px 20px;
    border-bottom: 1px solid #efefef;
    cursor: pointer;
    transition: all 0.15s ease;
    font-size: 14px;
    box-sizing: border-box;
    p {
      text-align: left;
      margin-left: -8px;
    }
    &:hover {
      background: #fafafa;
    }
  }
  &__loading {
    width: 50px;
    margin: 20px auto;
    img{
      max-width: 100%;
    }
  }
  &__noResultTip, &__noSearchResultTip{
    text-align: center;
    margin-top: 20px;
    font-size: 14px;
    color: #48505f;
  }
}
.filterable {
  .cascade__list {
    border-top: 1px solid #efefef;
    height: calc(100vh - 81px);
  }
}
</style>