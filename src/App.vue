<template>
  <div id="hierarchy-visualizer">
    <header id="header-area">
      <div>Hierarchy Visualizer</div>
    </header>
    <div id="main-area">
      <div id="left-area">
        <div>
          <text-reader @load="text=$event" />
        </div>
        <textarea id="textarea" v-model="text" />
      </div>
      <div id="center-area">
        <div id="ignore-root">
          <label>
            Ignore
          </label>
          <div id="ignore-options">
            <div>
              <input id="ignore-hash" ref="ignoreHash" type="checkbox" checked="true" value="#" />
              <label for="ignore-hash">
                #
              </label>
            </div>
            <div>
              <input id="ignore-double-slash" type="checkbox" value="//" />
              <label for="ignore-double-slash">
                //
              </label>
            </div>
          </div>
        </div>
        <div>
          <label for="delimiter-input">
            Delimiter
          </label>
          <input id="delimiter-input" ref="delimiter" type="text" value="." size="5" />
        </div>
        <div>
          <label for="assignment-operator-input">
            Assignment Operator
          </label>
          <input id="assignment-operator-input" ref="assingmentOperator" type="text" value="=" size="5" />
        </div>
        <button id="generate-button" @click="generateTreeView">Generate Tree View</button>
      </div>
      <div id="right-area">
        <div>
        </div>
        <div id="tree-view-area">
          <ul>
            <item v-for="(children, key) in tree" :key="key"
                  :keyString="key" :children="children" :ignoreSigns="ignoreSigns"
                  :isDisplayed="true" @item-clicked="itemClicked" />
          </ul>
        </div>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
import TextReader from "./components/TextReader"
import Item from "./components/Item"

export default {
  name: 'app',
  data () {
    return {
      text: "Paste the delimiter separated text here!",
      ignoreSigns: [],
      tree: {},
      focusedItem: null,
      items: []
    }
  },
  components: {
    TextReader,
    Item
  },
  methods: {
    initialize() {
      this.items = [];
    },
    generateTreeView() {
      this.initialize();

      const lines = this.text.split('\n');
      const delimiter = this.$refs.delimiter.value;
      const assingmentOperator = this.$refs.assingmentOperator.value;
      if (this.$refs.ignoreHash.checked) {
        this.ignoreSigns.push(this.$refs.ignoreHash.value);
      }
      const filteredLines = [];
      for (let i = 0; i < lines.length; i++) {
        const line = lines[i];
        if (!this.ignoreSigns.includes(line.charAt(0)) && line.length > 0) {
          filteredLines.push(line);
        }
      }
      const tree = {};
      for (let i = 0; i < filteredLines.length; i++) {
        const line = filteredLines[i];
        if (!line.includes(assingmentOperator)) {
          continue;
        }
        const keyValue = line.split(assingmentOperator);
        const key = keyValue[0];
        let value = keyValue[1];
        // In case assingmentOperator exists in value.
        for (let j = 2; j < keyValue.length; j++) {
          value = value.concat(keyValue[j]);
        }
        const pieces = key.split(delimiter);
        let curNode = tree;
        for (let k = 0; k < pieces.length - 1; k++) {
          const piece = pieces[k];
          const dirName = '@dir_'.concat(piece);
          if (!curNode[dirName]) {
            curNode[dirName] = {};
          }
          curNode = curNode[dirName];
        }
        const end = pieces[pieces.length - 1];
        if (!curNode[end]) {
          curNode[end] = [];
        }
        curNode[end].push(value);
      }
      this.tree = tree;

      const arrowKeys = ['ArrowUp', 'ArrowRight', 'ArrowDown', 'ArrowLeft'];
      window.addEventListener('keydown', function(e) {
        if (arrowKeys.includes(e.key)) {
          this.onArrowKey(e);
        }
      }.bind(this));
    },
    itemClicked(target) {
      if (this.focusedItem) {
        this.focusedItem.isFocused = false;
      }
      target.isFocused = true;
      this.focusedItem = target;
    },
    onArrowKey(e) {
      if (!this.focusedItem) {
        return;
      }
      e.preventDefault();
      let index = this.items.indexOf(this.focusedItem);
      switch (e.key) {
        case 'ArrowRight':
          if (this.focusedItem.isFolder) {
            if (this.focusedItem.isOpen) {
              this.itemClicked(this.items[index + 1]);
            } else {
              this.focusedItem.isOpen = true;
            }
          }
          break;
        case 'ArrowLeft':
          if (this.focusedItem.isFolder && this.focusedItem.isOpen) {
            this.focusedItem.isOpen = false;
          } else if (this.focusedItem.$parent.$parent.$parent) {
            this.itemClicked(this.focusedItem.$parent);
          }
          break;
        case 'ArrowUp':
          while (index > 0) {
            index--;
            const nextItem = this.items[index];
            if (nextItem.isDisplayed) {
              this.itemClicked(nextItem);
              break;
            }
          }
          break;
        case 'ArrowDown':
          while (index < this.items.length - 1) {
            index++;
            const nextItem = this.items[index];
            if (nextItem.isDisplayed) {
              this.itemClicked(nextItem);
              break;
            }
          }
          break;
        default:
          return;
      }
    }
  }
}
</script>

<style>
  body {
    margin: 0;
  }
  #hierarchy-visualizer {
    min-height: 100vh;
    background-color: #343233;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;

    display: grid;
    grid-template-rows: 40px 1fr;
  }
  #header-area {
    background-color: #584b4f;
    color: #efeae1;
    font-size: 20px;
    padding-left: 20px;

    display: grid;
    align-items: center;
  }
  #main-area {
    color: #efdab9;

    display: grid;
    grid-template-columns: 1fr 15% 1fr;
  }
  #left-area {
    padding: 10px 20px 20px;

    display: grid;
    grid-template-rows: 40px 1fr;
  }
  #textarea {
    width: 100%;
    border-radius: 4px;
  }
  #right-area {
    padding: 10px 20px 20px;
    display: grid;
    grid-template-rows: 40px 1fr;
    max-height: calc(100vh - 40px);
    box-sizing: border-box;
  }
  #tree-view-area {
    overflow-y: scroll;
    position: relative;
    background-color: #3d3a39;
    border-width: 1px;
    border-radius: 4px;
    border-style: solid;
    color: #efdab9;
    fill: #efdab9;
  }
  #tree-view-area ul {
    padding-left: 20px;
    list-style-type: none;
  }
  #center-area {
    display: grid;
    grid-gap: 10px;
    align-content: center;
    justify-content: center;
  }
  #generate-button {
    border-radius: 5px;
    padding: 4px 12px;
    cursor: pointer;
    background-color: #ffd152;
    color: #343233;
    font-weight: 500;
  }
  #ignore-root {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
  #ignore-options {
    display: grid;
    grid-template-rows: 1fr 1fr;
  }
</style>
