<template>
	<view class="content">
        <x-nav-bar v-if="isShow" :no-border="true" title="禧小保" leftWidth="200rpx" :rightWidth="isShow" >
            <view slot="right" class="top-right" @click="openUrl('/pages/home/index',2)">回到首页</view>
        </x-nav-bar>
        <x-nav-bar v-else :no-border="true" title="禧小保"></x-nav-bar>
		<web-view :webview-styles="webviewStyles" :src="url" @message="getMsg"></web-view>
	</view>
</template>

<script>
	import { mapState } from 'vuex'
	export default {
		data(){
			return {
                url: '',
                webviewStyles: {
                    progress: {
                        color: '#FF5000'
                    }
                },
                isShow: false
			}
		},
        computed:{
            ...mapState(['userinfo','isLogin', 'statusBarHeight']),
        },
        onLoad(option) {
            if (option.url) {
                this.url = decodeURIComponent(option.url)
            } else {
                this.showToast('参数错误')
                setTimeout(() => {
                    this.backPage()
                }, 2000)
            }
            if(option.num==1){
                this.isShow = true
            }
        },
		onReady() {
            // #ifdef APP-PLUS
            var currentWebview = this.$mp.page.$getAppWebview() //获取当前页面的webview对象
            setTimeout(() => {
                let wv = currentWebview.children()[0]
                wv.setStyle({top: uni.getSystemInfoSync().statusBarHeight+44, bottom: uni.getSystemInfoSync().windowBottom})
            }, 500) //如果是页面初始化调用时，需要延时一下
            // #endif
		},
		methods:{
            getMsg (e) {
                console.log(e)
            }
		}
	}
</script>

<style lang="less" scoped>
    .top-right{text-align: center; margin: 0 auto;}
</style>
