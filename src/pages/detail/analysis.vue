<template>
    <div class="sales-board">
      <div class="sales-board-intro">
        <h2>流量分析</h2>
        <p>是指在获得网站访问量基本数据的情况下对有关数据进行统计、分析，从中发现用户访问网站的规律，并将这些规律与网络营销策略等相结合，从而发现目前网络营销活动中可能存在的问题，并为进一步修正或重新制定网络营销策略提供依据。当然这样的定义是站在网络营销管理的角度来考虑的</p>
      </div>
      <div class="sales-board-form">
          <div class="sales-board-line">
              <div class="sales-board-line-left">
                  购买数量：
              </div>
              <div class="sales-board-line-right">
                <v-counter :max="15" :min="1" @on-change="onParamChange('buynum',$event)"></v-counter>
              </div>
          </div>
          <div class="sales-board-line">
              <div class="sales-board-line-left">
                  产品类型：
              </div>
              <div class="sales-board-line-right">
                 <VSelection :selections="productTypes" @on-change="onParamChange('buyType',$event)"></VSelection>
              </div>
          </div>
          <div class="sales-board-line">
              <div class="sales-board-line-left">
                  有效时间：
              </div>
              <div class="sales-board-line-right">
                 <v-chooser :selections="timeType" @on-change="onParamChange('period',$event)"></v-chooser>
              </div>
          </div>
          <div class="sales-board-line">
              <div class="sales-board-line-left">
                  产品版本：
              </div>
              <div class="sales-board-line-right">
                 <v-multi :selections="versionType" @on-change="onParamChange('version',$event)"></v-multi>
              </div>
          </div>
          <div class="sales-board-line">
              <div class="sales-board-line-left">
                  总价：
              </div>
              <div class="sales-board-line-right">
                  {{price}}元
              </div>
          </div>
          <div class="sales-board-line">
              <div class="sales-board-line-left">&nbsp;</div>
              <div class="sales-board-line-right">
                  <div class="button" @click="showPayDialog">
                    立即购买
                  </div>
              </div>
          </div>
      </div>
      <div class="sales-board-des">
        <h2>产品说明</h2>
        <p>网站访问统计分析报告的基础数据源于网站流量统计信息，但其价值远高于原始数据资料。专业的网站访问统计分析报告对网络营销的价值，正如专业的财务分析报告对企业经营策略的价值。</p>

        <h3>用户行为指标</h3>
        <ul>
          <li>用户行为指标主要反映用户是如何来到网站的、在网站上停留了多长时间、访问了哪些页面等，主要的统计指标包括：</li>
          <li>用户在网站的停留时间；</li>
          <li>用户来源网站（也叫“引导网站”）；</li>
          <li>用户所使用的搜索引擎及其关键词；</li>
          <li>在不同时段的用户访问量情况等。</li>
        </ul>

        <h3>浏览网站方式</h3>
        <ul>
          <li>用户上网设备类型</li>
          <li>用户浏览器的名称和版本</li>
          <li>访问者电脑分辨率显示模式</li>
          <li>用户所使用的操作系统名称和版本</li>
          <li>用户所在地理区域分布状况等</li>
        </ul>
      </div>
   
      <my-dialog :isShow="isShowPayDialog" @onClose="hidePayDialog">
        <table class="buy-dialog-table">
          <tr>
            <th>购买数量</th>
            <th>产品类型</th>
            <th>有效时间</th>
            <th>产品版本</th>
            <th>总价</th>
          </tr>
          <tr>
            <td>{{ buynum }}</td>
            <td>{{ buyType.label }}</td>
            <td>{{ period.label }}</td>
            <td>
              <span v-for="item in version" >
                {{item.label}}
              </span>
            </td>
            <td> {{price}}</td>
          </tr>
        </table>
        <h3 class="buy-dialog-title">请选择银行</h3>
          <BankChooser @on-change="onChangeBank"></BankChooser>
        <div class="button buy-dialog-btn" @click="confirmBuy">
          确认购买
        </div>
      </my-dialog>
        <my-dialog :isShow="isShowErrDialog"
        @on-close="hideErrDialog"></my-dialog>
      <checkOrder :isShowCheckDialog="isShowCheckOrder"
      :order-id="orderId"
      @on-close-check-dialog="hideCheckOrder"></checkOrder>
  </div>
