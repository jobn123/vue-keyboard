<template>
  <div class="keyboard" v-clickoutside="closeModal">
    <p v-for="keys in keyList" :key="keys.index">
      <template v-for="key in keys">
        <i v-if="key === 'top'" @click.stop="clickKey" @touchend.stop="clickKey" :class="capsactive ?'tab-top_active' : 'tab-top'" :key="key"></i>
        <i v-else-if="key === '123'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-num" :key="key">123</i>
        <i v-else-if="key === 'en'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-en" :key="key">英文</i>
        <i v-else-if="key === 'del'" @click.stop="clickKey" @touchend.stop="clickKey" class="key-delete" :key="key"></i>
        <i v-else-if="key === 'blank'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-blank" :key="key">space</i>
        <i v-else-if="key === 'symbol'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-symbol" :key="key">#+=</i>
        <!-- <i v-else-if="key === 'point'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-point" :key="key">·</i> -->
        <i v-else-if="key === 'enter'" @click.stop="clickKey" @touchend.stop="clickKey" class="tab-enter" :key="key">确定</i>
        <i v-else @click.stop="clickKey" @touchend.stop="clickKey" :key="key" class="keynormal" :id="key">{{ key }}</i>
      </template>
    </p>
  </div>
</template>

<script>
import Vue from 'vue'
Vue.directive('clickoutside', {
  bind (el, binding, vnode) {
    function documentHandler (e) {
      if (el.contains(e.target)) {
        return false
      }
      if (binding.expression) {
        binding.value(e)
      }
    }
    el.__vueClickOutside__ = documentHandler
    document.addEventListener('click', documentHandler)
  },
  update (el, binding, vnode) {},
  unbind (el, binding) {
    document.removeEventListener('click', el.__vueClickOutside__)
    delete el.__vueClickOutside__
  }
})

export default {
  data () {
    return {
      keyList: [],
      status: 0, // 0 小写 1 大写 2 数字 3 符号
      capsactive: false,
      lowercase: [
        ['q', 'w', 'e', 'r', 't', 'y', 'u', 'i', 'o', 'p'],
        ['a', 's', 'd', 'f', 'g', 'h', 'j', 'k', 'l'],
        ['top', 'z', 'x', 'c', 'v', 'b', 'n', 'm', 'del'],
        ['123', 'blank', 'enter']
      ],
      uppercase: [
        ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
        ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
        ['top', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', 'del'],
        ['123', 'blank', 'enter']
      ],
      numcase: [
        ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0'],
        ['-', '/', ':', ';', '(', ')', '@', '“', '”'],
        ['symbol', '。', ',', '--', '?', '!', '.', '~', 'del'],
        ['en', 'blank', 'enter']
      ],
      symbolcase: [
        ['￥', '【', '】', '{', '}', '#', '%', '^', '*', '+'],
        ['_', '=', '\\', '|', '~', '《', '》', '$', '&'],
        ['123', '...', '，', '、', '！', '·', '~', 'del'],
        ['en', 'blank', 'enter']
      ],
      // equip:!!navigator.userAgent.toLocaleLowerCase().match(/ipad|mobile/i)//是否是移动设备
      equip: true
    }
  },
  props: {
    option: {
      type: Object
    }
  },
  mounted () {
    this.keyList = this.lowercase
  },
  methods: {
    tabHandle ({ value = '' }) {
      if (value.indexOf('tab-num') > -1) {
        this.status = 2
        this.keyList = this.numcase
        // 数字键盘数据
      } else if (value.indexOf('key-delete') > -1) {
        this.emitValue('delete')
      } else if (value.indexOf('tab-blank') > -1) {
        this.emitValue(' ')
      } else if (value.indexOf('tab-enter') > -1) {
        this.emitValue('\n')
      } else if (value.indexOf('tab-point') > -1) {
        this.emitValue('.')
      } else if (value.indexOf('tab-symbol') > -1) {
        this.status = 3
        this.keyList = this.symbolcase
      } else if (value.indexOf('tab-en') > -1) {
        this.status = 0
        this.keyList = this.lowercase
      } else if (value.indexOf('tab-top') > -1) {
        if (this.status === 0) {
          this.status = 1
          this.keyList = this.uppercase
          this.capsactive = true
        } else {
          this.status = 0
          this.keyList = this.lowercase
          this.capsactive = false
        }
      }
    },
    clickKey (event) {
      if (event.type === 'click' && this.equip) return
      let value = event.srcElement.innerText
      console.log(value)
      value && value !== '英文' && value !== '#+=' && value !== '123' && value !== '确定' && value !== 'space' ? this.emitValue(value) : this.tabHandle(event.srcElement.classList)
    },
    emitValue (key) {
      this.$emit('keyVal', key)
    },
    closeModal (e) {
      if (e.target !== this.option.sourceDom) {
        this.$emit('close', false)
      }
    }
  }
}
</script>
<style scoped lang="scss">
@mixin caps($url) {
  @include size(.88rem, .88rem);
  background: url($url);
  background-size: 0.88rem 0.88rem;
}
@mixin size($w, $h) {
  width: $w;
  height: $h;
}
@mixin bg($url, $w, $h) {
  background: url($url);
  background-size: $w $h;
}
.keyboard {
  @include size(7.5rem, 4.14rem);
  background: url('../assets/keyboardbg.png');
  font-size: 0.4rem;
  padding-top: 0.5em;
  color: #F9F9F9;
  user-select: none;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 999;
  pointer-events: auto;
  .keynormal {
    position: relative;
    &:active {
      &::before {
        display: block;
        content: "" attr(id) "";
        position: absolute;
        top: -1.12rem;
        left: -.3rem;
        line-height: 1rem;
        text-align: center;
        @include size(1.16rem, 1rem);
        @include bg('../assets/currentkeyborder.png', 1.16rem, 1rem);
      }
    }
  }
  p {
    width: 100%;
    margin: 0 auto;
    height: 0.8rem;
    margin-bottom: 0.24rem;
    i {
      @include bg('../assets/keyborder.png', .6rem, .88rem);
      @include size(.6rem, .88rem);
      border-radius: 0.04rem;
      display: inline-block;
      margin-left: .1rem;
      line-height: 0.88rem;
      font-style: normal;
      font-size: 0.4rem;
      text-align: center;
      vertical-align: bottom;
    }
    .tab-top {
      @include caps('../assets/keyboard-caps.png');
    }
    .tab-top_active {
      @include caps('../assets/keyboard-caps_active.png');
    }
    .tab-symbol {
      @include caps('../assets/keynull.png');
    }
    .key-delete {
      @include caps('../assets/keyboard-del.png');
    }
    .tab-num, .tab-en {
      @include size(1.28rem, .88rem);
      @include bg('../assets/keymediumborder.png', 1.28rem, .88rem);
      color: #F9F9F9;
      font-size: .32rem;
    }
    .tab-point {
      width: 20px;
    }
    .tab-blank {
      @include size(4.14rem, .88rem);
      @include bg('../assets/spaceborder.png', 4.14rem, .88rem);
      font-size: .32rem;
      line-height: .88rem;
      text-align: center;
      color: #F9F9F9;
      border-radius: .08rem;
    }
    .tab-enter {
      @include size(1.28rem, .88rem);
      @include bg('../assets/keymediumborder.png', 1.28rem, .88rem);
      margin-left: .08rem;
      line-height: .88rem;
      text-align: center;
      font-size: .32rem;
    }
    &:nth-child(2) {
      width: 90%;
    }
  }
}
</style>
