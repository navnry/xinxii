<!-- 平安e生保2020升级 -->
<template>
	<view class="content">
		<x-nav-bar no-border title="保费试算"></x-nav-bar>
		<view class="info">
			<view class="info-box">
				<view class="info-box-title"><text>被保人信息</text></view>
				<x-list-item title="被保人出生日期" tips="注：28天（含）-55周岁（含）" height="auto">
					<x-date-picker slot="right" @onConfirm="bindInsuredDateChange" :defaultValue="insuredEndDate" :startDate="insuredStartDate" :endDate="insuredEndDate">
						<text class="slot-view" v-if="insuredBirth">{{insuredBirth}}</text>
						<text class="slot-view-placeholder" v-else>请选择</text>
					</x-date-picker>
				</x-list-item>
				<x-radio-box ref="gander" @change="setGander"></x-radio-box>
				<x-radio-box ref="isSocial" @change="setIsSocial"></x-radio-box>
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
				
				gander:{
					name: '被保人性别',
					list: [{name: "男",value: 1,checked: 0,show:true,disabled:false}, {name: "女",value: 0,checked: 0,show:true,disabled:false}]
				},
				isSocial:{
					name: '是否有医保',
					list: [{name: "是",value: 1,checked: 0,show:true,disabled:false}, {name: "否",value: 0,checked: 0,show:true,disabled:false}]
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
			
			this.$refs.gander.set({
				type:'radio',			
				title:this.gander.name,
				index:-1,				
				column:2,				
				list:this.gander.list	
			});
			this.$refs.isSocial.set({
				type:'radio',			
				title:this.isSocial.name,
				index:-1,				
				column:2,				
				list:this.isSocial.list	
			});
		},
		
		onLoad(option) {
			this.insuredStartDate = setLimitTime(55,'start') //被保人最大年龄限制
			this.insuredEndDate = setEndDayTime(28) //被保人最小年龄限制
			
			let data = JSON.parse(decodeURIComponent(option.data))
			this.detail = data
			this.sendData.productCode = data.serialNumber
			this.sendData.companyCode = data.producerCode
			this.sendData.trialType = data.trialType
			this.sendData.rateType = data.rateType
			
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
		
			
			bindInsuredDateChange(e) { //被保人年龄限制、保费区间
				let insuredAge = getAge(e.dateValue)
				this.insuredBirth = e.dateValue
				this.sendData.age = insuredAge
				this.getAmount()
			},
			
			setGander(data){
				console.log('被保人性别'+data.value) 
				this.sendData.gander = data.value
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
