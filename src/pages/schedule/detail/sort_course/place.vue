<template>
    <div>
        <div class="result">
            <a-breadcrumb>
                <a-breadcrumb-item>首页</a-breadcrumb-item>
                <a-breadcrumb-item><router-link to="/schedule/template">排课计划</router-link></a-breadcrumb-item>
                <a-breadcrumb-item><span @click="arrangeClass">排课详情</span></a-breadcrumb-item>
                <a-breadcrumb-item>选课排课</a-breadcrumb-item>
                <a-breadcrumb-item><router-link to="#">教室设置</router-link></a-breadcrumb-item>
            </a-breadcrumb>
        </div>
        <div class="content">
            <a-row>
                <a-col :span="17"><span style="font-size:1.5em">{{this.planData}}</span></a-col>
                <a-col>
                <button style="background-color: blue;
                        color: white;
                        height: 40px;
                        border: none;
                        border-radius: 5px;
                        float: right;
                        width: 100px" @click="back">返回</button>
                </a-col>
            </a-row>
        </div>
        <!-- /result -->
        <div class="table-bg">
            <a-row class="buttons">
                <a-col :span="3"><a-button style="width: 100px;height: 40px" @click="timesSetting">课时设置</a-button></a-col>
                <a-col :span="3"><a-button style="width: 100px;height: 40px" @click="oncesSetting" >课节设置</a-button></a-col>
                <a-col :span="3"><a-button style="width: 100px;height: 40px;background-color:#b9b9b9" @click="placeSetting">教室设置</a-button></a-col>
                <a-col :span="3"><a-button style="width: 100px;height: 40px" @click="courseSetting" disabled>课程设置</a-button></a-col>
                <a-col :span="3"><a-button style="width: 100px;height: 40px" @click="startArray" disabled>开始排课</a-button></a-col>
            </a-row>
            <a-table :rowKey="'classRoomId'"
                     :columns="columns"
                     :data-source="dataSource"
                     :pagination="false"
                     :bordered="true">
                <template slot="typeId" slot-scope="text">
                   <a-select :default-value="text" style="width: 150px" disabled>
                       <a-select-option value="0">专业教学场地</a-select-option>
                       <a-select-option value="1">公共教学场地</a-select-option>
                       <a-select-option value="2">行政班教室</a-select-option>
                   </a-select>
               </template>
                <template slot="action" slot-scope="text, record,index">
                    <a-popconfirm
                            v-if="dataSource.length"
                            title="确认删除?"
                            @confirm="() => onDelete(index)">
                        <a href="javascript:;">删除</a>
                    </a-popconfirm>
                </template>
            </a-table>
            <a-button icon="plus" style="color:white;
                    background-color:#3399cc;
                    margin:30px 0px; border-radius: 5px;
                    width: 150px;height: 40px" @click="addClass">添加教室</a-button>
            <button style="background-color: #00ccff;
                        color: white;
                        height: 40px;
                        border: none;
                        border-radius: 5px;
                        margin-top: 70px;
                       margin-left: 100px;
                        width: 150px" @click="Next">下一步</button>
            <!--        添加教室弹窗-->
            <a-modal title="添加教室"
                    :visible="visible"
                    :closable="false">
                <template slot="footer">
                    <a-button key="Save" type="primary" :loading="loading" @click="handleOk">保存</a-button>
                    <a-button key="back" @click="handleCancel">取消</a-button>
                </template>
                <template>
                    <a-tree  v-model="checkedKeys"
                             checkable
                             :checkedKeys="checkedKeys"
                             :tree-data="treeData"/>
                </template>
            </a-modal>
        </div>
    </div>
