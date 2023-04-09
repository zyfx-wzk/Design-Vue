<template>
    <scroll-view style="min-height: 100vh;" scroll-y="true">
        <view style="width: 100%;height: 50upx;"/>
        <view v-html="content" class="content-box"/>
    </scroll-view>
</template>

<script>
export default {
    name: "article",
    data() {
        return {
            url: "",
            content: ""
        }
    },
    onLoad(e) {
        this.url = e.url;
    },
    mounted() {
        window.backArticleContent = (content) => {
            return this.backArticleContent(content);
        }
        //初始化页面数据
        jsFunction.postArticleContent(this.url);
    },
    methods: {
        //请求文章信息
        postArticleContent() {
            jsFunction.postArticleContent(this.url);
        },
        //返回文章信息
        backArticleContent(content) {
            this.content = unescape(content);
        },
        tolower() {

        }
    }
}
</script>

<style scoped>
.content-box {
    width: 100%;
    height: 100%;
    overflow: hidden;
}

.content-box >>> img {
    width: 100%;
    height: auto;
}
</style>