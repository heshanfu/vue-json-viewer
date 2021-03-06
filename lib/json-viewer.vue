<template>
  <div :class="jvClass">
    <div class="jv-tooltip" v-if="copyable">
      <span class="jv-button" ref="clip" @click="clip" :class="{copied}">{{ copied ? 'copied!' : 'copy' }}</span>
    </div>
    <div class="jv-code" :class="{'open': expandCode}">
      <json-box
        ref="jsonBox"
        :value="value"
        :key-name="keyName"
        :sort="sort"></json-box>
    </div>
    <div v-if="expandableCode" class="jv-more" @click="toggleExpandCode">
      <span class="jv-toggle" :class="{open: !!expandCode}"></span>
    </div>
</div>
</template>

<script>
import Vue from 'vue'
import JsonBox from './json-box'
import Clipboard from 'clipboard'

Vue.component(JsonBox.name, JsonBox)

export default {
  name: 'JsonViewer',
  props: {
    value: [Object, Array, String, Number, Boolean],
    keyName: String,
    expandDepth: {
      type: Number,
      default: 1
    },
    copyable: {
      type: Boolean,
      default: false
    },
    sort: {
      type: Boolean,
      default: false
    },
    boxed: {
      type: Boolean,
      default: false
    },
    theme: {
      type: String,
      default: 'jv-light'
    },
  },
  provide() {
    return {
      expandDepth: this.expandDepth,
    }
  },
  data() {
    return {
      copied: false,
      expandableCode: false,
      expandCode: false
    }
  },
  mounted: function () {
    this.onResized()
    this.$el.addEventListener("resized", this.onResized, true)
  },
  computed: {
    jvClass() {
      return 'jv-container ' + this.theme + (this.boxed ? ' boxed' : '')
    }
  },
  methods: {
    onResized() {
      this.$nextTick(() => {
        if (this.$refs.jsonBox.$el.clientHeight >= 250) {
          this.expandableCode = true
        } else {
          this.expandableCode = false
        }
      })
    },
    clip() {
      if (this.copied) {
        return
      }

      const clipBoard = new Clipboard(this.$refs.clip, {
        text: () => {
          return JSON.stringify(this.value, null, 2)
        }
      })

      clipBoard.on('success', () => {
        this.copied = true
        setTimeout(() => {
          this.copied = false
        }, 2000)
        this.$emit('copied')
        clipBoard.destroy()
      })
    },
    toggleExpandCode() {
      this.expandCode = !this.expandCode
    }
  },
  components: {
    JsonBox
  }
}
</script>

<style lang="scss">
.jv-container {
  box-sizing: border-box;
  position: relative;

  &.boxed {
    border: 1px solid #eee;
    border-radius: 6px;

    &:hover {
      box-shadow: 0 2px 7px rgba(0,0,0,.15);
      border-color: transparent;
      position: relative
    }
  }

  &.jv-light {
    background: #fff;
    white-space: nowrap;
    color: #525252;
    font-size: 14px;
    font-family: Consolas, Menlo, Courier, monospace;

    .jv-ellipsis {
      color: #999;
      background-color: #eee;
      display: inline-block;
      line-height: 0.9;
      font-size: 0.9em;
      padding: 0px 4px 2px 4px;
      border-radius: 3px;
      vertical-align: 2px;
      cursor: pointer;
      user-select: none;
    }
    .jv-button { color: #49b3ff }
    .jv-key { color: #111111 }
    .jv-item {
      &.jv-array { color: #111111 }
      &.jv-boolean { color: #fc1e70 }
      &.jv-function { color: #067bca }
      &.jv-number { color: #fc1e70 }
      &.jv-object { color: #111111 }
      &.jv-undefined { color: #e08331 }
      &.jv-string {
        color: #42b983;
        word-break: break-word;
        white-space: normal;
      }
    }
    .jv-code {
      .jv-toggle {
        &:before {
          padding: 0px 2px;
          border-radius: 2px;
        }
        &:hover {
          &:before {
            background: #eee;
          }
        }
      }
    }
  }

  .jv-code {
    max-height: 300px;
    overflow: hidden;
    padding: 20px;

    &.open {
      max-height: initial;
      overflow: visible;
      overflow-x: auto;
      padding-bottom: 45px;
    }
  }

  .jv-toggle {
    cursor: pointer;

    &:before {
      content: "⏷";
      padding: 0px 2px;
      border-radius: 2px;
      position: absolute;
    }
    &:after {
      content: " ";
      position: relative;
      display: inline-block;
      width: 8px;
    }

    &.open {
      &:before {
        content: "⏶";
      }
    }
  }

  .jv-more {
    position: absolute;
    z-index: 1;
    bottom: 0;
    left: 0;
    right: 0;
    height: 40px;
    width: 100%;
    text-align: center;
    cursor: pointer;

    .jv-toggle {
      position: relative;
      top:40%;
      z-index: 2;
      color: #888;
      transition: all 0.1s;
      // background: red;
    }

    &:after {
      content: "";
      width: 100%;
      height: 100%;
      position: absolute;
      bottom: 0;
      left: 0;
      z-index: 1;
      background: linear-gradient(to bottom, rgba(0,0,0,0) 20%, rgba(230, 230, 230, 1) 100%);
      transition: all 0.1s;
    }

    &:hover {
      .jv-toggle {
        top:50%;
        color: #111;
      }

      &:after {
      background: linear-gradient(to bottom, rgba(0,0,0,0) 20%, rgba(230, 230, 230, 0.3) 100%);
      }
    }
  }

  .jv-button {
    position: relative;
    cursor: pointer;
    display: inline-block;
    padding: 5px;
    z-index: 5;

    &.copied {
      opacity: 0.4;
      cursor: default;
    }
  }

  .jv-tooltip {
    position: absolute;
    right: 15px;
    top: 10px;
  }

  .j-icon {
    font-size: 12px;
  }
}
</style>
