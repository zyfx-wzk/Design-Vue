<template>
    <view class="container" @click="conClick" @touchstart="refreshStart"
          @touchmove="refreshMove" @touchend="refreshEnd">
        <!-- 刷新组件需搭配scroll-view使用，并在pages.json中添加 "disableScroll":true-->
        <refresh ref="refresh" @isRefresh='isRefresh'></refresh>
        <!-- 点击反馈组件 -->
        <clickCircle ref="clickCircle"></clickCircle>
        <view class='nav'>
            <!-- #ifdef H5 -->
            <view style="height: 44px;width: 100%;">边距盒子</view>
            <!-- #endif -->
            <!-- 导航栏 agents导航栏标题 -->
            <navTab ref="navTab" :tabTitle="tabTitle[curIndex]" @changeTab='changeTab'></navTab>
        </view>
        <!-- 回到顶部悬浮按钮 -->
        <movable-area style="height: 100vh; width: 100vw;position: absolute;bottom: 0;">
            <movable-view class="addBtnBox" style="padding-top:250upx;padding-bottom:180upx;"
                          :x="isX" :y="isY" direction="all">
                <view class="addBtn" @click="changeIndex">切换</view>
            </movable-view>
            <movable-view class="addBtnBox" style="padding-top:100upx;padding-bottom:50upx;"
                          @touchmove.stop="" :x="isX" :y="isY" direction="all">
                <view class="addBtn" @click="toTop">TOP</view>
            </movable-view>
        </movable-area>
        <!-- swiper切换 swiper-item表示一页 scroll-view表示滚动视窗 -->
        <swiper style="min-height: 100vh;" :current="currentTab" @change="swiperTab">
            <swiper-item v-for="(listItem,listIndex) in list" :key="listIndex">
                <scroll-view style="height: 100%;" scroll-y="true" @scrolltolower="lower"
                             scroll-with-animation :scroll-into-view="toView">
                    <view :id="'top'+listIndex" style="width: 100%;height: 120upx;">顶部占位盒子</view>
                    <view class='content'>
                        <view class='card' v-for="(item,index) in listItem" v-if="listItem.length > 0"
                              :key="index" v-on:click="conClickItem(item)">
                            <div class="text-content">{{ item.title }}</div>
                            <div class="time-content">
                                <p style="float: right;">{{ item.time }}</p>
                            </div>
                        </view>
                    </view>
                    <view class='noCard' v-if="listItem.length===0">
                        暂无信息
                    </view>
                    <view style="width: 100%;height: 100upx;opacity:0;">底部占位盒子</view>
                </scroll-view>
            </swiper-item>
        </swiper>
    </view>
</template>

<script>
const util = require('../../util/util.js');
import refresh from '../../components/refresh.vue';
import navTab from '../../components/navTab.vue';
import clickCircle from '../../components/clickCircle.vue';

