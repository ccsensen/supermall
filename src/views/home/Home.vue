<template>
    <div id="home">
        <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
        <scroll class="content"
                ref="scroll"
                @scroll="contentScroll"
                @pullingUp="loadMore"
                :probe-type="3"
                :pull-up-load="true">
            <home-swiper :banners="banners" />
            <recommend-view :recommends="recommends" />
            <feature-view />
            <tab-control class="tab-control" :titles="['流行','新款','精选']" @tabClick="tabClick"/>
            <goods-list :goods="showGoods"></goods-list>
        </scroll>
        <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>

    </div>
</template>

<script>
    import NavBar from 'components/common/navbar/NavBar'
    import TabControl from 'components/content/tabControl/TabControl'
    import GoodsList from 'components/content/goods/GoodsList'
    import Scroll from 'components/common/scroll/Scroll'
    import BackTop from 'components/content/backTop/BackTop'

    import {getHomeMultidata,getHomeGoods} from 'network/home'

    import RecommendView from './childComps/RecommendView'
    import HomeSwiper from './childComps/HomeSwiper'
    import FeatureView from './childComps/FeatureView'


    export default {
        name: "Home",
        components:{
            NavBar,
            TabControl,
            GoodsList,
            Scroll,
            BackTop,
            RecommendView,
            HomeSwiper,
            FeatureView
        },
        data(){
            return {
                banners: [],
                recommends: [],
                goods: {
                    'pop': {page:0,list:[]},
                    'new': {page:0,list:[]},
                    'sell': {page:0,list:[]}
                },
                currentType: 'pop',
                isShowBackTop: false
            }
        },
        created(){
            //请求多个数据
            this.getHomeMultidata();

            this.getHomeGoods('pop');
            this.getHomeGoods('new');
            this.getHomeGoods('sell');
        },
        computed:{
          showGoods(){
              return this.goods[this.currentType].list;
          }
        },
        methods:{
            tabClick(index){
                switch(index){
                    case 0:
                        this.currentType = 'pop';
                        break;
                    case 1:
                        this.currentType = 'new';
                        break;
                    case 2:
                        this.currentType = 'sell';
                        break;
                }
            },
            contentScroll(position){
                this.isShowBackTop = (-position.y) > 1000;
            },
            loadMore(){
                this.getHomeGoods(this.currentType);
            },
            backClick(){
                this.$refs.scroll.scrollTo(0,0);
            },
            //网络请求的方法
            getHomeMultidata(){
                getHomeMultidata().then(res => {
                    this.banners = res.data.banner.list;
                    this.recommends = res.data.recommend.list;
                })
            },
            getHomeGoods(type){
                const page = this.goods[type].page + 1;
                getHomeGoods(type,page).then(res => {
                    this.goods[type].list.push(...res.data.list);
                    this.goods[type].page += 1;
                    this.$refs.scroll.finishPullUp();
                })
            }


        }
    }
</script>

<style scoped>
    #home{
        position: relative;
        height: 100vh;
    }
    .home-nav{
        background-color: var(--color-tint);
        color: #fff;

        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        z-index: 9;
    }
    .tab-control{
        position: sticky;
        top: 44px;
        z-index: 9;
    }
    .content{
        position: absolute;
        top: 44px;
        bottom: 49px;
        left: 0;
        right: 0;
        overflow: hidden;
    }
</style>