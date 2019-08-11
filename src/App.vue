<template>
  <div id="app">
    <div class="container">
      <specification :spec="spec" :sku="sku" />
    </div>
  </div>
</template>

<script>
import Specification from '@/components/specification'

export default {
  data () {
    return {
      spec: [
        {
          'id': 1,
          'name': '颜色',
          'category_id': null,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'pivot': {
            'goods_id': 1,
            'specification_id': 1
          },
          'attribute': [
            '红色',
            '黑色'
          ]
        },
        {
          'id': 2,
          'name': '尺寸',
          'category_id': null,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'pivot': {
            'goods_id': 1,
            'specification_id': 2
          },
          'attribute': [
            'xxl',
            'xxxl'
          ]
        }
      ],
      sku: [
        {
          'id': 1,
          'code': null,
          'price': '0.00',
          'marked_price': '0.00',
          'stock': 0,
          'goods_id': 1,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'combination': {
            '颜色': '红色',
            '尺寸': 'xxl'
          }
        },
        {
          'id': 2,
          'code': null,
          'price': '0.00',
          'marked_price': '0.00',
          'stock': 0,
          'goods_id': 1,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'combination': {
            '颜色': '红色',
            '尺寸': 'xxxl'
          }
        },
        {
          'id': 3,
          'code': null,
          'price': '0.00',
          'marked_price': '0.00',
          'stock': 0,
          'goods_id': 1,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'combination': {
            '颜色': '黑色',
            '尺寸': 'xxl'
          }
        },
        {
          'id': 4,
          'code': null,
          'price': '0.00',
          'marked_price': '0.00',
          'stock': 0,
          'goods_id': 1,
          'created_at': '2019-08-11 22:48:23',
          'updated_at': '2019-08-11 22:48:23',
          'combination': {
            '颜色': '黑色',
            '尺寸': 'xxxl'
          }
        }
      ]
    }
  },

  components: {
    Specification
  },

  methods: {
    handleChangeSpec (spec) {
      this.spec = spec
    },

    handleChangeSku (sku) {
      this.sku = sku
    }
  }
}
</script>

<style lang="scss">
.container {
  width: 1000px;
  margin: auto;
}
</style>
