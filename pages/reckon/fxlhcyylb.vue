<!-- 超越保医疗险/安旅中国-京东安联境内旅行保险 -->
<template>
	<view class="content">
		<x-nav-bar no-border title="保费试算"></x-nav-bar>
		<view class="info">
			<view class="info-box">
				<view class="info-box-title"><text>被保人信息</text></view>
				<x-radio-box ref="planCode" @change="setPlanCode" style="margin-bottom: 24rpx;"></x-radio-box>
				<x-list-item title="被保人出生日期" tips="注：30天（含）-60周岁（含）" height="auto">
					<x-date-picker slot="right" @onConfirm="bindInsuredDateChange" :defaultValue="insuredEndDate" :startDate="insuredStartDate" :endDate="insuredEndDate">
						<text class="slot-view" v-if="insuredBirth">{{insuredBirth}}</text>
						<text class="slot-view-placeholder" v-else>请选择</text>
					</x-date-picker>
				</x-list-item>
				<x-radio-box ref="isSocial" @change="setIsSocial"></x-radio-box>
				<x-radio-box ref="allowance"></x-radio-box>
				<x-radio-box ref="deductible"></x-radio-box>
				<x-radio-box ref="institution"></x-radio-box>
			</view>
		</view>
		
		<view class="reckon">
			<view class="reckon-wrap">
				<view class="price">
					<text>{{resultPrice}}</text>
					<text class="sec-text">元/年</text>
				</view>
				<view class="btn">
					<button type="default" @click="toPay(detail.insuranceLink)">立即投保</button>
				</view>
			</view>
		</view>
		<x-modal ref="modal" title="您还未登录" cancelText="继续投保" confirmText="登录" content="加入新禧保险经纪，申请执业资质，认证通过后可享禧小保全平台推广费。" @cancel="insure(detail.insuranceLink)" @sure="sure" ></x-modal>
		<x-modal ref="modal1" title="您还未成为认证经纪人" cancelText="继续投保" confirmText="立即认证" content="加入新禧保险经纪，申请执业资质，认证通过后可享禧小保全平台推广费。" @cancel="insure(detail.insuranceLink)" @sure="certification" ></x-modal>
	</view>
</template>

