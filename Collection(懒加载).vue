<template>
    <section>
        <mt-header fixed style="z-index: 999" title="收藏">
            <mt-button icon="back" slot="left" @click.native="swipeMove()"></mt-button>
        </mt-header>
        <div class="topEmpty"></div>


        <div class="page-loadmore-wrapper" ref="wrapper">
            <mt-loadmore :bottom-method="loadBottom"
                         @bottom-status-change="handleBottomChange"
                         :bottom-all-loaded="allLoaded"
                         :auto-fill="false"
                         ref="loadmore">
                <ul class="page-loadmore-list">

                    <li v-for="(item,index) in Collection" :key="index" class="page-loadmore-listitem">
                            <Invofhead :cartva="item.type" :imm="item.invalid">
                            </Invofhead>
                            <Cl_good @delte="delte(item.favorite_id,index)">
                                <Invofcart
                                        :Collect_list="item.invalid=='true' ? 'Collect_list' :'Como'"
                                        :imm="item.invalid"
                                        :imgsrc="item.img"
                                        :liname="item.goods_name"
                                        :iphone="item.describe" :num="item.goods_price">
                                </Invofcart>
                            </Cl_good>
                    </li>

                </ul>
                <div slot="bottom" class="mint-loadmore-bottom">
                    <span v-show="bottomStatus !== 'loading'" :class="{ 'is-rotate': bottomStatus === 'drop' }">上拉加载更多...</span>
                    <span v-show="bottomStatus === 'loading'"></span>
                </div>
            </mt-loadmore>
        </div>

    </section>
</template>

<script type="text/javascript">
    import axios from '~/plugins/axios'
    import Cl_good from '~/components/common/Cl_good'
    import Invofcart from '~/components/mine/Invof_cart'
    import Invofhead from '~/components/mine/Invof_head'
    import { MessageBox } from 'mint-ui';

    let http = require('~/plugins/http');
    let Getaxios = http.default;
//        let Getaxios;
//        if(process.browser){
//            let http = require('~/plugins/http');
//            Getaxios = http.default;
//        }

    export default{
        data(){
            return {
                imgsrc:"",
                value:"555",
//                compter:"",//跟新页面数据
                Page:1,//懒加载页面
                Collectionmore:[],
                Collection:[],
                allLoaded: false,//若为真，则 loadBottom 不会被再次触发
                bottomStatus: '', //拉后效果
            }
        },
//        async asyncData () {
//            let { data } = await Getaxios.getData('get', 'favorites?page=1')
//            return {
//                Collection:data
//            }
//        },
        created(){
            Getaxios.getData('get', 'favorites?page=1')
                .then(res=>{
                    console.log(res.code);
                    this.Collection=res.data
                })
        },
        components: {
            axios,
            Cl_good,
            Invofcart,
            Invofhead
        },
        methods:{
            compter(page){
                Getaxios.getData('get', 'favorites?page='+page)
                    .then(res =>{
                        console.log(123);
                        console.log(res.data);
                        this.Collectionmore=res.data;
                        this.Collection=this.Collection.concat(this.Collectionmore)
                    })
            },
            swipeMove(){
                this.$router.push({path: '/mine'});
             },
            delte(item,index){
                MessageBox.confirm('确定执行此操作?').then(action => {
//                    console.log(item);
                    let favorite_id=[];
                    favorite_id.push(item)
                    console.log(favorite_id);
                    this.Collection.splice(index,1);
                    Getaxios.getData('delete', 'favorites/favorites',{favorite_id:favorite_id})
                        .then(res =>{
                            console.log(123);
                            console.log(res.data);
                        })
//                    window.location.reload()
                }).catch((error) => {});

            },
            //组件底部状态发生变化时的回调函数
            handleBottomChange(status) {
                this.bottomStatus = status;
            },
            //上拉刷新执行的方法    只加
            loadBottom() {
                let that=this;
                setTimeout(() => {
//                    console.log(this.Collection.length);
                    if ((this.Collectionmore.length%10)==0) {
//                        console.log(123);
                        this.Page+=1;
//                        console.log(this.perPage);
                        this.compter(this.Page)
                    } else {
//                        console.log(321);
                        this.allLoaded = true;
                    }
                    that.$refs.loadmore.onBottomLoaded();// 固定方法，查询完要调用一次，用于重新定位
                }, 500);
            }
        },
        head () {
            return {
                title: '收藏'
            }
        }
    }

</script>

<style type="text/css" scoped lang="less">
    .mint-header{
        height:45.8px;
        color: #333333;
        font-size: 18.1px;
        background-color: white;
    }
    .topEmpty{
        content: '';
        display: block;
        height:45.8px;
        overflow: hidden;
    }
    .minelist{

        width: 100%;
        height: 41.7px;
        position: relative;
        background-color: white;
        p{
            position: absolute;
            left: 13.9px;
            line-height: 42.7px;
            font-size: 12.5px;
            display: inline-block;
            color: #333333;
        }
        img{
            display: inline-block;
            position: absolute;
            right: 13.9px;
            width: 5.6px;
            height: 10.4px;
            top: 50%;
            margin-top: -5.2px;
        }
    }
    .f2{
        background-color: #f6f6f6;
        img{
            right: 40px;
            width: 8.7px;
            height:8.7px;
            margin-top: -4.9px;
        }
        span{
            display: inline-block;
            position: absolute;
            top: 50%;
            margin-top: -7.9px;
            right: 13.9px;
            color: #999999;
            font-size: 10.4px;
        }
    }
    .Collect_list{
        width: 100%;
        height: 105.6px;
        background-color: white;
        position: relative;
        /deep/.mint-cell-swipe-button{
            line-height: 100px;
        }
        /deep/ .mint-cell-wrapper, .mint-cell-swipe .mint-cell-left, .mint-cell-swipe .mint-cell-right{
            height: 105.6px;
        }
        .list_li{

            position: absolute;
            top: 17.4px;
            left: 13.9px;
            width: 92.4px;
            height: 69.4px;
            border: 1px solid #f1f1f1;
            display: flex;
            align-items: center;
            justify-content: center;
            img{
                width: 35.1px;
                height: 34px;
            }
        }
        .list_ri{
            position: absolute;
            top: 17.4px;
            left: 113.9px;

            h4{
                font-size: 13.9px;
                color: #333333;
            }
            p{
                margin-top: 6.4px;
                color: #666666;
                font-size: 10.45px;
            }
            .Co_money{
                margin-top: 12.4px;
                color: #eb0c1c;
                font-size: 10.4px;
                strong  {
                    font-size: 14.6px;
                }
            }
        }
    }
    .f2{
        .ff2{
            h4{
                color: #999999;
            }
            p{
                color: #999999;
            }
            .fo2{
                color: #999999;
            }
        }
    }
</style>