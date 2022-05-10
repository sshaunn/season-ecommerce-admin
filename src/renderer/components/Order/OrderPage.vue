<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item>订单列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="content-main">
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="待付款" name="first"></el-tab-pane>
                <el-tab-pane label="待确认" name="second"></el-tab-pane>
                <el-tab-pane label="待收货" name="third"></el-tab-pane>
                <el-tab-pane label="已收货" name="fourth"></el-tab-pane>
                <el-tab-pane label="全部订单" name="sixth"></el-tab-pane>
            </el-tabs>
            <div class="filter-box">
                <el-form :inline="true" :model="filterForm" class="demo-form-inline">
                    <el-form-item label="收货人姓名">
                        <el-input class="filter-input" v-model="filterForm.contactName" placeholder="请输入姓名"></el-input>
                    </el-form-item>
                    <el-form-item label="收货人电话">
                        <el-input class="filter-input" v-model="filterForm.contactNumber"
                                  placeholder="请输入电话"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmitFilter">查询</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <div class="form-table-box">
                <div v-for="item in tableData" v-bind:key="item.id" class="list-wrap clearfix" >
                    <div class="header clearfix">
                        <div class="left">
                            <div class="order-id" style="margin-right: 10px">#{{ item.id }}</div>
                            <div class="off-text" v-if="item.offline_pay=== 1">线下支付订单</div>
                            <div class="on-text" v-if="item.offline_pay=== 0">线上支付订单</div>
                            <div class="pickup-text" v-if="item.is_pickup === 1">自提订单</div>
                            <div class="delivery-text" v-if="item.is_pickup === 0">配送订单</div>
                            <div class="status-text">{{ getStatusLabelFromNumber(item.order_status) }}</div>
                            <div class="add-time">下单时间：{{ convertTime(item.create_time) }}</div>
                            <div class="pay-time" v-if="item.pay_time">付款时间：{{ convertTime(item.pay_time) }}</div>
                        </div>
                        <div class="right">
                            <div class="goods-num">共{{ item.goodsCount }}件商品</div>
                            <div class="price-wrap">当前合计: ${{ item.total_price }} 澳元（含运费: ${{ item.freight_price }}
                                澳元）
                            </div>
                        </div>
                    </div>
                    <div class="content-wrap clearfix">
                        <div class="left">
                            <div class="goods-list" v-for="iitem in item.orderGoods" v-bind:key="iitem.id">
                                <img :src="iitem.list_pic_url" class="goods-img">
                                <div class="goods-name">{{ iitem.goods_name }}({{ iitem.goods_aka }})</div>
                                <div class="goods-number"><label>数量：</label>{{ iitem.number }}</div>
                            </div>
                        </div>
                        <div class="user-wrap">
                            <div class="avatar-wrap">
                                <img :src="item.userInfo.avatar" class="avatar-img">
                                <div class="nickname">{{ item.userInfo.nickname }}</div>
                            </div>
                        </div>
                        <div class="main">
                            <div class="m1">
                                <div class="user-name">{{ item.delivery_contact_name }}</div>
                                <div class="user-mobile">{{ item.delivery_contact_number }}</div>
                            </div>
                            <div class="user-address" v-if="item.is_pickup=== 1">
                                {{ item.pickup_address }}, {{ item.pickup_suburb }}, {{ item.pickup_postcode }}
                            </div>
                            <div class="user-address" v-if="item.is_pickup=== 0">
                                {{ item.delivery_address }}, {{ item.delivery_suburb }}, {{
                                    item.delivery_postcode
                                }}
                            </div>
                            <div v-if="item.postscript != ''" class="user-post">留言：{{ item.postscript }}</div>
                            <el-input class="admin-memo" type="textarea" @change="changeMemo(item.id,item.admin_memo)"
                                      v-model="item.admin_memo"
                                      placeholder="备注"></el-input>
                        </div>
                        <div class="wechatPic" v-if="item.transfer_screenshot_url">
                           <img @click="showLargePic(item)"
                                 v-if="item.transfer_screenshot_url"
                                 :src="item.transfer_screenshot_url"
                                 class="image-show">
                        </div>
                        <div class="wechatPic" v-if='!item.transfer_screenshot_url'>
                            <el-tag type="warning">转账截图尚未上传</el-tag>
                        </div>
                        <div class="right">
                            <el-button v-if="item.order_status === 301" size="small"
                                       @click="orderPrint(item)">打印帐单
                            </el-button>
                            <el-button class="right-detail" type="primary" plain size="small"
                                       @click="viewDetail(item.id)">查看详情
                            </el-button>
                            <el-button class="right-detail" style="margin-top: 10px" type="danger" plain size="small"
                                       @click="changeStatus(item.id)">变更状态
                            </el-button>

                        </div>
                    </div>
                </div>
            </div>
            <div class="page-box">
                <el-pagination @current-change="handlePageChange" :current-page="page" :page-size="10"
                               layout="total, prev, pager, next, jumper" :total="total">
                </el-pagination>
            </div>
        </div>
        <el-dialog :visible.sync="dialogFormVisible">
            <div class="dialog-wrap" ref="print" id="print-content">
                <h3 style="margin-bottom: 10px">订单：#{{ selectedOrder.id }}
                    {{ convertTime(selectedOrder.create_time) }}</h3>
                <div class="dialog-main">
                    <el-form :inline="true">
                        <el-form-item label="收货人姓名：">{{ selectedOrder.delivery_contact_name }}
                        </el-form-item>
                        <el-form-item label="手机：">{{ selectedOrder.delivery_contact_number }}
                        </el-form-item>
                        <el-form-item label="地址: ">{{ selectedOrder.print_address }}

                        </el-form-item>
                    </el-form>
                </div>
                <div class="list-wrap">
                    <div class="goods-list" v-for="ditem in selectedOrder.orderGoods" v-bind:key="ditem.id">
                        <div class="goods-name">{{ ditem.goods_name }}</div>
                        <div class="goods-spec">{{ ditem.goods_specifition_name_value }}</div>
                        <div class="goods-number"><label>数量：</label>{{ ditem.number }}</div>
                        <div class="goods-number"><label>单价：</label>${{ ditem.retail_price }}</div>
                        <div class="goods-number"><label>小计：</label>${{ ditem.retail_price * ditem.number }}</div>
                    </div>
                    <div class="total-price">合计：${{ selectedOrder.total_price }}（含运费：${{
                            selectedOrder.freight_price
                        }})
                    </div>

                </div>
                <div v-if="selectedOrder.postscript !== ''" class="user-post-t">买家留言：{{
                        selectedOrder.postscript
                    }}
                </div>
                <div v-if="selectedOrder.admin_memo !== ''" class="user-admin-t">备注：{{ selectedOrder.admin_memo }}</div>
            </div>

            <div slot="footer" class="dialog-footer print-footer">
                <el-button style="margin-right: 20px;" @click="dialogFormVisible = false">关闭</el-button>
                <el-button type="primary" @click="confirmPrint">打印</el-button>
            </div>
        </el-dialog>
        <el-dialog title="变更状态"
                   :visible.sync="statusVisible"
                   width="30%">
            <el-form :model="statusData">
                <el-form-item label="状态:" label-width="120px">
                    <el-select class="el-select-class" v-model="statusValue"
                               placeholder="选择状态">
                        <!---->
                        <el-option
                            v-for="item in statusList"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="statusVisible = false">取 消</el-button>
                <el-button type="primary" @click="statusConfirm">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog :visible.sync="picDialogVisible" center>
                <span slot="footer" class="dialog-footer">
                    <img class="image-show-large"
                         v-if="wechatPic"
                         :src="wechatPic">
                </span>
        </el-dialog>
    </div>

