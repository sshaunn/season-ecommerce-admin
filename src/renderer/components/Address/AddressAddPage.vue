<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item :to="{ name: 'address' }">自提点地址</el-breadcrumb-item>
                <el-breadcrumb-item>{{ infoForm.id ? '编辑地址' : '添加地址' }}</el-breadcrumb-item>
            </el-breadcrumb>
            <div class="operation-nav">
                <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
            </div>
        </div>
        <div class="content-main">
            <div class="form-table-box">
                <el-form ref="infoForm" :rules="infoRules" :model="infoForm" label-width="140px">
                    <el-form-item label="所在区域" prop="region">
                        <el-select v-model='infoForm.region' placeholder="请选择">
                            <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="详细地址" prop="address">
                        <el-input v-model="infoForm.address"></el-input>
                    </el-form-item>
                    <el-form-item label="所在区" prop="suburb">
                        <el-input type="suburb" v-model="infoForm.suburb"></el-input>
                    </el-form-item>
                    <el-form-item label="邮编" prop="postcode">
                        <el-input type="postcode" v-model="infoForm.postcode"></el-input>
                    </el-form-item>
                    <el-form-item label="所在州" prop="state">
                        <el-input type="state" disabled v-model="infoForm.state"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button v-if="infoForm.id > 0" type="primary" @click="saveAddressInfo">确定保存</el-button>
                        <el-button v-else type="primary" @click="addAddressInfo">确定添加</el-button>
                        <el-button @click="goBackPage">取消</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>
    </div>

</template>

<script>

import api from '@/config/api';

export default {
    data() {
        return {
            options: [{
                value: 'Melbourne - CBD',
                label: 'Melbourne - CBD'
            }, {
                value: 'Melbourne - Inner East',
                label: 'Melbourne - Inner East'
            }, {
                value: 'Melbourne - Inner South',
                label: 'Melbourne - Inner South'
            }, {
                value: 'Melbourne - North East',
                label: 'Melbourne - North East'
            }, {
                value: 'Melbourne - North West',
                label: 'Melbourne - North West'
            }, {
                value: 'Melbourne - Outer East',
                label: 'Melbourne - Outer East'
            }, {
                value: 'Melbourne - South East',
                label: 'Melbourne - South East'
            }, {
                value: 'Melbourne - West',
                label: 'Melbourne - West'
            }],

            infoForm: {
                id: 0,
                address: "",
                suburb: '',
                postcode: "",
                state: "VIC",
                region: '',
            },
            infoRules: {
                address: [
                    {required: true, message: '请填写详细地址', trigger: 'blur'},
                ],
                suburb: [
                    {required: true, message: '请填写所在Suburb', trigger: 'blur'},
                ],
                postcode: [
                    {required: true, message: '请填写邮编', trigger: 'blur'},
                ],
                region: [
                    {required: true, message: '请选择所在区域', trigger: 'blur'},
                ],
            },
        }
    },
    methods: {
        goBackPage() {
            this.$router.go(-1);
        },
        saveAddressInfo() {
            const sendData = {
                id: this.infoForm.id,
                newValues: {
                    state: this.infoForm.state,
                    address: this.infoForm.address,
                    postcode: this.infoForm.postcode,
                    suburb: this.infoForm.suburb,
                    region: this.infoForm.region,
                },
            }
            this.$refs['infoForm'].validate((valid) => {
                if (valid) {
                    this.axios.post('pickup_point/updatePickupPoint', sendData)
                        .then((response) => {
                            if (response.data.errno === 0) {
                                this.$message({
                                    type: 'success',
                                    message: '保存成功'
                                });
                                this.$router.push("/dashboard/address")
                            } else if (response.data.errno === 400) {
                                this.$message({
                                    type: 'error',
                                    message: response.data.errmsg
                                });
                            } else {
                                this.$message({
                                    type: 'error',
                                    message: '保存失败'
                                })
                            }

                        })
                } else {
                    return false;
                }
            });
        },
        addAddressInfo() {
            const sendData = {
                state: this.infoForm.state,
                address: this.infoForm.address,
                postcode: this.infoForm.postcode,
                suburb: this.infoForm.suburb,
                region: this.infoForm.region,
            };
            this.$refs['infoForm'].validate((valid) => {
                if (valid) {
                    this.axios.post('pickup_point/createPickupPoint', sendData)
                        .then((response) => {
                            if (response.data.errno === 0) {
                                this.$message({
                                    type: 'success',
                                    message: '添加成功'
                                });
                                this.$router.push({name: 'address'})
                            } else {
                                this.$message({
                                    type: 'error',
                                    message: '添加失败'
                                })
                            }
                        })
                } else {
                    return false;
                }
            });
        },
        getInfo() {
            //加载地址详情
            let that = this
            if (this.infoForm.id !== 0) {
                this.axios.post('pickup_point/getPickupPoint?id=' + this.infoForm.id).then((response) => {
                    if (response.data.errno === 0) {
                        let resInfo = response.data.data;
                        that.infoForm = resInfo;
                    }
                })
            }
        },
    },
    components: {},
    mounted() {
        this.infoForm.id = this.$route.query.id || 0;
        this.getInfo();
    }
}
</script>

<style scoped>

.el-select {
    width: 240px;
}

</style>