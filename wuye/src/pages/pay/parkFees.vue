<template>
	<div class="main">
		<!-- <mt-navbar id="navBar"  v-model="selected">
	    	<mt-tab-item id="c">停车缴费</mt-tab-item>
	    </mt-navbar> -->
		<mt-tab-container v-model="selected"> 
		  <mt-tab-container-item id="c">
			<!-- 停车缴费开始 -->
			  <mt-loadmore 
			  	:bottomMethod="carLoadBottom" 
			  	:auto-fill = "false"
			  	:bottomAllLoaded = "carisLastPage"
				ref="loadmore"
				@bottom-status-change="handleBottomChange"
			  >
			  	<Bill :bill-info="carBillInfo" @itemClick="itemClick"></Bill>
				<div slot="bottom" class="mint-loadmore-bottom">
					<!-- :class="{ 'is-rotate ': bottomStatus === 'drop' }" -->
					<span v-show="bottomStatus !== 'loading'" >上拉加载</span>
					<span v-show="bottomStatus === 'loading'">
						<mt-spinner type="snake"></mt-spinner>
					</span>
				</div> 
			  </mt-loadmore>
			  <div style="widtt:100%;height:0.92rem;"></div>
			  	<div class="btn-fixed">
		    		<div class="fl select-btn" v-show="quan" :class="{allSelected:carAllselect }" @click="allSelect(carBillInfo,'carAllselect')">全选&nbsp;</div>
		    		<div class="pay" @click="pay('carBillInfo','carAllPrice','carAllselect')">
		    			我要缴费
		    			<span>
		    				￥{{carAllPrice}}
		    			</span>
		    		</div>
        		</div>
			<!-- 停车缴费结束 -->
		  </mt-tab-container-item>
		  
		</mt-tab-container>
	</div>
</template>
<script>
	let vm;
	import wx from 'weixin-js-sdk';
	import Bill from '../../components/bill.vue';
	import { Loadmore} from 'mint-ui';
	import { Spinner } from 'mint-ui';
	export default {
		components:{Bill},
		computed:{
			//停车缴费总价
		  	carAllPrice : function(){
		  		let ap = 0;
		  		for(let i in this.carBillInfo){
		  			if(this.carBillInfo[i].selected == true){
		  				ap+=Number(this.carBillInfo[i].fee_price)
		  			}
		  		}
		  		return parseFloat(ap).toFixed(2)
		    },

		},
		data(){
			return {
				url : '/billList',
				stmtId:'',//快捷缴费 扫描出来的账单号
		  		params : {
		  			startDate:'',
		  			endDate:'',
		  			payStatus: '02', //写死
		  			currentPage : 1, //页码
		  			totalCount : 0, //第几条开始
		  		},
				carisLastPage:false,//停车缴费是否最后一页
				carAllselect:false,//停车缴费全选
				carBillInfo:[],//停车缴费数据
				selected:'c', //选项卡 默认选中
				carBillPage :1, //停车缴费页码
				reduceMode:1,
				quan:false,
				permit_skip_pay:'1',
				bottomStatus: '',
			}
		},
		created(){
		  	vm = this;
		},
		mounted(){
			this.common.checkRegisterStatus();
			//微信配置
		  	let url = location.href.split('#')[0]
		  	vm.receiveData.wxconfig(vm,wx,['scanQRCode'],url);
		  	
		  	//请求 停车缴费 和 物业缴费首屏数据
		  	vm.receiveData.getData(vm, vm.url, 'data',function(){
							
		  		vm.billInfo = vm.data.result.bill_info;//物业缴费
		  		vm.reduceMode = vm.data.result.reduce_mode;//减免方式
		  		vm.carBillInfo = vm.data.result.car_bill_info;//停车缴费
				vm.pay_least_month = vm.data.result.pay_least_month; //最少支付月数 
				vm.permit_skip_pay=vm.data.result.permit_skip_pay;  
		  	},vm.params) ;
		  	
		},
		methods:{
			 handleBottomChange(status) {
			  vm.bottomStatus= status
		 	 },
			carLoadBottom(){//停车缴费，上拉加载数据
		  		//临时接收的数组
		  		let tempArr = null;
		  		//页码自增 
		  		vm.carBillPage+=1
				vm.params.currentPage = vm.carBillPage;
				//请求接口数据
				vm.receiveData.getData(
					vm,
					vm.url,
					'pageData2',
					function(){
		  				tempArr = vm.pageData2.result.car_bill_info;//停车缴费
		  				if(tempArr && tempArr.length > 0){
		  					vm.carBillInfo =vm.carBillInfo.concat(tempArr) //停车缴费
		  					vm.carAllselect = false;
		  				}else{
							  vm.carisLastPage = true;
							  vm.quan=true;
						  }
		  				 vm.$refs.loadmore.onBottomLoaded()
		  			},vm.params)
		  			
		  	},
		  	//点击物业缴费按钮
			  pay(list,allPrice,allselect){//第一个参数 账单数组，第二个参数 总价 第三个参数 是否全选,所有参数 string
		  		// this.common.checkRegisterStatus();
		  		if( vm[allPrice] < 0.01){
		  			alert('请选择账单后支付');
		  			return
		  		}
		  		let selectedArr = [];
		  		var sel_bill_arr = new Array();
		  		if(vm[allselect] == true){
		  			//全部选中
		  			//sel_bill_arr = vm[list];
		  			// var ret = jQuery.inArray(vm[list][i].service_fee_cycle,sel_bill_arr);
		  			// if(-1 == ret){
		  			// 	sel_bill_arr.push(vm[list][i].service_fee_cycle);
		  			// }
		  			// selectedArr.push(vm[list][i])
		  			for(let i in vm[list]){
		  				var ret = jQuery.inArray(vm[list][i].service_fee_cycle,sel_bill_arr);
			  			if(-1 == ret){
			  				sel_bill_arr.push(vm[list][i].service_fee_cycle);
			  			}	
	  					selectedArr.push(vm[list][i])
		  			}
		  			
		  		}else{
		  			//只选中一部分
		  			for( let i in vm[list]){
		  				if(vm[list][i].selected){
		  				var ret = jQuery.inArray(vm[list][i].service_fee_cycle,sel_bill_arr);
				  			if(-1 == ret){
				  				sel_bill_arr.push(vm[list][i].service_fee_cycle);
				  			}	
		  					selectedArr.push(vm[list][i])
		  				}
		  			}
		  		}
		  		let bills="";
		  		let pay_addr = selectedArr[0].pay_addr;
		  		//let pay_addr = sel_bill_arr[0].pay_addr;
		  		for(let i in selectedArr){
		  			if(selectedArr.length - 1 == i){
		  				bills+=selectedArr[i].bill_id;
		  			}else{
		  				bills+=selectedArr[i].bill_id+',';	
		  			}	  			
		  		}
		  		if(vm.pay_least_month>0){
					if(vm.pay_least_month>sel_bill_arr.length){
						alert("请至少选择"+vm.pay_least_month+"个月的账单进行支付。");
						return false;
					}
				}
		  		//------
		  		vm.str = 'https://www.e-shequ.com/pay/';
		  		let url = vm.str + "qibaopaymentdetail.html?#/?billIds="+bills+"&stmtId="+vm.stmtId+"&payAddr="+escape(pay_addr)+"&totalPrice="+vm[allPrice]+"&reduceMode="+vm.reduceMode;
	            window.location.href = url;

		  	},
		  	//点击某个选中按钮 params1:被点击按钮的下标 params2:被点击按钮所属的数组
		  	itemClick:function(index,b){//3个页面对应不同的三个数组 
				  let len = b.length
				//permit_skip_pay  0允许  1不允许
			if(vm.permit_skip_pay == '1') {
					if(b[index].selected){//选中状态下
						for(let i=index;i< len;i++){
							//后面的全部取消选中
							vm.$set(b[i],'selected',false);	
						}
						//某一个点击了取消后全选消失
						vm.carAllselect = false;
					}else{//未选中状态下，前面全部选中
						for(let j=index;j >= 0;j--){//
							vm.$set(b[j],'selected',true);	
						}
					}
			}else if(vm.permit_skip_pay == '0') {
				if(b[index].selected){//选中状态下
					vm.$set(b[index],'selected',false);	
					//某一个点击了取消后全选消失
					vm.carAllselect = false;
					//    console.log(vm.bAllSelect)
				}else{//未选中状态下，前面全部选中
						vm.$set(b[index],'selected',true);	
					}
			}	
		  	},

		  	//点击全选按钮 params:需要被全部选中的数组
		  	allSelect:function(arr,a){
		  		if(vm[a] ){//取消全选
		  			for(let i in arr){
		  				vm.$set(arr[i],'selected',false);
		  			}
		  		}else{//全部选中
		  			for(let i in arr){
		  				vm.$set(arr[i],'selected',true);
		  			}
		  		}
		  		//取反
		  		vm[a] = !vm[a];
		  	},
		}
	}
