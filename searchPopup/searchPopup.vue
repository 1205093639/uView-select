<template>
    <div class='searchPopup'>
        <u-popup :show="show" @close="close" mode="bottom" round="20">
            <div class="popup">
                <u-search placeholder="请输入搜索关键字" @change="onSearch" :showAction="scanAbled" :clearabled="true" v-model="keyword" @custom="scan" actionText="扫码"></u-search>
                <scroll-view scroll-y class="scroll-Y" @scrolltolower="scrolltolower">
                    <u-cell-group :border="false">
                        <u-cell clickable @click="getItem(item)" v-for="(item, index) in list" :key="index">
                            <div class="item title" v-if="title" slot="title">{{item[title]}}</div>
                            <div class="item label" v-if="label" slot="label">{{item[label]}}</div>
                            <div class="item value" v-if="value" slot="value">{{item[value]}}</div>
                        </u-cell>
                    </u-cell-group>
                    <u-loadmore :status="status" @loadmore="scrolltolower" loadmoreText="点击加载更多" />
                    <div class="placeholder"></div>
                </scroll-view>
            </div>
        </u-popup>
    </div>
</template>
<script>
export default {
    name: 'searchPopup',
    props: {
        scanAbled: {//是否能够扫码
            type: Boolean,
            default: false,
        },
        show: {
            type: Boolean,
            default: false,
        },
        getFirst: {//是否得到数据后自动选取第一条
            type: Boolean,
            default: false
        },
        apiName: {
            type: String,
            default: ''
        },
        currentItem: {
            type: Object,
            default: () => { return {} }
        },
        title: {
            type: String,
            default: ''
        },
        label: {
            type: String,
            default: 'label'
        },
        value: {
            type: String,
            default: 'value'
        },
        inputValue: {
            type: [String, Array],
            default: ''
        },
        otherData: {
            type: Object,
            default: () => { return {} }
        }
        //@confirm 确认事件


    },
    data() {
        return {
            hasFirst: false,
            keyword: '',
            page: 1,
            pageRow: 10,
            list: [],
            status: 'loadmore',
        }
    },
    methods: {

        scan() {//扫码
            console.log('扫描');
            // 允许从相机和相册扫码
            let _this = this
            uni.scanCode({
                success: function (res) {
                    console.log('条码类型：' + res.scanType);
                    console.log('条码内容：' + res.result);
                    _this.keyword = res.result
                    this.onSearch()
                }
            });
        },
        scrolltolower() {
            console.log('触底');
            if (this.status == 'nomore') return
            this.status == this.getData()
            console.log('触底加载');
        },
        onSearch() {
            this.status = 'loadmore';
            console.log('触发search');
            this.page = 1;
            this.list = []
            this.getData()
        },
        close() {
            this.$emit('update:show', false);
        },
        getData() {
            let data = {
                keyword: this.keyword,
                page: this.page,
                pageRow: this.pageRow,
                ...this.otherData,
            }
            if (this.status == 'loading' || this.status == 'nomore') return
            this.status = 'loading';
            this.$apis[this.apiName](data).then(res => {
                let resData = res?.RetData1?.data || res?.RetData1 || res?.data || res
                resData.forEach((e, i) => {
                    for (const key in e) {
                        e[key.toLowerCase()] = e[key]
                    }
                })
                if (resData.length) {
                    this.list.push(...resData)
                    if (this.getFirst && this.page == 1 && !this.hasFirst && !this.inputValue) this.getItem(this.list[0])
                    this.page++
                    this.status = 'loadmore';
                    if (resData.length < this.pageRow) this.status = 'nomore';
                } else {
                    this.status = 'nomore';
                }
            }).catch(err => {
                console.error(err);
                this.status = 'nomore';
            })
        },
        getItem(item) {
            if (Object.keys(this.currentItem).length) this.$emit('update:currentItem', item)
            this.hasFirst = true
            this.$emit('confirm', item)
            this.close()
        },

    },
    mounted() {
        this.apiName && this.onSearch()
    }
}
</script>
<style lang='scss' scoped>
.searchPopup {
    .popup {
        padding: 20px;
        .scroll-Y {
            height: 50vh;
        }
    }
    .placeholder {
        height: 20rpx;
    }
}
</style>