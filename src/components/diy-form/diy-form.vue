<template>
  <div class="form-cell" v-if="formList.length>0">
    <div v-for="(item,idx) in formList" :key="idx" @click="setActiveIdx(idx)">
      <div v-if="item.type==='input'" class="form-cell-item cell-item" :class="{nobor:idx===formList.length-1}">
        <div class="form-cell-item__label">{{item.label}}</div>
        <div class="form-cell-item__content">
          <input placeholder-class="__placeholder" @input="inputChange($event,idx)" :value="item.value" class="input" :placeholder="item.desc" />
        </div>
      </div>
      <div v-if="item.type==='picker'" class="form-cell-item cell-item" :class="{nobor:idx===formList.length-1}">
        <div class="form-cell-item__label">{{item.label}}</div>
        <div class="form-cell-item__content">
          <picker style="width: 100%" :range="item.options" @change="pickerChange($event,idx)">
            <div class="" v-if="item.value">{{item.value}}</div>
            <div class="__placeholder" v-else>{{item.desc||'请选择'}}</div>
          </picker>
        </div>
        <div class="form-cell-item__right">
          <layout-icon type="iconicon-arrow-right" color="#999"></layout-icon>
        </div>
      </div>
      <div v-if="item.type==='area'" class="form-cell-item cell-item" :class="{nobor:idx===formList.length-1}">
        <div class="form-cell-item__label">{{item.label}}</div>
        <div class="form-cell-item__content">

          <wzw-address @up="updateAddress" ref="address">
            <div @click="openPop('address')">
              <div class="__placeholder" v-if="!item.value">{{item.desc||'选择位置'}}</div>
              <div v-if="item.value" class="fz-12" style="line-height: 16px;">{{item.value}}</div>
            </div>
          </wzw-address>
        </div>
        <div class="form-cell-item__right">
          <layout-icon type="iconicon-arrow-right" color="#999"></layout-icon>
        </div>
      </div>
      <div v-if="item.type==='img'" class="form-upload-item cell-item" :class="{nobor:idx===formList.length-1}">
        <div class="form-upload-item__label">{{item.label}}</div>
        <div class="form-upload-item__content">

          <upload-image
            :key="idx"
            :limit="1"
            :ext="idx"
            @done="upImgSuccess"
          ></upload-image>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import { objTranslate } from '@/common/helper'
import { hideLoading, showLoading } from '@/common/fun'
import LayoutIcon from '@/components/layout-icon/layout-icon'
import UploadImage from '@/components/upload-image/upload-image'
import WzwAddress from '@/components/wzw-address/wzw-address'

export default {
  name: 'DiyForm',
  components: { WzwAddress, UploadImage, LayoutIcon },
  props: {
    eid: {
      type: String,
      require: true
    },
    forms: {
      type: Array,
      require: true
    }
  },
  data () {
    return {
      idx: 0,
      formList: []
    }
  },
  watch: {
    formList: {
      deep: true,
      immediate: true,
      handler (newVal) {
        // console.log(`${this.eid} formData change`, newVal)
        if (Array.isArray(newVal) && newVal.length > 0) {
          this.$emit('update', objTranslate(newVal))
        }
      }
    }
  },

  created () {
    const _arr = objTranslate(this.forms)
    // 我也太聪明了
    if (Array.isArray(_arr) && _arr.length > 0) {
      this.formList = _arr.map(row => {
        // 只有picker是要特殊处理的
        if (row.type === 'picker') {
          row.options = row.value.split('|')
          row.value = ''
        }
        return row
      })
      console.log(this.formList)
    }
  },
  methods: {
    openPop (name) {
      this.$refs[name][0].show()
    },
    updateAddress (data) {
      // console.log(data)
      this.setVal(data.strArr.join(''))
    },
    getData () {
      // console.log('formList is', this.formList)
      // return objTranslate(this.formList)
    },
    openAddressChoose () {
      const rt = { province: '北京', city: '北京', area: '西山', town: '香山公园' }
      showLoading('获取地址')
      setTimeout(() => {
        hideLoading()
        this.addressCall(rt)
      }, 500)
    },
    setActiveIdx (idx) {
      console.log(`active index change,current idx is ${idx}`)
      this.idx = idx
    },
    setVal (val, idx) {
      const _idx = idx || this.idx
      this.$set(this.formList[_idx], 'value', val)
    },
    pickerChange (e,idx) {
      const val = e.detail.value
      this.setVal(this.formList[idx].options[val])
    },
    inputChange (e,idx) {
      const val = e.detail.value
      this.setVal(val, idx)
    },
    addressCall ({ province, city, area, town }) {
      // const val = e.detail.value
      this.setVal(Object.values({ province, city, area, town }).join(' '))
    },
    upImgSuccess ({ imgs, ext }) {
      console.log(imgs, ext)
      this.setVal(imgs.join('|'))
    }
  }
}
</script>
<style lang="scss" scoped>
  .nobor{
    border-bottom: none !important;
  }
  .form{

    &-panel{
      margin: 10px;
    }
    &-title{
      display: flex;
      align-items: center;
      padding: 10px 0;
      &__place{
        background: #00A8FF;
        width: 4px;
        height: 16px;
        margin: 0 10px;
        border-radius: 2px;
      }
      &__text{
        color: #333333;
        font-weight: bold;
      }
    }
    &-upload{

      &-item{
        display: block;
        margin: 0 10px;
        padding: 10px 0;
        border-bottom: 1px solid #eee;

        &__label{
          color: #555555;
        }
        &__content{
          padding: 10px 0;
        }

      }

    }
    &-cell{
      background: white;
      border-radius: 4px;
      overflow: hidden;
      font-size: 14px;

      &-item{
        display: flex;
        align-items: center;
        padding: 10px 0;
        border-bottom: 1px solid #eee;
        margin: 0 10px;

        &__label{
          width: 80px;
          padding: 0 10px 0 0;
          color: #555555;
          line-height: 1.3;
        }
        &__content{
          flex: 1;
          height: 32px;
          line-height: 32px;
          display: flex;
          align-items: center;
          .input{
            font-size: 14px;
            flex: 1;
            color: #555555;
            &::placeholder{
              color: #CAC8C8;
            }
          }

          .__placeholder{
            color: #CAC8C8;
          }

        }
        &__right{
          width: 32px;
          text-align: center;
        }

      }
    }

  }
</style>