</script>

<style scoped>
.mint-loadmore-bottom span {
    display: inline-block;
    -webkit-transition: .2s linear;
    transition: .2s linear;
    vertical-align: middle
}
 
.mint-loadmore-bottom span.is-rotate {
    -webkit-transform: rotate(180deg);
    transform: rotate(180deg)
}
a{color:black}
/*查询缴费*/

.query-data{
	height: 100%;
	padding: 0.25rem 1.2rem;
	margin-bottom: 0.2rem;
}
.virtual-input{
	width:3.4rem;
	border:1px solid #4c4c4c;
	border-radius: 5px;
	padding-left: 0.2rem;
	display: inline-block;
	height: 0.5rem;
}
.input-row{
	padding-bottom: 0.3rem;
	font: 0.26rem/0.5rem "";
	color: #a6937c;
	height: 0.5rem;
}
.top2{
	position: relative;
	top: -0.2rem;

}

.btn-fixed{
	position: fixed;
	color: #fff;
    left: 0;
    right: 0;
    bottom: 0;
    height: 0.92rem;
    line-height: 0.92rem;
    text-align: center;
}

.select-btn{
	padding-left: 36px;
    background: url('../../assets/img/pay/icon_unselect_white.png') no-repeat;
    background-color: rgba(0,0,0,0.6);
    background-size: 16px;
    background-position: 15px center;
    height: 0.92rem;
    line-height: 0.92rem;
}

.allSelected{
	background: url('../../assets/img/pay/icon_selected_white.png') no-repeat;
	 background-color: rgba(0,0,0,0.6);
    background-size: 16px;
    background-position: 15px center;

}

.pay{
	overflow: hidden;
	background: #ff8a00;
	text-align: center;	
}	

.main{
	margin:0 0.3rem;
}

 .mint-navbar .mint-tab-item{
 	border-bottom: 1px solid #cdcdcb;
 }

 .mint-navbar .mint-tab-item.is-selected{
 	border-bottom: 1px solid #ff8a00;
 	margin-bottom: 0;
 	color:#ff8a00;
 }
</style>