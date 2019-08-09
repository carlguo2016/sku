<template>
  <div class="standard">
    <div class="standard-container">
      <div class="standard-panel" v-for="(spec, indent) in specification" :key="indent">
        <div class="standard-head">
          <div class="standard-group">
            <div class="inline-text inline-title">规格名:</div>
            <el-input v-model="spec.name" @blur="handleChangeSpecification('add')" size="mini" class="inline-input" />
            <el-checkbox v-if="image && indent === 0" v-model="checked" label="规格图片" border size="mini"></el-checkbox>
          </div>
          <i class="el-icon-error" @click="handleRemoveSpecification(indent)"></i>
        </div>
        <div class="standard-lists">
          <div class="standard-value inline-text inline-title">规格值:</div>
            <div class="standard-value standard-input" v-for="(tag, index) in spec.attribute" :key="index">
              <el-tag size="medium" closable @close="handleRemoveAttribute(indent, index)">{{ tag }}</el-tag>
            </div>
            <div v-if="visiableCache[indent]" class="standard-input" style="width: 92px; margin: 0;">
              <el-input
                :ref="`input_${indent}`"
                v-model="attributeModelCache[indent]"
                size="mini"
                @keyup.enter.native="handleAddAttribute(indent)"
                @blur="handleAddAttribute(indent)"
              />
            </div>
            <el-button v-else size="mini" @click="handleToggleInput(indent)">添加规格值</el-button>
        </div>
      </div>
      <el-tooltip v-if="!enable" :content="`最多支持${max}条规格`" class="item" effect="dark" placement="top">
        <el-button type="info" class="standard-btn" size="mini" plain>添加规格</el-button>
      </el-tooltip>
      <el-button v-else type="primary" class="standard-btn" size="mini" plain @click="handlePushSpecification">添加规格</el-button>
    </div>
    <template v-if="skus.length">
      <el-table style="margin-top: 10px" border :data="data" :span-method="handleSpanMethod">
        <el-table-column
          v-for="(item, indent) in specification"
          :key="indent"
          :prop="item.name"
          :label="item.name">
        </el-table-column>
        <el-table-column
          prop="price"
          label="价格"
          width="120">
          <template slot-scope="scope">
            <el-input size="mini" v-model="skus[scope.$index].price" />
          </template>
        </el-table-column>
        <el-table-column
          prop="marked_price"
          label="市场价"
          width="120">
          <template slot-scope="scope">
            <el-input size="mini" v-model="skus[scope.$index].marked_price" />
          </template>
        </el-table-column>
        <el-table-column
          prop="stock"
          label="库存"
          width="120">
          <template slot-scope="scope">
            <el-input size="mini" v-model="skus[scope.$index].stock" />
          </template>
        </el-table-column>
      </el-table>
      <div class="standard-tools">
        <span class="inline-text">批量设置</span>
        <el-radio-group v-model="mode" size="mini">
          <el-radio-button label="price">价格</el-radio-button>
          <el-radio-button label="marked_price">市场价</el-radio-button>
          <el-radio-button label="stock">库存</el-radio-button>
        </el-radio-group>
        <el-input v-if="mode" v-model="value"><el-button type="primary" slot="append" icon="el-icon-setting" @click="handleSettingSkus"></el-button></el-input>
      </div>
    </template>
    <template v-if="test">
      <h3>规格</h3>
      <small><pre>{{specification}}</pre></small>
      <h3>库存组合</h3>
      <small><pre>{{skus}}</pre></small>
      <h3>表格数据</h3>
      <small><pre>{{data}}</pre></small>
      <h3>合并表格行</h3>
      <small><pre>{{spanCollection}}</pre></small>
    </template>
  </div>
</template>

<script>
import { uniq, cloneDeep, isEqual } from 'lodash'

