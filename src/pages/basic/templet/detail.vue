<template>
  <div>
    <div class="result">
      <a-breadcrumb>
        <a-breadcrumb-item>首页</a-breadcrumb-item>
        <a-breadcrumb-item>基础设置</a-breadcrumb-item>
        <a-breadcrumb-item><router-link to="/basic/template/admin">课表模板管理</router-link></a-breadcrumb-item>
        <a-breadcrumb-item><router-link to="#">课表模板编辑</router-link></a-breadcrumb-item>
      </a-breadcrumb>
    </div>
    <a-card>
      <a-form-model layout="horizontal" ref="ruleForm" :model="form" :rules="rules" style="overflow:hidden">
        <a-row>
          <a-col :md="24" :sm="24">
            <a-form-model-item label="模板名称" prop="templetName" :labelCol="{ span: 2 }" :wrapperCol="{ span: 5 }">
              <a-input v-model="form.templateName" placeholder="请输入" />
            </a-form-model-item>
          </a-col>
          <a-col :md="24" :sm="24">
            <a-col span="4">
              <a-form-model-item
                      label="工作日"
                      prop="workday"
                      :labelCol="{ span: 10 }"
                      :wrapperCol="{ span: 14 }">
                <a-select placeholder="请选择" v-model="form.workday">
                  <a-select-option value="1">1</a-select-option>
                  <a-select-option value="2">2</a-select-option>
                  <a-select-option value="3">3</a-select-option>
                  <a-select-option value="4">4</a-select-option>
                  <a-select-option value="5">5</a-select-option>
                  <a-select-option value="6">6</a-select-option>
                  <a-select-option value="7">7</a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
            <a-col span="4">
              <a-form-model-item label="假期"
                      prop="restday"
                      :labelCol="{ span: 10 }"
                      :wrapperCol="{ span: 14 }">
                <a-select placeholder="请选择" v-model="form.restday">
                  <a-select-option value="1">1</a-select-option>
                  <a-select-option value="2">2</a-select-option>
                  <a-select-option value="3">3</a-select-option>
                  <a-select-option value="4">4</a-select-option>
                  <a-select-option value="5">5</a-select-option>
                  <a-select-option value="6">6</a-select-option>
                  <a-select-option value="7">7</a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
          </a-col>
          <a-col span="4" v-for="item in activity" :key="item.name">
            <a-form-model-item
                    :label="item.name"
                    :prop="item.value"
                    :labelCol="{ span: 10 }"
                    :wrapperCol="{ span: 14 }">
              <a-select placeholder="请选择" v-model="form[item.value]">
                <a-select-option v-for="option in item.options" :key="option">{{
                  option
                  }}</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row style="float: right;">
          <a-button type="primary" style="margin-left: 8px" @click="setInfo">设置</a-button>
          <a-button @click="clearInfo" style="margin-left: 8px">清空</a-button>
          <!--        <a-button-->
          <!--                type="primary"-->
          <!--                style="margin-left: 8px"-->
          <!--                @click="showPublicModal">公共时段设置</a-button>-->
          <a-button type="primary" style="margin-left: 8px" @click="showTimeModal">节次时间设置</a-button>
        </a-row>
      </a-form-model>
      <a-table
              :columns="columns"
              :dataSource="dataSource"
              :rowKey="`activity`"
              :pagination='false'/>
      <a-row>
        <a-button @click="saveInfo" type="primary" style="margin-left: 8px">保存</a-button>
        <a-button @click="goBackAdmin" style="margin-left: 8px">返回</a-button>
      </a-row>
      <!--    节次时间段设置-->
      <a-modal :visible="timeModal"
               :closable="false"
              title="节次时间设置"
              @ok="handleOkTime"
              @cancel="handleCancelTime">
        <a-table :columns="timeColumns"
                :dataSource="timeData"
                :rowKey="'activity'"
                :pagination="false">
        <template slot="time" slot-scope="time,record,index">
            <a-time-picker v-if="startTime!==null" :default-value="startTime" format="HH:mm" @click="changeStartTime(index)"/>——
            <a-time-picker v-if="endTime!==null" :default-value="endTime" @change="changeTime(index)" format="HH:mm"/>
        </template>
        </a-table>
      </a-modal>
    </a-card>
  </div>
