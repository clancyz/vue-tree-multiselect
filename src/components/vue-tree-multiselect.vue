<template>
  <div class="vue-tree-multiselect-wrap" style="width:300px;">
    <button class="btn btn-default drop-down" @click="showDropDown = !showDropDown">
      <span class="selectedText">{{selectedText}}</span>
      <span class="caret"></span>
    </button>
    <div class="vue-tree-multiselect-item-wrap" v-show="showDropDown">
      <div class="vue-tree-multiselect-item" v-for="list in showData" track-by="$index">
        <ul>
          <template v-for="item in list" track-by="$index">
            <li v-show="item.parent === 'root' || item.parentActive" :class="{'active': item.active}" @mouseover="showNext(item)" @click="checkItem(item)">
              <label>
              <i class="check-item glyphicon" :class="{'glyphicon-unchecked':!item.selected && !item.partSelected, 'glyphicon-check': item.selected, 'glyphicon-edit': item.partSelected}"></i>
                <span class="item-text">{{item.text}}</span>
                <i class="glyphicon glyphicon-chevron-right"></i>
              </label>
            </li>
          </template>
        </ul>
      </div>
    </div>
  </div>
</template>
<style lang="less">
  ::-webkit-scrollbar {
      width: 8px;
      height: 8px
  }

  ::-webkit-scrollbar-thumb {
      box-shadow: inset 0 0 3px rgba(0,0,0,.5);
      border-radius: 4px;
      background: rgba(119,129,149,.8)
  }
  .vue-tree-multiselect-wrap {
    font-family: 'Microsoft Yahei';
    display: inline-block;
    width: 100%;
    .drop-down{
        width: 100%;
        display: block;
    }
    .vue-tree-multiselect-item-wrap{
      position: absolute;
      border: 1px solid #ddd;
      border-radius: 5px;
      z-index: 999;
      background: #fff;
      .vue-tree-multiselect-item{
        display: inline-block;
        float: left;
        min-width: 200px;
        height: 300px;
        overflow-y:scroll;
        ul{
          margin: 0;
          padding: 15px;
          li{
            list-style:none;
            padding:5px 5px 5px 8px;
            line-height: 1;
            &.active{
              background: #ddd;
            }
            label{
              cursor: pointer;
              .item-text{
              }
            }
          }
        }
      }
    }
  }
</style>

<script>
import clone from 'clone'