</template>

<script>
import VueBarcode from '../../../../node_modules/vue-barcode';
import ElButton from "../../../../node_modules/element-ui/packages/button/src/button.vue";
import api from '@/config/api';
import moment from 'moment';
import _ from 'lodash'
// Vue.component(VueBarcode.name, VueBarcode);

export default {
    data() {
        return {
            wechatPic: '',
            imgUrl: '',
            picDialogVisible: false,
            autoGoDelivery: true,
            sfHasValue: {},
            barcodeValue: 'test',
            printMiandan: false,
            rawHtml: '',
            testApi: 'http://sandboxapi.kdniao.com:8080/kdniaosandbox/gateway/exterfaceInvoke.json',
            checkAll: false,
            isIndeterminate: true,
            page: 1,
            total: 0,
            filterForm: {
                contactName: '',
                contactNumber: '',
            },
            tableData: [],
            activeName: 'second',
            order_status: 201,
            dialogVisible: false,
            dialogVisible2: false,
            dialogFormVisible: false,
            dialogPriceVisible: false,
            dialogText: '',
            dialogIndex: 0,
            order_sn: 0,
            order_id: 0,
            selectedOrder: {},
            dform: {
                method: 1,
            },
            orderInfo: {},
            isShow: true,
            deliveryCom: [],
            nowDeliveryId: '',
            formLabelWidth: "120px",
            senderInfo: {},
            receiverInfo: {},
            dialogExpressVisible: false,
            rePrintStatus: 0,
            changeSender: 0,
            changeReceive: 0,
            options: [],
            senderOptions: [],
            receiveOptions: [],
            receiver: {},
            sender: {},
            statusList: [
                {
                    value: '101',
                    label: '待付款'
                },
                {
                    value: '201',
                    label: '待确认'
                },
                {
                    value: '301',
                    label: '待收货'
                },
                {
                    value: '401',
                    label: '已收货'
                },
            ],
            statusVisible: false,
            statusValue: '',
            statusData: {},
            statusForm: {
                id: 0,
            },
        }
    },
    methods: {
        showLargePic(item) {
            this.picDialogVisible = true;
            this.wechatPic = item.transfer_screenshot_url;
        },
        convertTime(time) {
            return moment(time * 1000).format('YYYY-MM-DD HH:mm:ss')
        },
        getStatusLabelFromNumber(number) {
            let result = ''
            this.statusList.forEach(item => {
                if (item.value === number.toString()) {
                    result = item.label;
                }
            })
            return result;
        },
        changeStatus(id) {
            this.statusVisible = true;
            this.statusForm.id = id;
        },
        statusConfirm() {
            const sendData = {
                order_id: this.statusForm.id,
                order_status: Number(this.statusValue),
            };
            this.axios.put('order_au/updateOrder', sendData).then((response) => {
                this.statusVisible = false;
                this.getList();
            })
        },
        changeMemo(id, text) {
            this.axios.put('order_au/updateOrder', {
                admin_memo: text,
                order_id: id
            }).then((response) => {
                if (response.data.errno === 0) {
                    this.$message({
                        type: 'success',
                        message: '保存成功!'
                    });
                } else {
                    this.$message({
                        type: 'error',
                        message: '保存失败'
                    })
                }
            })
        },
        viewDetail(index) {
            this.$router.push({name: 'order_detail', query: {id: index}})
        },
        handleClick(tab, event) {
            let pindex = tab._data.index;
            if (pindex == 0) {
                this.order_status = '101'
            } else if (pindex == 1) {
                this.order_status = '201'
            } else if (pindex == 2) {
                this.order_status = '301'
            } else if (pindex == 3) {
                this.order_status = '401'
            } else if (pindex == 4) {
                this.order_status = '100'
            }
            this.getList();
        },
        handlePageChange(val) {
            this.page = val;
            //保存到localStorage
            localStorage.setItem('orderPage', this.page)
            localStorage.setItem('orderFilterForm', JSON.stringify(this.filterForm));
            this.getList()
        },
        onSubmitFilter() {
            this.page = 1
            this.axios.get('order_au', {
                params: {
                    page: this.page,
                    status: Number(this.order_status),
                    contact_name: this.filterForm.contactName,
                    contact_number: this.filterForm.contactNumber
                }
            }).then((res) => {
                this.tableData = res.data.data.data;
                this.total = res.data.data.count;
            })
        },
        getList() {
            this.axios.get('order_au', {
                params: {
                    page: this.page,
                    contact_name: this.filterForm.contactName,
                    contact_number: this.filterForm.contactNumber,
                    status: this.order_status,
                }
            }).then((response) => {
                this.tableData = response.data.data.data;
                this.total = response.data.data.count;
            })
        },
        orderPrint(item) {
            this.selectedOrder = item;
            this.selectedOrder.print_address = '';
            this.selectedOrder.is_pickup === 1 ?
                this.selectedOrder.print_address = `${this.selectedOrder.pickup_address}, ${this.selectedOrder.pickup_suburb}, ${this.selectedOrder.pickup_postcode}` :
                this.selectedOrder.print_address = `${this.selectedOrder.delivery_address}, ${this.selectedOrder.delivery_suburb}, ${this.selectedOrder.delivery_postcode}`;
            this.dialogFormVisible = true;
        },
        confirmPrint() {
            this.$print(this.$refs.print)
        },
    },
    components: {ElButton, 'barcode': VueBarcode},

    mounted() {
        this.imgUrl = api.url + 'api/fileUpload/upload';
        this.getList();
    }
}