export default {
    components: {
        refresh,
        navTab,
        clickCircle,
    },
    data() {
        return {
            curIndex: 0,//现在的学校
            currentPage: 'index',
            currentTab: 0, //sweiper所在页
            toView: '', //回到顶部id
            isX: 0, //放在store统一管理
            isY: 999, //放在store统一管理
            tabTitle: [
                ['聚集天大', '综合新闻', '校内新闻'],
                ['公告', '新闻', '快讯']], //导航栏格式 --导航栏组件
            size: [], //记录下每个页面的已有数据
            list: [] //数据格式
        };
    },
    onLoad(e) {
    },
    onShow() {
    },
    onHide() {
    },
    mounted() {
        window.backPageListInfo = (info) => {
            return this.backPageListInfo(info);
        }
        window.backCurInfoList = (info) => {
            return this.backCurInfoList(info);
        }
        //初始化页面数据
        this.changeReload(true);
        uni.showLoading({
            title: '初始化数据中',
            mask: true
        })
    },
    methods: {
        changeIndex() {
            uni.showActionSheet({
                itemList: ["天津大学", "天津仁爱学院"],
                success: ((res) => {
                    this.curIndex = res.tapIndex;
                    this.changeReload(true);
                    uni.showLoading({
                        title: '初始化数据中',
                        mask: true
                    })
                }),
                fail: ((res) => {
                })
            });
        },
        changeReload(init) {
            let title = "";
            if (this.curIndex === 0) {
                title = "天津大学"
            } else if (this.curIndex === 1) {
                title = "天津仁爱学院"
            }
            uni.setNavigationBarTitle({
                title: title
            })
            //请求基础页面信息
            this.postPageListInfo(init);
        },
        postPageListInfo(init) {
            jsFunction.postPageListInfo(this.curIndex, init);
        },
        //返回基础页面信息
        backPageListInfo(info) {
            info = unescape(info);
            let list = JSON.parse(info);
            for (const obj of list) {
                //vue无法直接监听过于复杂的数据结构的改变
                this.list[obj.index] = obj.info;
                this.size[obj.index] = obj.info.length;
            }
            uni.hideLoading();
            this.$forceUpdate();//数据结构比较复杂,双向绑定无法监听,手动刷新页面
        },
        postCurInfoList() {
            jsFunction.postCurInfoList(this.currentTab, this.size[this.currentTab]);
        },
        backCurInfoList(info) {
            let obj = JSON.parse(info);
            uni.hideLoading()
            if (obj.hasOwnProperty("info")) {
                uni.showToast({
                    icon: 'none',
                    title: '数据刷新成功',
                    duration: 1000
                })
                this.list[obj.index] = this.list[obj.index].concat(obj.info);
                this.size[obj.index] = this.list[obj.index].length;
            } else {
                uni.showToast({
                    icon: 'none',
                    title: '没有更多数据',
                    duration: 1000
                })
            }
            this.$forceUpdate();
        },

        // swiper 滑动
        swiperTab: function (e) {
            let index = e.detail.current //获取索引
            this.$refs.navTab.longClick(index);
        },
        // 加载更多 util.throttle为防抖函数
        lower: util.throttle(function (e) {
            uni.showLoading({
                title: '加载中',
                mask: true
            })
            this.postCurInfoList();
        }, 300),

        //点击文章事件
        conClickItem(item) {
            uni.navigateTo({
                url: 'article?url=' + item.url,
            });
        },

        toTop() {
            this.toView = ''
            setTimeout(() => {
                this.toView = 'top' + this.currentTab
            }, 10)
        },
        changeTab(index) {
            this.currentTab = index
        },
        //点击反馈事件
        conClick(e) {
            this.$refs.clickCircle.conClick(e);
        },
        // 刷新touch监听
        refreshStart(e) {
            this.$refs.refresh.refreshStart(e);
        },
        refreshMove(e) {
            this.$refs.refresh.refreshMove(e);
        },
        refreshEnd(e) {
            this.$refs.refresh.refreshEnd(e);
        },
        isRefresh() {
            this.changeReload(false);
            setTimeout(() => {
                uni.showToast({
                    icon: 'success',
                    title: '刷新成功'
                })
                this.$refs.refresh.endAfter() //刷新结束调用
            }, 1000)
        }
    }
};
</script>

<style lang="scss">
.addBtnBox {
    position: fixed;
    z-index: 900;
    width: 140upx;
    height: 50upx;
    align-items: flex-end;
    justify-content: center;
    display: flex;

    .addBtn {
        width: 100upx;
        height: 100upx;
        border-radius: 50%;
        color: white;
        font-size: 10px;
        font-weight: bold;
        background: #444857;
        line-height: 100upx;
        text-align: center;
        box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
    }
}

.container {
    width: 100vw;
    font-size: 28upx;
    min-height: 100vh;
    overflow: hidden;
    color: #6B8082;
    position: relative;
    background-color: #f6f6f6;
}

.content {
    width: 100%;
}

.card {
    width: 90%;
    min-height: 120upx;
    margin: 0 auto 20upx auto;
    background: #FFFFFF;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    box-shadow: 0 0 5px 0 rgba(0, 0, 0, 0.10);
    border-radius: 5px;
    position: relative;
}

.noCard {
    width: 90%;
    height: 120upx;
    margin: auto;
    background-color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #999999;
    box-shadow: 0 0 10upx 0 rgba(0, 0, 0, 0.10);
    border-radius: 5upx;
}

.text-content {
    width: 90%;
}

.time-content {
    width: 90%;
}

.nav {
    position: fixed;
    left: 0;
    top: 0;
    color: white;
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: flex-start;
    font-size: 24upx;
    background-color: #50B7EA;
    z-index: 996;
}

.searchInput999 {
    width: 90%;
    margin: 0 auto;
    background: white;
    border-radius: 30upx;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 56upx;
}

.search999 {
    width: 32upx;
    height: 32upx;
}

.searchBox999 {
    width: 56upx;
    height: 56upx;
    display: flex;
    justify-content: center;
    align-items: center;
}

.input999 {
    color: #999;
    width: 80%;
}
</style>