export default {
  props: {
    spec: Array,
    sku: Array,
    // 测试
    test: Boolean
  },
  data () {
    return {
      max: 3,
      image: false,
      checked: false,
      // 规格集合
      specification: this.spec,
      // 库存集合
      skus: this.sku,
      // 缓存input规格值
      attributeModelCache: [],
      // 缓存input切换状态
      visiableCache: [],
      data: [],
      spanCollection: [],
      // 快捷设置
      mode: '',
      value: ''
    }
  },

  computed: {
    // 计算添加规格状态
    enable () {
      return this.specification.length < this.max
    },

    skusCombination () {
      return this.skus.map(item => item.combination)
    }
  },

  watch: {
    spec () {
      this.$emit('change-spec', this.spec)
    },

    skus () {
      this.$emit('change-sku', this.skus)
    }
  },

  methods: {
    // 计算table 每行重复
    computeRowspan () {
      this.spanCollection = []
      const rowspan = (name) => {
        let span = []
        let dot = 0

        this.data.map((item, index) => {
          if (index === 0) {
            span.push(1)
          } else {
            if (item[name] === this.data[index - 1][name]) {
              span[dot] += 1
              span.push(0)
            } else {
              dot = index
              span.push(1)
            }
          }
        })

        this.spanCollection.push(span)
      }

      this.specification.map(item => {
        rowspan(item.name)
      })
    },
    // 添加规格
    handlePushSpecification () {
      console.log(this.enable)
      if (!this.enable) return

      this.specification.push({
        name: '',
        attribute: []
      })
    },
    // 删除规格
    handleRemoveSpecification (indent) {
      this.specification.splice(indent, 1)

      this.handleChangeSpecification('del')
    },
    // 切换input状态
    handleToggleInput (indent, status = true) {
      this.$set(this.visiableCache, indent, status)

      status && this.$nextTick(() => {
        this.$refs[`input_${indent}`][0].focus()
      })
    },
    // 重置input model
    handleResetAttributeCache (indent) {
      this.attributeModelCache[indent] = ''
    },
    // 添加规格值
    handleAddAttribute (indent) {
      let attributes = (this.attributeModelCache[indent] || '').trim()

      if (attributes) {
        // 空格分割
        let tags = attributes.split(/\s+/)

        // 去重
        tags = uniq(this.specification[indent].attribute.concat(tags))
        // 更新indent所属attribute
        this.$set(this.specification[indent], 'attribute', tags)

        this.handleResetAttributeCache(indent)

        this.handleChangeSpecification('add')
      }

      this.handleToggleInput(indent, false)
    },
    // 移除规格值
    handleRemoveAttribute (indent, index) {
      this.specification[indent].attribute.splice(index, 1)

      this.handleChangeSpecification('del')
    },
    // 计算乘积
    computedSkus (indent) {
      let num = 1

      this.specification.map((item, index) => {
        if (index >= indent && item.attribute.length) {
          num *= item.attribute.length
        }
      })
      // console.log(num)
      return num
    },
    // 监听规格值变化
    handleChangeSpecification (action) {
      // 深度拷贝库存集合
      const deep = cloneDeep(this.skus)

      if (action === 'del') {
        this.skus = []
      }

      console.log(this.computedSkus(0))

      for (let i = 0; i < this.computedSkus(0); i++) {
        this.handleChangeStocks(action, i, deep)
      }
    },
    // 生成库存数据集合
    handleChangeStocks (action, index, deep) {
      let sku = {
        price: 0,
        stock: 0,
        marked_price: 0,
        combination: this.getCombination(index)
      }

      const combination = sku.combination
      if (action === 'add') {
        // 查找是否已存在库存组合
        if (this.skusCombination.findIndex(item => isEqual(combination, item)) === -1) {
          this.$set(this.skus, index, sku)
        }
      } else if (action === 'del') {
        // 删除库存
        let collection = ''

        deep.forEach(item => {
          console.log(combination, item.combination, isEqual(combination, item.combination))
          if (isEqual(combination, item.combination)) {
            collection = item
            return false
          }
        })
        this.skus.push(collection || sku)
      }

      this.tableData()
    },
    tableData () {
      this.data = []

      this.skus.map((item, index) => {
        this.data.push({
          ...item,
          ...item.combination
        })
      })

      this.computeRowspan()
      // console.log(data)
      // return data
    },
    // 获取规格属性
    handleGetSpecificationAttribute (indent, index) {
      // return this.specification[indent].attribute[index]
      // 获取当前规格项目下的属性值
      const attributes = this.specification[indent].attribute
      let _index

      // 判断是否最后一个规格项目
      if (this.specification[indent + 1] && this.specification[indent + 1].attribute.length) {
        _index = index / this.computedSkus(indent + 1)
      } else {
        _index = index
      }

      const i = Math.floor(_index % attributes.length)

      if (i.toString() !== 'NaN') {
        return attributes[i]
      } else {
        return ''
      }
    },
    // 组合
    getCombination (index) {
      let combination = {}

      this.specification.map((item, indent) => {
        combination[item.name] = this.handleGetSpecificationAttribute(indent, index)
      })

      return combination
    },
    // 设置rowspan
    handleSpanMethod ({ row, column, rowIndex, columnIndex }) {
      console.log(row, column, rowIndex, columnIndex)

      for (let i = 0; i < this.specification.length; i++) {
        if (columnIndex === i) {
          if (this.spanCollection[i] && this.spanCollection[i][rowIndex]) {
            return {
              rowspan: this.spanCollection[i][rowIndex],
              colspan: 1
            }
          } else {
            return {
              rowspan: 0,
              colspan: 0
            }
          }
        }
      }

      // if (columnIndex < this.specification.length) {
      //   return {
      //     rowspan: row.rowspan,
      //     colspan: 1
      //   }
      // }

      // return {
      //   rowspan: 0,
      //   colspan: 0
      // }
    },
    // 判断是否显示当前行
    computedShow (indent, index) {
      // 如果当前项目下没有属性，则不显示
      if (!this.specification[indent]) {
        return false
      // 自己悟一下吧
      } else if (index % this.computedSkus(indent + 1) === 0) {
        return true
      } else {
        return false
      }
    },

    handleSettingSkus () {
      if (this.value === '') return

      this.skus.map(item => {
        item[this.mode] = this.value
      })
    }
  },

  created () {
    if (this.test) {
      this.specification = [
        {
          name: '颜色',
          attribute: [
            '红色', '白色', '黑色'
          ]
        }
      ]

      this.skus = [
        {
          price: 0,
          stock: 0,
          marked_price: 0,
          combination: {
            '颜色': '红色'
          }
        },
        {
          price: 0,
          stock: 0,
          marked_price: 0,
          combination: {
            '颜色': '白色'
          }
        },
        {
          price: 0,
          stock: 0,
          marked_price: 0,
          combination: {
            '颜色': '黑色'
          }
        }
      ]
    }
  },

  mounted () {
    this.tableData()
  }
}
</script>

