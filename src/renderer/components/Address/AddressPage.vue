<template>
    <div class="content-page">
        <div class="content-nav">
            <el-breadcrumb class="breadcrumb" separator="/">
                <el-breadcrumb-item>自提点地址</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="content-main">
            <div class="form-table-box">
                <el-table :data="addressList" style="width: 100%" border stripe>
                    <el-table-column prop="id" label="ID" width="60"></el-table-column>
                    <el-table-column prop="fullAddress" label="地址"></el-table-column>
                    <el-table-column label="操作" width="180">
                        <template scope="scope">
                            <el-button size="small" @click="handleRowEdit(scope.$index, scope.row)">编辑</el-button>
                            <el-button plain size="small" type="danger"
                                       @click="handleRowDelete(scope.$index, scope.row)">删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </div>
    </div>

</template>

<script>
export default {
    data() {
        return {
            addressList: [],
        }
    },
    methods: {
        handleRowEdit(index, row) {
            this.$router.push({name: 'address_add', query: {id: row.id}})
        },
        handleRowDelete(index, row) {
            this.$confirm('确定要删除?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.axios.post('pickup_point/deletePickupPoint', {id: row.id}).then((response) => {
                    if (response.data.errno === 0) {
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        this.getList();
                    }
                })
            });
        },
        onSubmitFilter() {
            this.page = 1
            this.getList()
        },
        getList() {
            this.axios.get('pickup_point/getAllPickupPoints').then((response) => {
                this.addressList = this.convertAddressList(response.data.data);
            })
        },
        convertAddressList(addressList) {
            addressList.forEach(addressObj => {
                addressObj['fullAddress'] = addressObj.address + ', ' + addressObj.suburb + ', ' +
                    addressObj.state + ', ' + addressObj.postcode + ', ' + addressObj.region
            })
            return addressList
        }

    },
    components: {},
    mounted() {
        this.getList();
    }

}
</script>

<style scoped>

</style>