</template>

<script>
import VSelection from '../../components/selection.vue'
import VCounter from '../../components/counter.vue'
import VChooser from '../../components/chooser.vue'
import VMulti from '../../components/multiChooser.vue'
import _ from 'lodash'
import Dialog from '../../components/dialog.vue'
import BankChooser from '../../components/bankChooser.vue'
import checkOrder from '../../components/checkOrder.vue'
export default {
  components:{
    VSelection,
    VCounter,
    VChooser,
    VMulti,
    MyDialog:Dialog,
    BankChooser,
    checkOrder
  },
  mounted(){
      this.buynum=0,
      this.buyType=this.productTypes[0],
      this.version=[this.versionType[0]],
      this.period=this.timeType[0],
      this.getPrice()
  },
  methods:{
    onParamChange(attr,value){
      this[attr]=value
     this.getPrice()
    },
    getPrice(){
      let versionArr=_.map(this.version,'value')
      let reqArg={
        buyNumber:this.buynum,
        buyType:this.buyType.value,
        period:this.period.value,
        version:versionArr.join(',')
      }
      this.$http.post('/api/getPrice',reqArg)
      .then((res)=>{
        this.price=res.data.amount
      })
    },
    showPayDialog () {
      this.isShowPayDialog = true
    },
    hidePayDialog () {
      this.isShowPayDialog= false
    },
    onChangeBank(bankObj){
        this.bankId=bankObj.id
    },
    confirmBuy(){
      let versionArr = _.map(this.versions, (item) => {
        return item.value
      })
      let reqArg={
        buyNumber:this.buynum,
        buyType:this.buyType.value,
        period:this.period.value,
        version:versionArr.join(','),
        bankId:this.bankId
      }
      this.$http.post('/api/createOrder', reqArg)
      .then((res) => {
        this.orderId = res.data.orderId
        this.isShowCheckOrder = true
        this.isShowPayDialog = false
      }, (err) => {
        this.isShowBuyDialog = false
        this.isShowErrDialog = true
      })
    },
    hideErrDialog(){
      this.isShowErrDialog=false
    },
    hideCheckOrder(){
      this.isShowCheckOrder=false
    }
  },
  data(){
    return {
      isShowCheckOrder:false,
      isShowErrDialog:false,
      orderId:null,
      bankId:null,
      isShowPayDialog: false,
      buynum:1,
      buyType:{},
      version:[],
      period:{},
      price:0,
    productTypes:[
    {
      label:'入门',
      value:1
    },
    {
      label:'中级',
      value:2
    },
    {
      label:'高级',
      value:3
    }],

    timeType:[
      {
        label:'半年',
        value:1
      },
      {
        label:'1年',
        value:1
      },
      {
        label:'2年',
        value:1
      }],

      versionType:[
      {
        label:'商户版',
        value:1
      },
      {
        label:'代理商版',
        value:2
      },
      {
        label:'专家版',
        value:3
      }]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.buy-dialog-title {
  font-size: 16px;
  font-weight: bold;
}
.buy-dialog-btn {
  margin-top: 20px;
}
.buy-dialog-table {
  width: 100%;
  margin-bottom: 20px;
}
.buy-dialog-table td,
.buy-dialog-table th{
  border: 1px solid #e3e3e3;
  text-align: center;
  padding: 5px 0;
}
.buy-dialog-table th {
  background: #4fc08d;
  color: #fff;
  border: 1px solid #4fc08d;
}
</style>
