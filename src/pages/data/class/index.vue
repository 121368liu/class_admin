<template>
    <div>
        <!-- result -->
        <div class="result">
            <a-breadcrumb>
                <a-breadcrumb-item>首页</a-breadcrumb-item>
                <a-breadcrumb-item><a href="">数据字典</a></a-breadcrumb-item>
                <a-breadcrumb-item><a href="">教室类型</a></a-breadcrumb-item>
            </a-breadcrumb>
        </div>
        <!-- /result -->
        <a-card class="table-bg">
            <a-row class="buttons">
                <a-col :span="3">
                    <a-button @click="addClass" type="primary"
                              style="background-color: #1abc9c">新增</a-button>
                </a-col>
                <a-col :span="3">
                    <a-button @click="Delete" style="background-color: #ff0000">删除</a-button>
                </a-col>
                <a-col :span="3">
                    <a-button @click="edit" type="primary"
                              style="background-color: #1abc9c">编辑</a-button>
                </a-col>
            </a-row>
            <a-row>
                <a-table :rowKey="'key'"
                         :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
                         :selectedRows="selectedRows"
                        :columns="columns"
                         :data-source="data"
                         :bordered="true"
                         :pagination="false"
                         style="width: 40%;margin-left: 50px;">
                </a-table>
            </a-row>
        </a-card>
        <a-modal
                :visible="addClassVisit"
                @ok="handleOk"
                @cancel="handleCancel"
                width="500px"
                :closable="false"
                ok-text="保存"
                cancel-text="取消"
                v-model="addClassVisit">
            <div>
                <a-form :form="form" :label-col="{span:5}" :wrapper-col="{span:12}" @submit="addClassHandleSubmint">
                    <a-form-item label="类名名称：">
                        <a-input placeholder="请输入"
                        v-decorate="['note',{rules:[{required:true,message:'请输入类名名称'}]}]"
                        style="width: 275px"></a-input>
                    </a-form-item>
                    <a-form-item label="备注：">
                        <a-input
                                :rows="4"
                                placeholder="请输入"
                                v-decorate="['note',{rules:[{required:true,message:'请输入类名名称'}]}]"
                                style="width: 275px;height:200px"></a-input>
                    </a-form-item>
                    <a-form-item label="是否启用">
                        <a-switch v-decorator="['switch', { valuePropName: 'checked' }]" />
                    </a-form-item>
                </a-form>
            </div>
        </a-modal>
        <a-modal
                :visible="editVisit"
                @ok="handleOk"
                @cancel="handleCancel"
                width="500px"
                :closable="false"
                ok-text="保存"
                cancel-text="取消"
                v-model="editVisit">
            <div>
                <a-form :form="form" :label-col="{span:5}" :wrapper-col="{span:12}" @submit="editHandleSubmint">
                    <a-form-item label="类名名称：">
                        <a-input placeholder="请输入"
                                 v-decorate="['note',{rules:[{required:true,message:'请输入类名名称'}]}]"
                                 style="width: 275px"></a-input>
                    </a-form-item>
                    <a-form-item label="备注：">
                        <a-input
                                placeholder="请输入"
                                v-decorate="['note',{rules:[{required:true,message:'请输入类名名称'}]}]"
                                style="width: 275px;height:200px"></a-input>
                    </a-form-item>
                    <a-form-item label="是否启用">
                        <a-switch v-decorator="['switch', { valuePropName: 'checked' }]" />
                    </a-form-item>
                </a-form>
            </div>

        </a-modal>
    </div>
</template>
<script>
    const columns=[
        {
            title:'名称',
            dataIndex:'name',
            key:'name',
            align:'center',
            width:'33%'
        },
        {
            title:'备注',
            dataIndex:'remark',
            key:'remark',
            align:'center',
            width:'33%'
        },
        {
            title:'状态',
            dataIndex:'situation',
            key:'situation',
            align:'center',
            width:'34%',
        },
    ]
    const data=[
        {
            key:'1',
            name:'行政班教室',
            situation:'可用',
        },
        {
            key:'2',
            name:'物理专用',
            situation:'关闭',
        },
        {
            key:'3',
            name:'英语专用',
        },
        {
            key:'4',
            name:'化学专用',
        }
    ]
    export default {
        data() {
            return {
                columns,
                data,
                addClassVisit: false,
                editVisit: false,
                selectedRowKeys: [], // Check here to configure the default column
                selectedRows:[],
            };
        },
        methods:{
            addClass() {
                this.addClassVisit = true;
            },
            edit() {
                this.editVisit = true;
            },
            handleOk() {
                setTimeout(() => {
                    this.addClassVisit = false;
                    this.editVisit = false;
                    // this.data.unshift({
                    //     name:'生物专用',
                    //     situation: '可用'
                    // })
                }, 2000);
            },
            handleCancel() {
                console.log('Clicked cancel button');
                this.addClassVisit = false;
                this.editVisit = false;
            },
            editHandleSubmint(){
            },
            form(){},
            addClassHandleSubmint(){},
            onSelectChange( selectedRowKeys,selectedRows) {
                this.selectedRowKeys = selectedRowKeys;
                this.selectedRows=selectedRows
            },
            Delete(){
                this.data = this.data.filter(item => this.selectedRowKeys.indexOf(item.key) < 0)
                this.selectedRows = this.selectedRows.filter(item => this.selectedRowKeys.indexOf(item.key) < 0)
            },
        }
    };
</script>

<style lang="less" scoped>
    .result{
        width: 100%;
        background-color: white;
        height:50px;
        margin: 20px 0px 10px 0px;
        padding-left: 25px;
        padding-top: 15px;
        vertical-align: top;
        border-radius: 5px;
    }
    .table-bg{
        background-color: white;
        margin: 0px 0px 20px 0px;
        padding: 20px 25px;
        border-radius: 5px;
        text-align: center;
        width: 100%;
        height: 500px;

    }
    .buttons{
        margin:15px 5px 20px 5px;
        padding:2px 4px;
        margin-left: 70px;
    }
    .buttons button{
        width: 110px;
        height: 45px;
        color:white;
    }
    /deep/ Table {
        .ant-table-thead > tr > th {
            background-color: #f4f4f4;
        }
    }

</style>
