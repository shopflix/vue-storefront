<template>
  <div class="ckt-item info">
    <div class="top-ckt">
      <span class="title-top">Consignee</span>
      <span class="other-top"><a href="javascript:;" @click="handleAddAddress($refs.addressForm)">Add address</a></span>
      <div class="clearfix"></div>
    </div>

    <!--Address list-->
    <div class="center-ckt-info" :style="{height: (expanded ? addressList.length * 42 : 114) + 'px'}">
      <div v-if="addressList && !addressList.length" class="empyt-addr">
        <img src="../../assets/images/icon-empty-member.png" alt="">
        <p>You dont have the shipping address yet, please first[<a href="javascript:void(0)" @click="handleAddAddress">Add a shipping address</a>]</p>
      </div>
      <ul v-else id="address_list">
        <li
          v-for="item in ckAddressList"
          :key="item.addr_id"
          :class="['li-ckt-info', addressId === item.addr_id && 'selected']"
        >
          <div class="ckt-checkbox info" @click="handleSelectAddress(item)"> <!--w: 150px-->
            <span v-if="item.ship_address_name" :title="item.ship_address_name">{{ item.ship_address_name }}</span>
            <span v-else :title="item.name">{{ item.name }}</span>
          </div>
          <div class="name-li-ckt-info" :title="item.name">{{ item.name }}</div>
          <div class="address-li-ckt-info" :title="formatterAddress(item)">{{ formatterAddress(item) }}</div>  <!--40Character length-->
          <div class="mobile-li-ckt-info" :title="item.mobile">{{ item.mobile }}</div>
          <div v-if="item.def_addr" class="default-li-ckt-info">The default address</div>
          <div class="operate-li-ckt-info">
            <a v-if="!item.def_addr" href="javascript:;" class="set" @click="handleSetDefaultAddress(item)">Set to default</a>
            <a href="javascript:;" class="edit" @click="handleEaitAddress(item)">edit</a>
            <a v-if="!item.def_addr" href="javascript:;" class="delete" @click="handleDeleteAddress(item)">delete</a>
          </div>
        </li>
      </ul>
    </div>
    <!--Address listend-->
    <div
      v-if="addressList.length > 3"
      :class="['collapse-ckt-info', expanded && 'more']"
      @click="() => { expanded = !expanded }"
    >
      <span class="more-collapse-ckt">{{ expanded ? 'Put the address' : 'An address' }}</span>
      <i class="icon-collapse-ckt-info"></i>
    </div>
    <div class="placeholder-20"></div>
    <div id="addressForm" style="display: none">
      <el-form :model="addressForm" :rules="addressRules" ref="addressForm" label-width="100px" style="width: 450px">
        <el-form-item label="Name of consignee" prop="name">
          <el-input v-model="addressForm.name" size="small"></el-input>
        </el-form-item>
        <el-form-item label="contact" prop="mobile">
          <el-input v-model="addressForm.mobile" size="small" :maxlength="11"></el-input>
        </el-form-item>
        <el-form-item label="Receiving area" prop="region">
          <en-region-picker :api="MixinRegionApi" :default="regions" @changed="(object) => { this.addressForm.region = object.last_id }"/>
        </el-form-item>
        <el-form-item label="Detailed address" prop="addr">
          <el-input v-model="addressForm.addr" size="small"></el-input>
        </el-form-item>
        <el-form-item label="Address the alias" prop="ship_address_name">
          <el-input v-model="addressForm.ship_address_name" size="small" placeholder="The company、In the home、School or something"></el-input>
        </el-form-item>
        <el-form-item label="Set to default">
          <el-checkbox v-model="addressForm.def_addr" :true-label="1" :false-label="0">default</el-checkbox>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
  /**
   * The settlement page
   * Delivery address component
   */
  import Vue from 'vue'
  import { Checkbox, Form, FormItem } from 'element-ui'
  Vue.use(Checkbox).use(Form).use(FormItem)
  import addressMixin from '@/pages/member/addressMixin'
  import * as API_Trade from '@/api/trade'
  export default {
    name: 'checkout-address',
    // Share mixins with personal centers
    mixins: [addressMixin],
    props: ['address-id'],
    data() {
      return {
        expanded: false
      }
    },
    computed: {
      /** Put the selected address in the first place*/
      ckAddressList() {
        const address = this.addressList
        if (!address || address.length === 0) return address
        const selIndex = address.findIndex(item => item.addr_id === this.addressId)
        if (selIndex === -1) return address
        const selAddress = JSON.parse(JSON.stringify(address[selIndex]))
        address.splice(selIndex, 1)
        address.splice(0, 0, selAddress)
        return address
      }
    },
    watch: {
      // If there is no address, set the first one to the selected address
      // If there is an address, compare the address
      addressList: function (newVal, oldVal) {
        if (this.addressId || !newVal || !newVal.length || newVal.length === oldVal.length) return
        this.handleSelectAddress(newVal[0], false)
      }
    },
    methods: {
      /** Formatting address information*/
      formatterAddress(address) {
        return `${address.province} ${address.city} ${address.county} ${address.town} ${address.addr}`
      },
      /** Select the shipping address*/
      handleSelectAddress(item, showMsg) {
        if (item.addr_id === this.addressId) return
        API_Trade.setAddressId(item.addr_id).then(response => {
          showMsg !== false && this.$message.success('Set up the success！')
          this.$emit('change', item)
        })
      }
    }
  }
</script>

<style type="text/scss" lang="scss" scoped>
  @import "../../assets/styles/color";
  .add-address-btn {
    position: absolute;
    top: 0;
    right: 0;
  }
  .address-list {
    /deep/ .el-alert { margin: 10px 0}
    /deep/ .delete-btn { color: $color-main }
    /deep/ .address-cell {
      text-align: center;
    }
  }
  #addressForm{
    padding: 10px 20px;
    /deep/ .app-address { margin-top: 7px }
  }
  .empyt-addr {
    width: 100%;
    text-align: center;
    a {
      color: $color-href;
    }
  }
</style>
