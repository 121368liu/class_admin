<template>
  <div>
    <div class="result">
      <a-breadcrumb>
        <a-breadcrumb-item>首页</a-breadcrumb-item>
        <a-breadcrumb-item>基础设置</a-breadcrumb-item>
        <a-breadcrumb-item><router-link to="#">教室管理</router-link></a-breadcrumb-item>
      </a-breadcrumb>
    </div>
    <a-card>
    <div>
      <div class="operator">
        <a-button @click="showModal" type="primary">新建</a-button>
      </div>
      <a-table
              :rowKey="'roomId'"
              :columns="columns"
              :dataSource="dataSource"
              :pagination="pagination"
              @change="onTablechange">
        <template slot="classroom" slot-scope="buildingEntity">
           <span v-for="(c,index) in buildingEntity" :key="index">{{c.name}}</span>
        </template>
        <template slot="operation" slot-scope="text, record">
           <span @click="edit(record.roomId)" style="color:blue">编辑</span>|
          <a-popconfirm v-if="dataSource.length"
                        title="确认删除?"
                        cancelText="取消"
                        okText="确定"
                        @confirm="() => deleteItem(record.roomId)">
            <a href="javascript:;" style="color: red">删除</a>
          </a-popconfirm>
          <span @click="gotoNew(record.roomId)" style="color:#1abc9c">规则设置</span>
        </template>
      </a-table>
    </div>
    <a-modal :title="changeTitle"
            :visible="show"
            :closable="false">
      <template slot="footer">
        <a-button key="Save" type="primary" :loading="loading" @click="handleOk">保存</a-button>
        <a-button key="back" @click="handleCancel">取消</a-button>
      </template>
      <a-form-model :model="form" :rules="rules" ref="ruleForm">
        <a-form-model-item label="教室名称" ref="classroomName" prop="classroomName">
          <a-input v-model="form.classroomName" placeholder="请输入你想要新增的场地名称"></a-input>
        </a-form-model-item>
        <a-form-item label="所属教学楼" ref="buildingId" prop="buildingId">
          <a-select v-model="form.buildingId" :default-value="buildings[0].buildingId" placeholder="请选择场地所在教学楼" @change="changeBuilding">
            <a-select-option v-for="(building,index) in this.buildings" :key="index" :value="building.buildingId">
              {{ building.name}}
            </a-select-option>
          </a-select>
        </a-form-item>
        <a-form-model-item label="楼层" ref="floor" prop="floor">
          <a-select placeholder="请选择场地所在楼层" v-model="form.floor">
            <a-select-option v-for="(f,index) in this.floors" :key="index" :value="f">
              {{ f }}
            </a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item label="类型" ref="type" prop="type">
          <a-radio-group v-model="form.type"  @change="changePlace">
            <a-radio value="专业教学场地">专业教学场地</a-radio>
            <a-radio value="公共教学场地">公共教学场地</a-radio>
            <a-radio value="行政班教室">行政班教室</a-radio>
          </a-radio-group>
        </a-form-model-item>
        <a-form-model-item label="容纳人数" ref="capacity" prop="capacity">
          <a-input v-model="form.capacity" placeholder="请输入你想要新增的场地容量" :disabled="capDis"/>
        </a-form-model-item>
        <a-form-item label="状态" ref="status" prop="status">
          <a-switch v-model="form.status"/>
        </a-form-item>
      </a-form-model>
    </a-modal>
  </a-card>
  </div>
</template>

