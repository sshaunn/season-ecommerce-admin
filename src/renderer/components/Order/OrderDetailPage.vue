<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item :to="{ name: 'order' }">订单管理</el-breadcrumb-item>
                <el-breadcrumb-item>{{ infoForm.id ? '订单详情' : '添加订单' }}</el-breadcrumb-item>
            </el-breadcrumb>
            <div class="operation-nav">
                <el-button type="primary" @click="goBackPage" size="small" icon="arrow-left">返回列表</el-button>
            </div>
        </div>
        <div class="order-status-text">
            <div style='display: flex; align-items: center'>
                <label>{{ getStatusLabelFromNumber(infoForm.order_status) }}</label>
                <div class="pickup-text" v-if="infoForm.is_pickup === 1">自提订单</div>
                <div class="delivery-text" v-if="infoForm.is_pickup === 0">配送订单</div>
            </div>


            <div>
                <el-button type="danger" plain @click="changeStatus">变更状态</el-button>
            </div>
        </div>
        <div class="content-main">
            <div class="form-table-box">
                <el-tabs v-model="activeName">
                    <el-tab-pane label="订单信息" name="first">
                        <div class="content-title marginTop">买家信息：</div>
                        <el-table :data="addressData" style="width: 100%" border stripe>
                            <el-table-column prop="user_id" label="用户id" width="70"></el-table-column>
                            <el-table-column prop="nickname" label="昵称" width="100"></el-table-column>
                            <el-table-column prop="avatar" label="头像" width="80">
                                <template scope="scope">
                                    <img :src="scope.row.avatar" alt="" style="width: 60px;height: 60px">
                                </template>
                            </el-table-column>
                            <el-table-column prop="name" label="客户名" width="100"></el-table-column>
                            <el-table-column prop="mobile" label="客户手机" width="120"></el-table-column>
                            <el-table-column prop="address" label="客户地址"></el-table-column>
                            <el-table-column prop="postscript" label="买家备注" width="300"></el-table-column>
                        </el-table>
                        <div class="content-title marginTop">货物信息：</div>
                        <el-table :data="infoForm.orderGoods" style="width: 100%" border stripe>
                            <el-table-column prop="list_pic_url" label="商品图" width="120">list_pic_url
                                <template scope="scope">
                                    <img :src="scope.row.list_pic_url" alt="" style="width: 60px;height: 60px">
                                </template>
                            </el-table-column>
                            <el-table-column prop="goods_name" label="商品名"></el-table-column>
                            <el-table-column prop="goods_specifition_name_value" label="型号"
                                             width="100"></el-table-column>
                            <el-table-column prop="retail_price" label="售价" width="100">
                                <template scope="scope">
                                    <label>${{ scope.row.retail_price }}</label>
                                </template>
                            </el-table-column>
                            </el-table-column>
                            <el-table-column prop="number" label="购买数量" width="100"></el-table-column>
                            <el-table-column label="小计" width="100">
                                <template scope="scope">
                                    $<label>{{ scope.row.retail_price * scope.row.number }}</label>
                                </template>
                            </el-table-column>
                        </el-table>
                        <div class="detail-wrap">
                            <div class="price-col">
                                <div class="total-price"> 运费：${{ infoForm.freight_price }}</div>
                                <div class="total-price">
                                    合计：${{ infoForm.total_price }}（含运费：${{ infoForm.freight_price }}）
                                </div>
                            </div>
                            <div class="screenshot-col">
                                <img v-if='infoForm.transfer_screenshot_url'
                                     @click="picDialogVisible=true"
                                     :src="infoForm.transfer_screenshot_url"
                                     class="transfer-screenshot">
                            </div>
                            <el-dialog
                                :visible.sync="picDialogVisible"
                                center>
                                <span slot="footer" class="dialog-footer">
                                    <img class="image-show-large"
                                         v-if="infoForm.transfer_screenshot_url"
                                         :src="infoForm.transfer_screenshot_url">
                                </span>
                            </el-dialog>
                            <div v-if='!infoForm.transfer_screenshot_url'>
                                <el-tag type="warning" style="margin-left: 50px">转账截图尚未上传</el-tag>
                            </div>
                        </div>
                        <div class="memo-wrap">
                            <div class="content-title">卖家备注：</div>
                            <el-input
                                class="memo-input"
                                type="textarea"
                                autosize
                                placeholder="请输入内容"
                                v-model="infoForm.admin_memo">
                            </el-input>
                            <el-button size="small" type="primary" @click="saveAdminMemo">保存
                            </el-button>
                        </div>

                        <div class="footer">
                            <div class="item">
                                <div class="t">订单ID：</div>
                                <div class="c">{{ infoForm.id }}</div>
                            </div>
                            <div class="item">
                                <div class="t">加入时间：</div>
                                <div class="c">{{ convertTime(infoForm.create_time) }}</div>
                            </div>
                            <div class="item" v-if="infoForm.pay_time">
                                <div class="t">付款时间：</div>
                                <div class="c">{{ convertTime(infoForm.pay_time) }}</div>
                            </div>
                            <div class="item" v-if="infoForm.confirm_time">
                                <div class="t">确认时间：</div>
                                <div class="c">{{ infoForm.confirm_time }}</div>
                            </div>
                            <div class="item" v-if="infoForm.finish_time">
                                <div class="t">完成时间：</div>
                                <div class="c">{{ convertTime(infoForm.finish_time) }}</div>
                            </div>
                        </div>
                    </el-tab-pane>
                </el-tabs>
            </div>
        </div>
        <el-dialog
            title="变更状态"
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
    </div>