</template>
<script>
    import{message} from 'ant-design-vue'
    const columns = [
        {
            title: '序号',
            dataIndex: 'classRoomId',
            align:'center',
            customRender: function(t, r, index) {
                return parseInt(index) + 1
            }
        },
        {
            title: '教室名称',
            dataIndex: 'classroomName',
            align:'center',
        },
        {
            title: '所属教学楼',
            dataIndex: 'buildingName',
            align:'center',
        },
        {
            title: '教室类型',
            dataIndex: 'typeId',
            scopedSlots: { customRender: 'typeId' },
            align:'center',

        },
        {
            title: '楼层',
            dataIndex: 'floor',
            align:'center',
        },
        {
            title: '容量',
            dataIndex: 'capacity',
            align:'center',
        },
        {
            title: '操作',
            dataIndex: 'opt',
            scopedSlots: { customRender: 'action' },
            align:'center',
        },
    ];
    export default {
        data() {
            return {
                dataSource:[],
                columns,
                visible: false,
                loading: false,
                planData:"",
                planId:"",
                checkedKeys: [],
                treeData:[],
                deleteText:-1,
                classroom:[],
            };
        },
        async created() {
            let queryString = (window.location.hash || " ").split('?')[1]
            let planId = (queryString || " ").split('=')[1]
            this.planId = planId;
            if (planId) {
                //获取单个选课计划的信息
                let {data: {result, success}} = await this.$api.schedule.plan.schedulegetInfo({planId})
                this.planData = result.name
            }
            this.classroomInfo();
            this.buildingInfo();
        },
        //监测课程分支数据的获取
        watch: {
            checkedKeys(val) {
                console.log('watchDataOfKeys', val);
            },
        },
        methods: {
            //查看教室设置
            async classroomInfo(){
                let {data:{result,success}}=await this.$api.schedule.arrangeClass.getClass({planId:this.planId})
                console.log(result);
                this.dataSource=result;
                // console.log(this.dataSource);
            },
            //添加教室
            addClass() {
                this.visible = true;
                this.allBuildingInfo();
                console.log(this.dataSource);
                let allBuilding=[];
                for(let i=0;i<this.dataSource.length;i++){
                    allBuilding.push(this.dataSource[i].classRoomId);
                }
                console.log(allBuilding);
                this.checkedKeys=allBuilding;
            },
            //全部教室信息查看
            async allBuildingInfo(){
                this.treeData=[];
                //全部教室信息查询
                let { data:classroomData}  = await this.$api.basic.classroom.fetchList();
                console.log(classroomData.rows);
                let {data} = await this.$api.basic.classroom.fetchRoomList();
                // this.buildingsData = data.result
                console.log(data.result)
                for(let i in data.result) {
                    // 第一层
                    let numberTree = {}
                    numberTree.title = data.result[i].building_name
                    numberTree.key = data.result[i].building_Id
                    if (data.result[i].floor_list.length) {
                        //    第二层
                        numberTree.children = []
                        for (let j = 0; j < data.result[i].floor_list.length; j++) {
                            let item = data.result[i].floor_list[j]
                            let childData = {}
                            childData.key = data.result[i].building_Id+item.floor
                            childData.title = '第' + item.floor + '层'
                            if(item.classroom_list.length) {
                                childData.children = [];
                                for (let k in item.classroom_list) {
                                    let dataThree = {};
                                    dataThree.key = item.classroom_list[k].room_id;
                                    dataThree.title = item.classroom_list[k].classroom_name;
                                    childData.children.push(dataThree);
                                }
                            }
                            numberTree.children.push(childData)
                        }
                        this.treeData.push(numberTree)
                    }
                }
            },
            //获取教室相关信息
            async buildingInfo() {
                let {data} = await this.$api.basic.classroom.fetchList();
                console.log(data);
                this.classroom = data.rows;
                console.log(this.classroom);
            },
            //保存新增教室
            async handleOk(){
                let classId=this.checkedKeys;
                let classIds=classId.toString();
                let addData=[];
                let {data:{result,success}}=await this.$api.schedule.arrangeClass.addClassRoom({ids:classIds});
                console.log(result);
                this.visible=false;
                let list=[...result];
                let pushData=[];
                list.forEach(item=>{
                    addData={
                        classRoomId:item.classRoomId,
                        buildingName:item.buildingName,
                        capacity:item.capacity,
                        floor:item.floor,
                        typeId:item.typeId,
                        classroomName:item.classroomName,
                    }
                    console.log(addData);
                    pushData.push(addData);
                })
                // console.log(addData);
                console.log(pushData);
                this.dataSource=pushData;
                console.log(this.dataSource);
            },
            //取消新增教室
            handleCancel(){
              this.visible=false;
            },
            //课时设置
            timesSetting(){
                this.$router.push(`/schedule/detail/sort_course/index?planId=${this.planId}`)
            },
            //课节设置
            oncesSetting(){
                this.$router.push(`/schedule/detail/sort_course/time?planId=${this.planId}`)
            },
            //教室设置
            placeSetting(){
                this.$router.push(`/schedule/detail/sort_course/place?planId=${this.planId}`)
            },
            //课程设置
            courseSetting(){
                this.$router.push(`/schedule/detail/sort_course/course/index?planId=${this.planId}`)
            },
            //开始排课
            startArray(){
                this.$router.push(`/schedule/detail/start_class?planId=${this.planId}`)
            },
            //删除教室
            onDelete(deleId){
                console.log(deleId);
                this.dataSource.splice(deleId,1);
                console.log(this.dataSource);
            },
            //返回
            back(){
              this.$router.go(-1)
            },
            //保存并跳转至下一步
            Next(){
                this.$router.push(`/schedule/detail/sort_course/course/index?planId=${this.planId}`)
               this.saveData();
            },
            //保存教室设置
            async saveData(){
                let pushData=[];
                for(let i in this.dataSource){
                    pushData[i]={
                        classroomId:this.dataSource[i].classRoomId,
                        typeId:this.dataSource[i].typeId
                    }
                }
                let addData={
                    planId:this.planId,
                    classRoomInfo:pushData,
                }
                let {data}=await this.$api.schedule.arrangeClass.saveClass(addData);
                console.log(data);
            },
            //排课详情查看
            arrangeClass(){
                this.$router.push(`/schedule/detail/index?planId=${this.planId}`)
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
    .content{
        width: 100%;
        height: 100px;
        background-color: white;
        margin: 0px 0px 20px 0px;
        padding: 20px 25px;
        vertical-align: top;
        border-radius: 5px;
    }
    .table-bg{
        background-color: white;
        margin: 0px 0px 20px 0px;
        padding: 20px 25px;
        border-radius: 5px;
        text-align: center;
    }
    .buttons{
        margin:5px 5px 20px 5px;
        padding:2px 4px;
    }
    .buttons button{
        background-color:#e4e4e4;
        color:black;
    }
</style>