</script>

<style scoped>

.filter-input {
    width: 200px !important;
}

.print-footer {
    display: flex;
    justify-content: flex-end;
}


.filter-box {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}


.demo-form-inline {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.list-wrap {
    width: 100%;
    border: 1px solid #dfe5ed;
    margin-bottom: 10px;
}

.goods-img {
    width: 40px;
    height: 40px;
}

.list-wrap .header {
    width: 100%;
    height: 40px;
    background-color: rgba(236, 245, 255, 0.51);
    line-height: 40px;
    color: #1f2d3d;
    font-size: 13px;
    padding: 0 10px;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.header .left {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}

.header .right {
    display: flex;
    justify-content: flex-end;
    align-items: center;
}


.off-text {
    color: #fff;
    border-radius: 4px;
    background: #594d72;
    height: 15px;
    line-height: 15px;
    padding: 4px 10px;
    font-size: 12px;
    margin-right: 10px;
}

.on-text {
    color: #fff;
    border-radius: 4px;
    background: #62033c;
    height: 15px;
    line-height: 15px;
    padding: 4px 10px;
    font-size: 12px;
    margin-right: 10px;
}

.pickup-text {
    color: #fff;
    border-radius: 4px;
    background: #48a59e;
    height: 15px;
    line-height: 15px;
    padding: 4px 10px;
    font-size: 12px;
    margin-right: 10px;
}

.delivery-text {
    color: #fff;
    border-radius: 4px;
    background: #212fb0;
    height: 15px;
    line-height: 15px;
    padding: 4px 10px;
    font-size: 12px;
    margin-right: 10px;
}

.status-text {
    color: #f0797f;
    margin-right: 10px;
}

.add-time {
    margin-right: 20px;
}

.pay-time {
    margin-right: 20px;
}

.goods-num {
    margin-right: 20px;
}

.price-wrap {
    float: right;
    margin-right: 20px;
}

.content-wrap {
    width: 100%;
    display: flex;
    justify-content: flex-start;
}

.content-wrap .left {
    width: 30%;
    border-right: 1px solid #d1dbe5;
    padding: 20px 10px;
}

.content-wrap .user-wrap {
    width: 16%;
    border-right: 1px solid #d1dbe5;
    display: flex;
    flex-direction: column;
    padding: 20px 10px;
}

.content-wrap .user-wrap .avatar-wrap {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    margin-bottom: 10px;
}

.content-wrap .user-wrap .avatar-wrap .avatar-img {
    width: 40px;
    height: 40px;
    border-radius: 100px;
    margin-right: 10px;
}

.content-wrap .user-wrap .avatar-wrap .nickname {
    font-size: 14px;
}


.content-wrap .main {
    width: 36%;
    border-right: 1px solid #d1dbe5;
    padding: 20px 10px;
}

.content-wrap .right {
    width: 12%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}

.right .right-detail {
    margin-left: 0;
    margin-top: 6px;
}

.content-wrap .wechatPic {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 15%;
    border-right: 1px solid #d1dbe5;
    padding: 20px 10px;
    text-align: center;
}

.goods-list {
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid #f1f1f1;
    padding: 6px 0;
    align-items: center;
}

.goods-list:last-child {
    border-bottom: none;
    padding-bottom: 0;
}

.goods-list:first-child {
    padding-top: 0;
}

.dialog-wrap .list-wrap {
    margin-bottom: 10px;
    padding: 10px;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}

.dialog-wrap .main {
    padding: 10px;
    margin-bottom: 20px;
    border: 1px solid #d1dbe5;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}

.dialog-wrap .main div {
    font-size: 14px;
}

.goods-name {
    color: #5e7382;
    font-size: 14px;
    margin: 0 20px 0 10px;
    width: 180px;
}

.goods-spec {
    color: #0066cc;
    font-size: 14px;
    margin-right: 30px;
    width: 60px;
}

.goods-number {
    color: #ff3456;
    font-size: 14px;
    margin-right: 20px;
    width: 100px;
}

.goods-number label {
    color: #666;
}


.m1 {
    display: flex;
    justify-content: flex-start;
}

.dialog-main {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #f1f1f1;

}

.user-name {
    color: #000000;
    font-size: 14px;
    margin-right: 10px;
    line-height: 20px;
}

.user-mobile {
    color: #000000;
    font-size: 14px;
    line-height: 20px;
    margin-right: 20px;
}

.user-address {
    color: #333;
    font-size: 13px;
    line-height: 20px;
    margin-top: 10px;
    display: flex;
    justify-content: flex-start;
}

.user-post {
    color: #333;
    font-size: 14px;
    line-height: 20px;
    margin-top: 4px;
    background-color: #fbf7c5;
    padding: 10px;
}

.user-post-t {
    color: #333;
    font-size: 14px;
    line-height: 20px;
    margin-top: 4px;
    background-color: #fbf7c5;
    padding: 10px;
    margin: 10px 0;
}

.user-admin-t {
    color: #333;
    font-size: 14px;
    line-height: 20px;
    margin-top: 4px;
    background-color: #fde7e7;
    padding: 10px;
    margin: 10px 0;
}

.admin-memo {
    margin-top: 10px;
}

.address-input {
    width: 480px !important;
}

.senderInput {
    width: 120px !important;
}

.image-show {
    max-height: 150px;
    max-width: 150px;
    margin-right: 10px;
}

.image-show-large {
    max-height: 70vh;
    max-width: 70vw;
}

.total-price {
    margin-top: 20px;
    text-align: right;
    width: 100%;
}

/*.express-dialog{*/
/*display: flex;*/
/*!*justify-content: center;*!*/
/*}*/

</style>