</template>
<script>
  import moment from "moment";
  import {message} from "ant-design-vue"
  import StandardTable from "../../../components/table/StandardTable";
  const columns = [
    {
      title: "",
      dataIndex: "activity"
    },
    {
      title: "星期一",
      dataIndex: "monday"
    },
    {
      title: "星期二",
      dataIndex: "tuesday"
    },
    {
      title: "星期三",
      dataIndex: "wednesday"
    },
    {
      title: "星期四",
      dataIndex: "thursday"
    },
    {
      title: "星期五",
      dataIndex: "friday"
    },
    {
      title: "星期六",
      dataIndex: "saturday"
    },
    {
      title: "星期日",
      dataIndex: "sunday"
    }
  ];
  const activity = [
    {
      name: "早读",
      options: [0, 1, 2],
      value: "morningread"
    },
    {
      name: "上午",
      options: [0, 1, 2, 3, 4],
      value: "morning"
    },
    {
      name: "中午",
      options: [0, 1, 2],
      value: "noon"
    },
    {
      name: "下午",
      options: [0, 1, 2, 3, 4],
      value: "afternoon"
    },
    {
      name: "晚自习",
      options: [0, 1, 2, 3, 4],
      value: "evening"
    }
  ];
  const  publicColumns=[
    {
      title:'时间段位置',
      dataIndex:'timeLocate',
      align:'center',
    },
    {
      title:'时间段名称',
      dataIndex:'timeName',
      align:'center',
    },];
  const publicData=[
    {
      no:'1',
      timeLocate:'早读1之后',
      timeName:'早餐时间',
    }, {
      no:'2',
      timeLocate:'上午4之后',
      timeName:'午餐时间',
    }
  ];
  const timeColumns = [
    {
      title: "时间段名",
      dataIndex: "activity"
    },
    {
      title: "时间设置",
      dataIndex: "time",
      scopedSlots: { customRender: "time" }
    }
  ];

  export default {
    name: "QueryList",
    components: {StandardTable},
    data() {
      return {
        columns,
        timeColumns,
        timeData: [],
        timeQuery:{},
        dataSource: [],
        selectedRowKeys: [],
        selectedRows: [],
        startTime:null,
        endTime:null,
        activity,
        publicColumns,
        publicData,
        publicModal: false,
        timeModal: false,
        getTimeDatas:{},
        form: {
          templateName: undefined,
          workday: undefined,
          restday: undefined,
          morningread: undefined,
          morning: undefined,
          noon: undefined,
          afternoon: undefined,
          evening: undefined,
        },
        public:{
          addTimeName:undefined,
          addTimeLocation:undefined,
        },
        publicRules:{
          addTimeName:[
            {
              required:true,
              message:"请输入时间段名称！",
              trigger:"blur"
            }
          ],
          addTimeLocation:[
            {
              required:true,
              message:"请输入时间段位置！",
              trigger:"blur"
            }
          ]
        },
        rules: {
          templateName: [
            {
              required: true,
              message: "请输入课表模板名称",
              trigger: "blur"
            }
          ],
          workday: [
            {
              required: true,
              message: "请选择工作日天数",
              trigger: "change"
            }
          ],
          restday: [
            {
              required: true,
              message: "请选择休息天数",
              trigger: "change"
            }
          ],
          morningread: [
            {
              required: true,
              message: "请选择早读节数",
              trigger: "change"
            }
          ],
          morning: [
            {
              required: true,
              message: "请选择上午节数",
              trigger: "change"
            }
          ],
          noon: [
            {
              required: true,
              message: "请选择中午节数",
              trigger: "change"
            }
          ],
          afternoon: [
            {
              required: true,
              message: "请选择下午节数",
              trigger: "change"
            }
          ],
          evening: [
            {
              required: true,
              message: "请选择晚自习节数",
              trigger: "change"
            }
          ]
        },
        loading:false,
        platHour:{},
      };
    },
    created() {
      this.lookInfo();
      console.log(this.$router.history.current.query.id);
    },
    methods: {
      moment,
      //获取当前信息
      async lookInfo(){
        let queryString=(window.location.hash || " ").split('?')[1]
        let id=(queryString || " ").split('=')[1]
        if(id) {
          let {data} = await this.$api.basic.template.fetchTemplate({id});
          console.log(data.result);
          this.getTimeDatas=data.result.timeSetting;
          console.log(this.getTimeDatas);
          let activities = [];
          let timeDatas = [];
          let list = [...this.activity];
          list.forEach(item => {
            for (let i = 1; i <= data.result[item.value]; i++) {
              activities.push({
                activity: item.name + i,
                value: item.value + i
              });
              // console.log(item.value+i);
              timeDatas.push({
                activity: item.name + i,
                value: item.value + i,
                time: {
                  startTime:this.getTimeDatas[item.value+i].split("-")[0],
                  endTime:this.getTimeDatas[item.value+i].split("-")[1],
                },
              });
              console.log(this.getTimeDatas);
              console.log(timeDatas);
            }
          });
          this.dataSource = activities;
          this.timeData = timeDatas;
          console.log(this.timeData);
          for(let i in this.timeData){
            console.log(this.timeData[i].time);
            this.platHour=this.timeData[i].time;
            // this.startTime=moment(this.timeData[i].time.startTime,"HH:mm");
            // this.endTime=moment(this.timeData[i].time.endTime,"HH:mm");
            this.startTime=this.timeData[i].time.startTime;
            this.endTime=this.timeData[i].time.endTime;
            console.log(this.startTime);
            console.log(this.endTime);
            // this.startTime=this.timeData[i].time.startTime;
            // this.endTime=this.timeData[i].time.endTime;
            // this.startTime=this.startTime.format("HH:mm");
            // console.log(typeof this.timeData[i].time.startTime);
            // console.log(typeof this.startTime);
            // console.log(this.timeData[i].time.endTime);
          }
          this.form.templateName=data.result.templateName;
          this.form.workday=data.result.workday;
          this.form.restday=data.result.restday;
          this.form.afternoon=data.result.afternoon;
          this.form.evening=data.result.evening;
          this.form.morning=data.result.morning;
          this.form.morningread=data.result.morningread;
          this.form.noon=data.result.noon;
        }
      },
      onchange(selectedRowKeys, selectedRows) {
        this.selectedRowKeys = selectedRowKeys
        this.selectedRows = selectedRows
      },
      changeTime(index,time) {
        console.log("设置结束时间",index);
        // console.log("设置结束时间",time._d);
      },
      changeStartTime(index,time,timeString){
        console.log("index",index);
        console.log("time",time);
        console.log("timeString",timeString);
      },
      setInfo() {
        this.$refs.ruleForm.validate(valid => {
          if (valid) {
            let activities = [];
            let timeDatas = [];
            let list = [...this.activity];
            console.log(this.form);
            list.forEach(item => {
              for (let i = 1; i <= this.form[item.value]; i++) {
                activities.push({
                  activity: item.name + i,
                  value: item.value + i
                });
                timeDatas.push({
                  activity: item.name + i,
                  value: item.value + i,
                  time: [moment(undefined), moment(undefined)]
                });
              }
            });
            this.dataSource = activities;
            this.timeData = timeDatas;
            console
          } else return false;
        });
      },
      async saveInfo(){
        if(Number(this.form.workday)+Number(this.form.restday)==7){
          if(this.$router.history.current.query.id==undefined){
            let query={...this.form,timeSetting:this.timeQuery};
            let {data}=await this.$api.basic.template.saveTemplate(query);
            console.log(data.result);
            this.$router.push("/basic/template/admin");
            this.$refs.ruleForm.resetFields();
          }else{
            console.log(this.timeQuery);
            let query={id:this.$router.history.current.query.id,...this.form,timeSetting:this.timeQuery};
            console.log(query);
            let {data}=await this.$api.basic.template.saveTemplate(query);
            console.log(data.result);
            this.$router.push("/basic/template/admin");
          }
        }else{
           message.info("工作日+假期之和不等于7，请重新选择")
        }
      },
      clearInfo() {
        this.$refs.ruleForm.resetFields();
      },
      goBackAdmin() {
        this.$router.push("/basic/template/admin");
      },
      // showPublicModal() {
      //   this.publicModal = true;
      // },
      // handlePublicok() {
      //   this.publicModal = false;
      // },
      // handlePublicCancel() {
      //   this.publicModal = false;
      // },
      showTimeModal() {
        this.timeModal = true;
      },
      handleOkTime() {
        this.timeModal = false;
        this.timeData.forEach(item=>{
          console.log(item);
          let time=[]
          item.time.forEach(t=>time.push(t.format('HH:mm')))
          this.timeQuery[item.value]=time.join('-')
        })
        console.log(this.timeQuery);
      },
      handleCancelTime() {
        this.timeModal = false;
      },
      onSelectChange( selectedRowKeys,selectedRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectedRows=selectedRows
      },
      handlePublicDelete(){
          this.dataSource = this.dataSource.filter(item => this.selectedRowKeys.indexOf(item.key) < 0)
          this.selectedRows = this.selectedRows.filter(item => this.selectedRowKeys.indexOf(item.key) < 0)
      },
      addTime(){
        this.$refs.publicForm.validate(valid => {
          if (valid) {
            setTimeout(() => {
              this.publicData.push({
                key: this.publicData.length + 1,
                no: this.publicData.length + 1,
                timeLocate:this.public.addTimeLocation,
                timeName:this.public.addTimeName
              })
            }, 300)
          } else {
            console.log('error submit!!');
            return false
          }
        });
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
  @media screen and (max-width: 900px) {
  }
</style>