<style lang="scss">
.standard {
  width: 100%;
  box-sizing: border-box;
  overflow: hidden;

  .el-input__inner {
    padding: 0 5px;
  }

  .inline-text {
    font-size: 12px;
  }

  .inline-title {
    width: 50px;
    height: 28px;
    line-height: 28px;
  }

  .inline-input {
    width: 120px;
  }

  .standard-input {
    margin-right: 10px;
  }

  .standard-panel {

    &:hover {
      .el-icon-error {
        margin-right: 0;
      }
    }
  }

  .standard-head {
    padding: 7px 10px;
    background-color: #f8f8f8;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;

    .el-icon-error {
      margin-right: -50px;
      cursor: pointer;
      transition: all .3s;
    }

    .standard-group {
      display: flex;
      flex-flow: row nowrap;
      align-items: center;
    }

    .el-checkbox.is-bordered {
      border: 0;
    }
  }

  .standard-lists {
    padding: 10px 10px 0;
    display: flex;
    flex-flow: row wrap;
    align-items: flex-start;
    margin-bottom: 10px;
  }

  .standard-tools {
    margin-top: 10px;
    display: flex;
    flex-flow: row nowrap;
    align-items: center;

    .inline-text {
      margin-right: 10px
    }

    .el-input {
      margin-left: 10px;
      width: 150px;
    }
  }
}
</style>