<script>
  import {message} from "ant-design-vue";
  const columns = [
    {
      title: "序号",
      dataIndex: "roomId",
      align:'center',
      customRender: function(t, r, index) {
        return parseInt(index) + 1
      }
    },
    {
      title: "教室名称",
      dataIndex: "name",
      align:'center',
    },
    {
      title: "楼层",
      dataIndex: "floor",
      align:'center',
    },
    {
      title: "类型",
      dataIndex: "type",
      align:'center',
      customRender:(text)=>text==0?'专业教学场地':text==1? '公共教学场地':'行政班教室'
    },
    {
      title: "容纳人数",
      dataIndex: "capacity",
      align:'center',
    },
    {
      title: "所属教学楼",
      dataIndex: "buildingEntity",
      align:'center',
      // scopedSlots: { customRender: "classroom" }
      customRender:(text)=>{
        let building="";
        if(text.name==""){
         building="----";
        }
        else{
          building=text.name;
        }
        return building;
      }
    },
    {
      title: "状态",
      dataIndex: "status",
      customRender: (text)=>text == 1 ? "可用" : "已占用"
    },
    {
      title: "操作",
      dataIndex: "operation",
      scopedSlots: { customRender: "operation" }
    }
  ];
  export default {
    name: "classroom",
    data() {
      return {
        show: false,
        loading:false,
        pagination:{
          total:0,
          pageSize:20,     //每页中显示20条数据
          showSizeChanger:true,
          onShowSizeChange:(current,pageSize)=> {
            this.pageSize=pageSize;
          }, //改变每页数量时更新显示
          showTotal:total=>`共有${total}条数据`,
        },
        //查询参数
        queryParam:{
          page:1,//第几页
          size:20,   //每页中显示的数据条数
        },
        placeName: "",
        columns: columns,
        buildings: [{buildingId:"",name:""}],
        floors: [],
        dataSource: [],
        editText:-1,
        changeTitle:'新增教室',
        capDis:false,
        formItemLayout: {
          labelCol: { span: 6 },
          wrapperCol: { span: 14 }
        },
        form:{
          classroomName:"",
          floor:'',
          type: '',
          buildingId:"",
          capacity:'',
          status:true,
        },
        rules:{
          classroomName:[
            {
              required:true,
              message:"请输入教室名称！",
              trigger:"blur"
            }
          ],
          buildingId:[
            {
              required:true,
              message:"请选择所属教学楼！",
              trigger:"change"
            }
          ],
          floor:[
            {
              required:true,
              message:"请输入楼层！",
              trigger:"blur"
            }
          ],
          type: [{
              required: true,
              message: '请选择教室类型',
              trigger: 'change'
            }
          ],
          capacity:[
            {
              required:true,
              message:"请输入教室容纳人数！",
              trigger:"blur"
            }
          ],
        }
      };
    },
    created() {
      this.buildingInfo();
      this.classroomAndBuilding();
    },
    methods: {
      //表格监听
      onTablechange(pagination) {
        this.pagination.current=pagination.current;
        this.pagination.pageSize=pagination.pageSize;
        this.queryParam.page=pagination.current;
        this.queryParam.size=pagination.pageSize;
        console.log(this.pagination.current);
        console.log(this.pagination.pageSize);
        this.buildingInfo();
      },
      async buildingInfo(){
        let {data}=await this.$api.basic.classroom.fetchList({rowCount: this.queryParam.size,current:this.queryParam.page});
        this.dataSource=data.rows;
        console.log(this.dataSource);
        const pagination={...this.pagination};
        pagination.total=data.total;
        this.pagination=pagination;
      },
      async classroomAndBuilding(){
        //获取教室和教学楼相关信息
        let {data:buildings}=await this.$api.basic.building.fetchList();
        this.buildings =buildings.rows
      },
      //新增教室弹框出现
      showModal() {
         this.changeTitle='新增教室'
         this.show = true;
         this.form.status=true;
         this.form.classroomName="";
         this.form.buildingId="";
        this.form.floor="";
        this.form.capacity="";
        this.form.type="";
      },
      //选择教室类型
      changePlace(e){
        console.log(e.target.value);
        if(e.target.value=="行政班教室"){
          this.capDis=true;
        }else{
          this.capDis=false;
        }
      },
      //保存弹窗
      async handleOk() {
        if(this.form.classroomName==""||this.form.buildingId=="" ||this.form.floor=="" ||this.form.capacity==""){
          message.warning("保存失败,请检查输入数据是否有空");
        }else{
          if(this.changeTitle=="新增教室") {
            console.log(this.form.status);
            let formData = {
              name:this.form.classroomName,
              buildingId:this.form.buildingId,
              floor:this.form.floor,
              capacity: Number(this.form.capacity),
              type:this.form.type=='专业教学场地'?0:this.form.type=='公共教学场地'?1:2,
              status: this.form.status==true?1:0,
              subId: 1
            };
            console.log(formData)
            let addData = { ...formData};
            let res = await this.$api.basic.classroom.saveClassRoom(addData);
            console.log(res);
            this.buildingInfo();
            this.show = false;
          }else{
            let formData = {
              roomId:this.dataSource[this.editText].roomId,
              name:this.form.classroomName,
              buildingId:this.form.buildingId,
              floor:this.form.floor,
              capacity: Number(this.form.capacity),
              type:this.form.type=='专业教学场地'?0:this.form.type=='公共教学场地'?1 :2,
              status: this.form.status ? 1 : 0,
              subId: 1
            };
            let addData = { ...formData};
            let {data:saveData} = await this.$api.basic.classroom.saveClassRoom(addData);
            let { data:classroomData } = await this.$api.basic.classroom.fetchList();
            this.dataSource=classroomData.rows;
            this.show = false;
          }
        }
      },
      //关闭弹窗
      handleCancel() {
        this.show = false;
      },
      //选择所在教学楼
      async changeBuilding() {
        let {data}=await this.$api.basic.building.fetchBuilding({buildingId:this.form.buildingId});
        this.form.floor = ''
        for(let j=1;j<data.result.floor+1;j++){
          this.floors=j;
        }
        return this.floors;
      },
      //跳转至规则设置
      gotoNew(id) {
        // console.log(this.dataSource);
        // console.log(id);
        this.$router.push("/basic/classroom/rule?id=" +id);
      },
      //编辑教室弹框
      edit(id){
        this.changeTitle='编辑教室';
        this.show = true;
        this.editText=this.dataSource.findIndex(item=>item.roomId==id);
        this.form.classroomName=this.dataSource[this.editText].name;
        this.form.buildingId=this.dataSource[this.editText].buildingId;
        this.form.floor=this.dataSource[this.editText].floor;
        this.form.capacity=this.dataSource[this.editText].capacity;
        console.log(this.dataSource[this.editText].type)
        if(this.dataSource[this.editText].type==0){
          this.form.type="专业教学场地";
        }else if(this.dataSource[this.editText].type==1){
          this.form.type="公共教学场地";
        }else{
          this.form.type="行政班教室";
        }
        console.log(this.dataSource[this.editText].status);
        // if(this.dataSource[this.editText].status==false){
        //   this.form.status=
        // }
        this.form.status=this.dataSource[this.editText].status==0?false:true;
      },
      //删除
      async deleteItem(row) {
        console.log(row)
        let {data} = await this.$api.basic.classroom.deleteBuilding({ids: [row]});
        console.log(data);
        if(data&&data.success){
          this.buildingInfo();
          message.success('删除成功')
        } else{
          message.warning(data.message);
        }
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
  .operator {
    margin-bottom: 18px;
  }
  @media screen and (max-width: 900px) {

  }
</style>
