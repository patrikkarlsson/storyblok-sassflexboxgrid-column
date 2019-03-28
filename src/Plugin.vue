<template>
  <div class="classlist">
      <div class="add">
        <select v-model="currentType">
          <option :value="type" v-for="(type, index) in types" :key="type + index" :checked="currentType == type">{{type}}</option>
        </select>
        <span>-</span>
        <select v-model="currentWidth">
          <option v-if="showEmptyWidth()"></option>
          <option :value="width" v-for="(width, index) in widths" :key="width + index" :checked="currentWidth == width">{{width}}</option>
        </select>
        <div v-if="showColumns()" >
          <span>-</span>
          <select v-model="currentColumn">
            <option v-if="showEmptyWidth()"></option>
            <option :value="column" v-for="(column, index) in columns" :key="column + index" :checked="currentColumn == column">{{column}}</option>
          </select>
        </div>
        <button class="uk-button uk-button-primary" @click="add()">+</button>
      </div>
      <span class="list__header">Classnames</span>
      <ul class="list">
        <li class="list__item" v-for="(classname, index) in model.classnames" :key="'classname' + index"><span>{{classname}}</span><button class="uk-button uk-button-secondary uk-button-small" @click="remove(index)"><i class="uk-icon-trash"></i></button></li>
      </ul>
  </div>
</template>

<script>
const TYPE = {
  col: 'col',
  offset: 'offset',
  first: 'first',
  last: 'last'
}
export default {
  mixins: [window.Storyblok.plugin],
  data: () => ({
    currentColumn: null,
    currentWidth: null,
    currentType: TYPE.col,
  }),
  methods: {
    initWith() {
      return {
        plugin: 'sassflexboxgrid-column',
        classnames: []
      }
    },
    showEmptyWidth() {
      return [TYPE.col, TYPE.first, TYPE.last].includes(this.currentType)
    },
    showColumns() {
      if(this.currentType == TYPE.col && this.currentWidth || this.currentType == TYPE.offset) {
        return true
      }
      this.currentColumn = null
      return false;
    },
    pluginCreated() {
      this.types = [TYPE.col, TYPE.offset, TYPE.first, TYPE.last]
      this.widths = ['xs', 'sm', 'md', 'lg', 'xl']
      this.columns = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
    },
    remove(index) {
      this.model.classnames.splice(index, 1)
    },
    search(classname) {
      const splittedClassname = classname.split('-')
      let searchTerm = classname
      let position = -1

      if(splittedClassname.length == 3 || splittedClassname.length == 4) {
        searchTerm = splittedClassname.slice(0, splittedClassname.length - 1).join('-')
      }

      this.model.classnames.some((item, index) => {
        const foundMatch = item.includes(searchTerm)
        if(foundMatch) {
          position = index
          return true
        }
        return false
      })

      return position

    },
    add() {
      let currentClassName = this.currentType;
      
      if(this.currentType == TYPE.offset) {
        currentClassName = `col-${this.currentWidth}-offset-${this.currentColumn}`;  
      } else {
        if(this.currentWidth) {
          currentClassName += `-${this.currentWidth}`
        }
        if(this.currentColumn) {
          currentClassName += `-${this.currentColumn}`
        }
      }

      const searchIndex = this.search(currentClassName)

      if(searchIndex != -1) {
        this.model.classnames.splice(searchIndex, 1, currentClassName)
      } else {
        this.model.classnames.push(currentClassName)
      }
    }
  },
  watch: {
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
      },
      deep: true
    },
    currentType() {
      this.currentWidth = this.showEmptyWidth() ? null : 'xs';
      this.currentColumn = (this.currentType == TYPE.offset && !this.showEmptyWidth()) ? 1 : null;
    }
  }
}
</script>

<style>
  .add {
    display: flex;
    align-items: center;
    flex: 1;
    margin-bottom: 10px;
  }
  
  .add span {
    display: inline-block;
    margin: 0 5px;
  }

  .add div {
    display: flex;
    align-items: center;
  }

  .uk-form .add select {
    padding: 0 25px 0 15px !important;
  }

  .uk-form .add button {
    margin-left: 5px;
  }

  .uk-form .list {
    padding: 0;
    margin: 0;
  }

  .uk-form .list .list__item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 10px;
  }
</style>