<script>
	import {formatTime,getAge,setLimitTime,setEndDayTime} from '../../common/utils/index.js'
	import { mapState } from 'vuex'
	export default {
		computed: mapState(['isLogin','userinfo']),
		data() {
			return {
				
				detail:{},
				resultPrice:'— — —',
				
				//被保人
				insuredBirth: '',
				insuredStartDate: '',
				insuredEndDate: '',
				planCode:{
					name:'保障计划',
					list: [{name: "专家版",value: 1,checked: 0,show:true,disabled:false}, {name: "轻奢版",value: 2,checked: 0,show:true,disabled:false}]
				},
				isSocial:{
					name: '是否有医保',
					list: [{name: "是",value: 1,checked: 0,show:true,disabled:false}, {name: "否",value: 0,checked: 0,show:true,disabled:false}]
				},
				allowance:{
					name: '重大疾病津贴保险金',
					list: [{name: '1万',value: 10000,checked:0,show: true,disabled:true}, {name: '1.5万',value: 15000,checked: 0,show:false,disabled:true}]
				},
				deductible:{
					name: '年免赔额',
					list: [{name: '1万',value: 10000,checked:0,show: true,disabled:true}, {name: '1.5万',value: 15000,checked: 0,show:false,disabled:true}]
				},
				institution:{
					name: '医疗机构',
					list: [{name: '普通部',value: 'PT',checked:0,show: true,disabled:true}, {name: '普通部、特需部、VIP部及国际部',value: 'VIP',checked: 0,show:false,disabled:true}]
				},
				
				sendData: {
					indemnityAmount: 300000, //基本保额
					gander: '', //被保人性别
					age: '', //被保人年龄
					paymentPeriod: '', //交费期限(年)
					guaranteePeriod: '', //保障期
					planCode: '', //计划代码
					productCode: '', //产品代码
					companyCode: '', //公司代码
					exemptType: '', //豁免类型 1:被保人豁免 2：投保人豁免 空则是无豁免
					trialType: 'Client', //保费试算类型(1.Api; 2.Client)
					policyholderAge: '', //投保人年龄
					policyholderGander: '', //投保人性别
					rateType:'',
					isSocial:'' //是否有医保
				}
			}
		},
		mounted(){
			// planCode
			this.$refs.planCode.set({
				type:'radio',			
				title:this.planCode.name,
				index:0,				
				column:2,				
				list:this.planCode.list	
			});
			this.$refs.isSocial.set({
				type:'radio',			
				title:this.isSocial.name,
				index:-1,				
				column:2,				
				list:this.isSocial.list	
			});
			this.$refs.allowance.set({
				type:'radio',			
				title:this.allowance.name,
				index:-1,				
				column:2,				
				list:this.allowance.list	
			});
			this.$refs.deductible.set({
				type:'radio',			
				title:this.deductible.name,
				index:-1,				
				column:2,				
				list:this.deductible.list	
			});
			this.$refs.institution.set({
				type:'radio',			
				title:this.institution.name,
				index:-1,				
				column:1,				
				list:this.institution.list	
			});
		},
		
		onLoad(option) {
			this.insuredStartDate = setLimitTime(60,'start') //被保人最大年龄限制
			this.insuredEndDate = setEndDayTime(30) //被保人最小年龄限制
			
			let data = JSON.parse(decodeURIComponent(option.data))
			this.detail = data
			this.sendData.productCode = data.serialNumber
			this.sendData.companyCode = data.producerCode
			this.sendData.trialType = data.trialType
			this.sendData.rateType = data.rateType
			this.sendData.planCode = 1
			
		},
		methods: {
			getAmount(){/*从后台获取匹配结果*/
				this.resultPrice = '— — —'
				this.$api.post('pdc/product/premium.do',this.sendData).then(res=>{
					console.log(res)
					
					if(res.success){
						this.resultPrice = res.data
					}
				}).catch(err=>{
					console.log(err)
				})
			},
		
			setPlanCode(data){
				console.log('保障计划'+data.value) 
				let val = data.value
				switch(val){
					case 1:
						this.allowance.list.map(item=>{
							item.show = item.value == 15000 ? false : true
						})
						this.deductible.list.map(item=>{
							item.show = item.value == 15000 ? false : true
						})
						this.institution.list.map(item=>{
							item.show = item.value == 'VIP' ? false : true
						})
					break;
					default :
						this.allowance.list.map(item=>{
							item.show = item.value == 15000 ? true : false
						})
						this.deductible.list.map(item=>{
							item.show = item.value == 15000 ? true : false
						})
						this.institution.list.map(item=>{
							item.show = item.value == 'VIP' ? true : false
						})
				}
				this.sendData.planCode = val
				this.getAmount()
			},
			
			bindInsuredDateChange(e) { 
				let insuredAge = getAge(e.dateValue)
				this.insuredBirth = e.dateValue
				this.sendData.age = insuredAge
				this.getAmount()
			},
			
			setIsSocial(data){
				console.log('是否有医保'+data.value) 
				this.sendData.isSocial = data.value
				this.getAmount()
			},
			toPay(url){
			    // #ifndef H5
			    if(!this.isLogin){
			    	this.$refs.modal.open()
			        return
			    }else{
			        if(this.userinfo.reviewResultCode&&this.userinfo.reviewResultCode.code==5){
			            this.insure(url)
			        }else{
			            this.$refs.modal1.open()
			        }
			    }
			    // #endif
			    // #ifdef H5
			    this.insure(url)
			    // #endif
			    
			},
			insure(url){
			    let data = {
			        url,
			        type: this.detail.insuranceType,
			        linkType: this.detail.linkType,
			        name:this.detail.producerName
			    }
			    uni.navigateTo({
			        url: '../ebook/productnotify?data=' + encodeURIComponent(JSON.stringify(data))
			    })
			},
			sure(){
				this.openUrl('../my/login')
			},
			certification(){
			    if(this.userinfo.reviewResultCode.code){
			        this.openUrl('../join/audit?code='+this.userinfo.reviewResultCode.code+'&reason='+this.userinfo.reviewResultCode.reason)
			    }else{
			        this.openUrl('../join/identity')
			    }
			},
		}
	}
</script>

<style lang="less" scoped>
	.content{padding: 0 24rpx 170rpx;}
	.info-box{padding: 24rpx;
		background-color: #FFFFFF;margin-top: 24rpx;border-radius: 12rpx;
		.x-cont{padding: 0;
			.slot-view-placeholder{color: #999999;}
		}
		.info-box-title{position: relative;display: flex;align-items: center;margin-bottom: 48rpx;
			&::before{content: "";display: block;width: 6rpx;height: 34rpx;background: #FF5000;border-radius: 3rpx;margin-right: 8rpx;}
			>text{font-size: 36rpx;font-weight: 500;line-height: 50rpx;}
		}
	}
	.reckon{position: fixed;bottom: 0;left: 0;;width: 100%;height: 120rpx;background: #FFFFFF;border-top: 2rpx solid #EBEDF0;padding: 16rpx 24rpx;
		.reckon-wrap{display: flex;align-items: center;justify-content: space-between;
			.price{flex: 1;margin-right: 24rpx;color: #FF5000;font-size: 48rpx;
				.sec-text{font-size: 28rpx;font-weight: 400;line-height: 40rpx;margin-left: 4rpx;}
			}
			.btn{
				button{width: 232rpx;height: 88rpx;background: #FF5000;border-radius: 6rpx;font-size: 34rpx;font-weight: 500;color: #FFFFFF;line-height: 88rpx;}
			}
		}
	}
</style>
