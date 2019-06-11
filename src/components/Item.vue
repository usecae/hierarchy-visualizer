<template>
  <li>
    <div class="item-hover-target" v-bind:class="{ 'item-hovered': isHovered, 'item-focused': isFocused }"
      @click="onClick" @mouseover="mouseOver" @mouseout="mouseOut" />
    <div class="item-row">
      <caret v-if="isFolder" class="item-content" v-bind:class="{ 'caret-open': isOpen }"
        @click="onClick" @mouseover="mouseOver" @mouseout="mouseOut" />
      <caret v-if="!isFolder" style="visibility:hidden" />
      <folder-icon v-if="isFolder" class="item-content"
        @click="onClick" @mouseover="mouseOver" @mouseout="mouseOut" />
      <span class="item-content" @click="onClick" @mouseover="mouseOver" @mouseout="mouseOut">{{name}}</span>
      <!-- <span v-if="!isFolder" v-for="itemValue in children" :key="itemValue" class="item-value item-content"
        @mouseover="mouseOver" @mouseout="mouseOut">
        {{filterComment(itemValue)}}
      </span> -->
    </div>
    <ul v-if="isFolder" v-show="isOpen">
      <item v-for="(childValue, childKey) in children" :key="childKey"
        :keyString="childKey" :children="childValue" :ignoreSigns="ignoreSigns"
        :isDisplayed="isOpen && isDisplayed" @item-clicked="itemClicked" />
    </ul>
  </li>
</template>

<script>
import Caret from "./Caret";
import FolderIcon from "./FolderIcon";

const dirPrefix = '@dir_';
export default {
  name: 'item',
  components: {
    Caret,
    FolderIcon
  },
  data: function() {
    return {
      isOpen: false,
      isFocused: false,
      isHovered: false
    }
  },
  props: {
    keyString: String,
    children: Object|Array,
    ignoreSigns: Array,
    isDisplayed: Boolean
  },
  computed: {
    isFolder: function() {
      return this.keyString.includes(dirPrefix);
    },
    name: function() {
      return this.keyString.includes(dirPrefix) ? this.keyString.slice(dirPrefix.length) : this.keyString;
    }
  },
  beforeCreate: function() {
    this.$root.$children[0].items.push(this);
  },
  methods: {
    onClick: function() {
      this.$emit("item-clicked", this);
      if (this.isFolder) {
        this.isOpen = !this.isOpen;
      }
    },
    itemClicked: function(target) {
      this.$emit("item-clicked", target);
    },
    mouseOver: function() {
      this.isHovered = true;
    },
    mouseOut: function() {
      this.isHovered = false;
    },
    filterComment: function(itemValue) {
      let ignoreIndex = -1;
      for (let i = 0; i < this.ignoreSigns.length; i++) {
        const ignoreSign = this.ignoreSigns[i];
        const temp = itemValue.indexOf(ignoreSign);
        if (temp >= 0 && (ignoreIndex < 0 || temp < ignoreIndex)) {
          ignoreIndex = temp;
        }
      }
      return ignoreIndex >= 0 ? itemValue.slice(0, ignoreIndex) : itemValue;
    }
  }
};
</script>

<style>
.item-hover-target {
  position: absolute;
  left: 1px;
  right: 0px;
  height: 20px;
  line-height: 20px;
  cursor: pointer;
}
.item-hovered {
  background-color: slateblue;
}
.item-focused {
  background-color: firebrick;
}
.item-row {
  display: flex;
  align-items: center;
  height: 20px;
}
.item-content {
  position: relative;
  cursor: pointer;
  font-size: 14px;
}
.item-value {
  font-weight: lighter;
  color: gray;
  padding-left: 6px;
}
</style>
