<template>
    <div>
        <van-search v-model="value" placeholder="请输入搜索关键词" right-icon="scan"/>

        <van-row>
            <!--<van-swipe class="my-swipe" :autoplay="3000" indicator-color="white">-->
                <!--<van-swipe-item>1</van-swipe-item>-->
                <!--<van-swipe-item>2</van-swipe-item>-->
                <!--<van-swipe-item>3</van-swipe-item>-->
                <!--<van-swipe-item>4</van-swipe-item>-->
            <!--</van-swipe>-->
            <!--轮播图-->
            <van-swipe :autoplay="3000" style="height: 200px;" lazy-render>
                <van-swipe-item v-for="image in images" :key="image">
                    <img :src="image" />
                </van-swipe-item>
            </van-swipe>
            <van-col span="24">

                <van-tabs @click="onClick"
                          sticky
                          title-active-color="#E32DAB"
                          color="#E32DAB"
                          :line-width="40"
                          :line-height="2">
                    <!--商品分类标签-->
                    <van-tab v-for="index in categories.length"
                             :title="categories[index-1].name"
                             class="tab">
                        <!--商品卡片-->
                            <van-card @click="buy(index)" v-for="(item,index) in furns"
                                      :price="item.price"
                                      :desc="item.desc"
                                      :title="item.title"
                                      :thumb="item.thumb"
                            >
                                <template #tags>
                                    <van-tag v-for="tag in item.tag" color="#f2826a" style="margin-left: 5px;">
                                        {{tag.name}}
                                    </van-tag>
                                </template>
                            </van-card>
                    </van-tab>
                </van-tabs>
            </van-col>
        </van-row>
        <!--底部导航-->
        <van-tabbar v-model="active" active-color="#E32DAB" inactive-color="#000">
            <van-tabbar-item icon="home-o">首页</van-tabbar-item>
            <van-tabbar-item icon="search">查询</van-tabbar-item>
            <van-tabbar-item icon="friends-o">个人</van-tabbar-item>
        </van-tabbar>
        <!--弹出层-->
        <van-sku
                v-model="show"
                :sku="sku"
                :goods="goods"
                :hide-stock="sku.hide_stock"
                @buy-clicked="onBuyClicked"
        >
            <template #sku-actions="props">
                <div class="van-sku-actions">

                    <!--直接触发sku内部事件，通过内部事件执行onBuyClicked回调-->
                    <van-button
                            square
                            size="large"
                            type="danger"
                            @click="props.skuEventBus.$emit('sku:buy')"
                    >
                        购买
                    </van-button>
                </div>
            </template>
        </van-sku>

    </div>
</template>

<script>
    import {
        Toast,
        PullRefresh,
        Swipe,
        SwipeItem
    } from 'vant';

    export default {
        comments: {
            [PullRefresh.name]: PullRefresh,
            [Swipe.name]: Swipe,
            [SwipeItem.name]: SwipeItem
        },
        data() {
            return {
                categories: [],
                furns: [],
                show: true,
                sku: {},
                goods: {},
                active: 0,
                value: "",
                images:[
                    '../static/l01.jpg',
                    '../static/l02.jpg',
                    '../static/l03.jpg',
                    '../static/l04.jpg'
                ]
            }
        },
        mounted() {
            const _this = this
            axios.get('http://localhost:8181/furn/index').then(function (resp) {
                _this.furns = resp.data.data.furns
                _this.categories = resp.data.data.categories
            })
        },
        methods: {
            onClick(index) {
                const _this = this
                axios.get('http://localhost:8181/furn/findByCategoryType/' + this.categories[index].type)
                    .then(function (resp) {
                        _this.furns = resp.data.data
                    })
            },
            buy(index) {
                this.show = true
                const _this = this
                axios.get('http://localhost:8181/furn/findSpecsByFurnitureId/' + this.furns[index].id)
                    .then(function (resp) {
                        _this.goods = resp.data.data.goods
                        _this.sku = resp.data.data.skuVO
                    })
            },
            onBuyClicked(item) {
                this.$store.state.specsId = item.selectedSkuComb.s1
                this.$store.state.quantity = item.selectedNum
                this.$router.push('/addressList')
            }
        }
    }
</script>

<style scoped>
    .my-swipe .van-swipe-item {
        color: #fff;
        font-size: 20px;
        line-height: 150px;
        text-align: center;
        background-color: rgba(59, 62, 58, 0.4);
    }

    .tab{
        height: 726px;
    }
</style>