export default {
    props: {
        initData: {
            type: Object,
            default: {}
        },
        defaultSelectedData: {
            type: Array,
            default () {
                return []
            }
        },
        levelText: {
            type: String,
            required: true,
            default: ''
        },
        levelKey: {
            type: String,
            required: true,
            default: ''
        },
        keyName: {
            type: String,
            default: 'key'
        },
        valName: {
            type: String,
            default: 'text'
        },
        selectedText: {
            type: String,
            default: '请选择'
        },
        allSelectedText: {
            type: String,
            default: '全部'
        },
        allNonSelectedText: {
            type: String,
            default: '请选择'
        },
        disabled: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            showDropDown: false,
            showData: [],
            documentListener: null,
            firstInit: true
        }
    },
    computed: {
        arrData() {
            if (!this.initData) return [];
            return this.walkTree(this.initData[this.levelKeyArr[0]]);
        },
        level() {
            return this.levelKeyArr.length;
        },
        levelKeyArr() {
            return this.levelKey.split(',');
        },
        levelTextArr() {
            return this.levelText.split(',');
        },
        allSelected() {
            if (!this.showData[0]) return false;
            return (this.showData[0].every(v => v.selected)) ? true : false;
        },
        allNonSelected() {
            if (!this.showData[0]) return false;
            return (this.showData[0].every(v => !v.selected && !v.partSelected)) ? true : false;
        },
        outData() {
            let result = {};
            this.showData.forEach((v, index) => {
                result[this.levelKeyArr[index]] = v.filter((item) => {
                    return (item.selected || item.partSelected) && item.key.indexOf('all') === -1;
                }).map((item) => {
                    return item.key;
                }).join(',')
            })
            result.allSelected = this.allSelected;
            result.allNonSelected = this.allNonSelected;
            return result;
        }
    },
    watch: {
        'arrData': {
            handler(val) {
                this.showData = clone(val);
                if (this.firstInit) {
                    this.handleDefaultSelectedData(this.defaultSelectedData);
                    this.firstInit = false;
                }
            },
            deep: true
        }
    },
    methods: {
        checkProps() {

        },
        // basic data transform
        walkTree(data, index, parent, result) {
            if (!result) result = [];
            if (!data) return result;
            if (!index) index = 0;
            for (var i = 0, length = data.length; i < length; i++) {
                if (!result[index]) result[index] = [];
                let item = data[i];
                let tmp = {};
                tmp.key = item[this.keyName].toString();
                tmp.text = item[this.valName].toString();
                tmp.parentKey = parent === undefined ? 'root' : parent;
                tmp.parentActive = parent === undefined ? true : false;
                tmp.parentSelected = false;
                tmp.selected = false;
                tmp.partSelected = false;
                tmp.level = index;
                tmp.active = false;
                if (i === 0) {
                    if (result[index].length === 0 && index > 0) {
                        let allLevelCheckedItem = clone(tmp);
                        allLevelCheckedItem.key = 'all-root-' + index;
                        allLevelCheckedItem.text = '全部' + this.levelTextArr[index];
                        allLevelCheckedItem.parentKey = index === 1 ? 'all-root' : 'all-root-' + (index - 1);
                        result[index].push(allLevelCheckedItem);
                    }
                    let allcheckedItem = clone(tmp);
                    allcheckedItem.key = index === 0 ? 'all-root' : 'all-part-' + allcheckedItem.parentKey;
                    allcheckedItem.text = '全部' + this.levelTextArr[index];
                    result[index].push(allcheckedItem);
                }
                result[index].push(tmp);
                if (this.levelKeyArr.length > index + 1) {
                    this.walkTree(item[this.levelKeyArr[index + 1]], index + 1, item[this.keyName].toString(), result);
                }
            }
            index = 0;
            return result;
        },
        showNext(item) {
            // clear all active data
            this.showData[item.level].forEach((v) => {
                v.active = false;
            })
            item.active = true;
            let nextLevelData = this.showData[item.level + 1] ? this.showData[item.level + 1] : undefined;
            if (!nextLevelData) return;
            nextLevelData.forEach((v) => {
                v.parentActive = (v.parentKey === item.key) ? true : false;
            })
        },
        checkItem(item) {
            if (this.disabled && !this.firstInit) return;
            item.selected = !item.selected;
            item.partSelected = false;
            if (item.key.indexOf('all-part') >= 0 || item.key.indexOf('all-root') >= 0) {
                this.checkSiblings(item, item.selected);
            } else {
                this.checkPartRoot(item, item.selected);
                this.checkNext(item, item.selected);
            }
            this.checkParent(item, item.selected);
            this.getSelectedText();
        },
        checkSiblings(item, selected) {
            let currentLevelData = this.showData[item.level] ? this.showData[item.level] : undefined;
            if (!currentLevelData) return;
            if (item.key.indexOf('all-part') >= 0) {
                currentLevelData.forEach((v) => {
                    v.selected = (v.parentKey === item.parentKey) ? selected : v.selected;
                    if (this.showData[item.level + 1]) {
                        this.checkNext(v, selected);
                    }
                })
            } else {
                currentLevelData.forEach((v) => {
                    v.selected = selected;
                    v.partSelected = false;
                    // TODO 下钻
                    this.showData[v.level + 1] && this.showData[v.level + 1].forEach((v) => {
                        v.selected = selected;
                    })
                })
            }
        },
        checkPartRoot(item, selected) {
            let allPartSelected = false;
            let allPartNonSelected = false;
            let allPartRootKey = 'all-part-' + item.parentKey;
            if (item.level === 0) {
                allPartRootKey = 'all-' + item.parentKey;
            }
            let partData = this.showData[item.level].filter((v) => {
                return v.parentKey === item.parentKey && v.key !== allPartRootKey;
            });
            let selectedCount = partData.filter((v) => v.selected).length;
            allPartSelected = selectedCount === partData.length ? true : false;
            allPartNonSelected = selectedCount === 0 ? true : false;
            if (selectedCount > 0) {
                if (allPartSelected) {
                    this.showData[item.level].every((v) => {
                        if (v.key === allPartRootKey) {
                            v.selected = true;
                            v.partSelected = false;
                            return false;
                        }
                        return true;
                    })
                } else {
                    this.showData[item.level].forEach((v) => {
                        if (v.key === allPartRootKey) {
                            v.selected = false;
                            v.partSelected = true;
                            return false;
                        }
                        return true;
                    })
                }
            }
            if (allPartNonSelected) {
                this.showData[item.level].every((v) => {
                    if (v.key.indexOf('all-part') >= 0 || v.key === 'all-root') {
                        v.selected = false;
                        v.partSelected = false;
                        return false;
                    }
                    return true;
                })
            }
        },
        checkNext(item, selected) {
            let nextLevelData = this.showData[item.level + 1] ? this.showData[item.level + 1] : undefined;
            if (!nextLevelData) return;
            nextLevelData.forEach((v) => {
                v.selected = (v.parentKey === item.key) ? selected : v.selected;
                if (this.showData[v.level + 1]) {
                    this.checkNext(v);
                }
            })
        },
        checkParent(item, selected) {
            let parentLevelData = this.showData[item.level - 1] ? this.showData[item.level - 1] : undefined;
            if (!parentLevelData) return;
            // check if current selected items is ALL-SELECTED or ALL-NON-SELECTED
            let allSelected = false;
            if (item.key.indexOf('all-root') >= 0 || item.key.indexOf('all-part') >= 0) {
                allSelected = true;
            } else {
                if (this.showData[item.level].every((v) => {
                        return !(v.parentKey === item.parentKey && v.selected !== selected)
                    })) {
                    allSelected = true;
                }
            }

            if (item.key.indexOf('all-root') >= 0) {
                parentLevelData.forEach((v) => {
                    v.selected = selected;
                    v.partSelected = false;
                })
            } else {
                parentLevelData.forEach((v) => {
                    if (v.key === item.parentKey) {
                        v.selected = (allSelected && selected) ? true : false;
                        v.partSelected = (allSelected && !selected) ? false : true;
                        this.checkPartRoot(v, v.selected);
                    }
                    this.checkParent(v, selected);
                })
            }
        },
        getSelectedText() {
            let arr = [];
            this.levelTextArr.forEach((v, index) => {
                let varr = this.showData[index].filter((v) => {
                    return (v.selected || v.partSelected) && (v.key.indexOf('all') === -1);
                })
                arr.push(varr.length + '个' + v);
            })
            if (this.allSelected || this.allNonSelected) {
                this.selectedText = this.allSelected ? this.allSelectedText : this.allNonSelectedText;
            } else {
                this.selectedText = '已选择：' + arr.join('，');
            }
        },
        handleDefaultSelectedData(data) {
            if (!data.length) return;
            let me = this;
            data.forEach((v) => {
                me.showData[me.showData.length - 1].every((item) => {
                    if (v.toString() === item.key) {
                        me.checkItem(item);
                        return false;
                    } 
                    return true;
                })
            })
        },
        bindHideEvent() {
            let me = this;
            this.$el.addEventListener('click', function(e) {
                e.stopPropagation ? e.stopPropagation() : e.cancelBubble = true;
            })
            this.documentListener = function() {
                me.showDropDown = false;
            }
            document.addEventListener('click', this.documentListener);
        }
    },
    destroyed() {
        document.removeEventListener('click', this.documentListener);
    },
    ready() {
        this.checkProps();
        this.bindHideEvent();
    }
}
</script>