</template>

<script>
import ElButton from "../../../../node_modules/element-ui/packages/button/src/button.vue";
import moment from 'moment';

export default {
    data() {
        return {
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
            picDialogVisible: false,
            statusVisible: false,
            statusValue: '',
            statusData: {},
            activeName: 'first',
            dialogVisible: false,
            dialogAddressVisible: false,
            dialogGoodsListVisible: false,
            addressData: [],
            infoForm: {
                /* order_sn: 0,*/
            },
            infoRules: {
                user_id: [
                    {required: true, message: '请输入名称', trigger: 'blur'},
                ],
                order_sn: [
                    {required: true, message: '请输入简介', trigger: 'blur'},
                ],
                order_status: [
                    {required: true, message: '请选择商品图片', trigger: 'blur'},
                ],
            },
            options: [],
            addOptions: [],
            nowAddressData: {},
            goodsData: {},
            oldGoodsNumber: 0,
            expressData: {},
            is_finish: 0,
            on_posting: 0
        }
    },
    methods: {

        convertTime(time) {
            return moment(time * 1000).format('YYYY-MM-DD HH:mm:ss')
        },

        getStatusLabelFromNumber(number) {
            let result = '';
            this.statusList.forEach(item => {
                    if (number && item.value === number.toString()) {
                        result = item.label;
                    }
                }
            );
            return result;
        },


        changeStatus() {
            this.statusVisible = true;
        },
        statusConfirm() {
            const sendData = {
                order_id: this.infoForm.id,
                order_status: Number(this.statusValue),
            }
            this.axios.put('order_au/updateOrder', sendData).then((response) => {
                this.getInfo();
                this.statusVisible = false;
            })
        },

        saveAdminMemo() {

            this.axios.put('order_au/updateOrder', {
                admin_memo: this.infoForm.admin_memo,
                order_id: this.infoForm.id
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
        goBackPage() {
            this.$router.go(-1);
        },
        addressEdit() {
            this.dialogAddressVisible = true
        },
        getAllRegion() {
            let that = this;
            this.axios.get('order/getAllRegion').then((response) => {
                this.options = response.data.data;
            })
        },
        getInfo() {
            if (this.infoForm.id <= 0) {
                return false
            }
            this.axios.get('order_au/detail', {
                params: {
                    id: this.infoForm.id
                }
            }).then((response) => {
                this.infoForm = response.data.data;
                let data = {
                    user_id: this.infoForm.user_id,
                    name: this.infoForm.delivery_contact_name,
                    nickname: this.infoForm.userInfo.nickname,
                    avatar: this.infoForm.userInfo.avatar,
                    mobile: this.infoForm.delivery_contact_number,
                    postscript: this.infoForm.postscript,
                    address: this.infoForm.is_pickup === 1 ?
                        `${this.infoForm.pickup_address}, ${this.infoForm.pickup_suburb}, ${this.infoForm.pickup_postcode}` :
                        `${this.infoForm.delivery_address}, ${this.infoForm.delivery_suburb}, ${this.infoForm.delivery_postcode} `,
                    cAddress: this.infoForm.address,
                }
                this.addressData = [];
                this.addressData.push(data);
                this.nowAddressData = data;
                this.addOptions.push(
                    this.infoForm.province,
                    this.infoForm.city,
                    this.infoForm.district,
                )
            })
        },

    },
    components: {ElButton},
    mounted() {
        this.infoForm.id = this.$route.query.id || 0;
        this.getInfo();
    }
}
</script>

<style scoped>

.has-traces p {
    height: 30px;
    line-height: 30px;
}

.has-traces p label {
    font-size: 15px;
    color: #6f6f6f;
    width: 100px;
}


.traces-wrap li {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    height: 30px;

}

.memo-wrap {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}

.memo-input {
    margin-right: 10px;
}

.order-status-text {
    width: 100%;
    height: 100px;
    line-height: 100px;
    background: #fff;
    padding: 0 30px;
    box-sizing: border-box;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.content-title {
    margin-bottom: 8px;
    font-size: 14px;
}

.marginTop {
    margin-top: 20px;
}

.footer {
    width: 100%;
    margin: 30px 0;
    padding: 20px 0;
    border-top: 1px solid #d1dbe5;
}

.footer .item {
    font-size: 14px;
    color: #5e7382;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    margin-bottom: 10px;
}

.detail-wrap {
    display: flex;
    flex-direction: row;
    padding: 20px;
    border: 1px solid #f1f1f1;
    margin: 20px 0;
}

.price-col {
    display: flex;
    flex-direction: column;
}

.total-price {
    width: 100%;
    font-size: 14px;
    margin: 6px 0;
}

.item .t {
    width: 80px;
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
    margin-left: 10px;
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
    margin-left: 10px;
}

.transfer-screenshot {
    width: 100px;
    height: 100px;
    margin-left: 150px;
}

.image-show-large {
    max-height: 70vh;
    max-width: 70vw;
}

</style>
