<template>
  <div>
      <div class="result1">
          <a-breadcrumb>
              <a-breadcrumb-item>首页</a-breadcrumb-item>
              <a-breadcrumb-item><router-link to="/schedule/template">排课计划</router-link></a-breadcrumb-item>
              <a-breadcrumb-item><span @click="arrangeClass">排课详情</span></a-breadcrumb-item>
              <a-breadcrumb-item><a href="#">选课统计</a></a-breadcrumb-item>
          </a-breadcrumb>
      </div>
      <div>
          <div class="result">
              <a-row>
                  <a-col :span="12">
                      <span style="font-size:1.5em">{{this.planData}}</span>
                      <br>
                      <span class="link-font-color" style="margin-left:2em">选课时间：{{this.result.timeLimit}}</span>
                      <span class="link-font-color" style="margin-left:2em">选课中</span>
                  </a-col>
                  <a-col :span="12">
                      <a-row>
                          <a-col :span="6"><a-button style="width: 150px;height: 50px;background-color: #1abc9c;color: white" @click="changeTime">修改选课时间</a-button></a-col>
                          <a-col :span="6"><a-button style="width: 150px;height: 50px;background-color: #1abc9c;color: white" @click="changResult">修改选课结果</a-button></a-col>
<!--                          <a-col :span="6"><a-button style="width: 150px;height: 50px;background-color: red;color: white" @click="clearTable">清空</a-button></a-col>-->
                          <a-col :span="6" ><a-button style="width: 150px;height: 50px;background-color: blue;color: white" @click="back" >返回</a-button></a-col>
                      </a-row>
                  </a-col>
              </a-row>
          </div>
          <div class="info link-font-color" style="padding-left: 30px" >
              已有{{this.chooseCourseData.isChoosen}}人选课 （共{{this.chooseCourseData.total}}人）<font style="color:red">{{this.chooseCourseData.notChoosen}}人未选</font></div>
          <div class="table-bg">
              <!-- table -->
              <a-table
                      :columns="columns"
                      :data-source="dataSource"
                      :bordered = "true"
                      :pagination = "false">
              </a-table>
              <!-- charts -->
              <div style="margin-top:50px">
                  <a-row type="flex" justify="space-between" align="bottom">
                      <div id="barChart" style="width: 60%;height: 400px;"></div>
                  </a-row>
                  <a-row>
                      <div id="pieChart" style="width: 50%;height: 400px;" ></div>
                  </a-row>
              </div>
              <!-- charts -->
          </div>
      </div>
      <!--        修改选课时间弹框-->
      <a-modal title="修改选课时间"
              :visible="changeChooseTimeModal"
              :closable="false">
          <template slot="footer">
              <a-button key="Save" type="primary" :loading="loading" @click="handleOk()">保存</a-button>
              <a-button key="back" @click="handleCancel">取消</a-button>
          </template>
          <div class="chooseData">
              <a-form-model layout="horizontal" ref="ruleForm" :model="form" :rules="rules"
                            :label-col="{span:6}" :wrapper-col="{span:15}">
                  <a-form-model-item label="选课开始时间：" prop="startChooseTime" ref="startChooseTime">
                      <a-date-picker v-model="form.startChooseTime"
                                     :disabled-date="disabledStartDate"
                                     show-time
                                     format="YYYY-MM-DD"
                                     placeholder="设置开始选课时间"
                                     @openChange="handleStartOpenChange"
                                     valueFormat="YYYY-MM-DD"/>
                  </a-form-model-item>
                  <a-form-model-item label="选课结束时间：" prop="endChooseTime" ref="endChooseTime">
                      <a-date-picker v-model="form.endChooseTime"
                                     :disabled-date="disabledEndDate"
                                     show-time
                                     format="YYYY-MM-DD"
                                     placeholder="设置选课结束时间"
                                     :open="endOpen"
                                     @openChange="handleEndOpenChange"
                                     valueFormat="YYYY-MM-DD"/>
                  </a-form-model-item>
              </a-form-model>
          </div>
      </a-modal>
  </div>
</template>
<script>
    import echarts from 'echarts'
    import {message} from "ant-design-vue";
    const columns = [
        {
            title: '课程组合',
            dataIndex: 'courseCombination',
            customRender:(text,index,i)=>{
                let subject="";
                for(var j=0;j<text.length;j++){
                    if(j === text.length - 1){
                        subject = subject + text[j].subName
                    }else {
                        subject = subject + text[j].subName + "+";
                    }
                }
                return subject
            }
        },
        {
            title: '选课人数',
            dataIndex: 'combinationNum'
        }
    ];
    export default {
        data() {
            return {
                size : "small",
                dataSource:[],
                columns,
                changeChooseTimeModal:false,
                chooseCourseData:" ",
                staticChooseData:[],
                loading:false,
                endOpen:false,
                planData:" ",
                xName:[],
                yNumber:[],
                result:{},
                form:{
                  startChooseTime:null,
                  endChooseTime:null,
                },
                rules:{
                    startChooseTime:[
                        {
                            required:true,
                            message:"请选择开始选课时间",
                            trigger:"change",
                        }
                    ],
                    endChooseTime:[
                        {
                            required:true,
                            message:"请选择选课结束时间",
                            trigger:"change",
                        }
                    ]
                },
                planId:"",
                echartsDataCircle: [],
                tipsName: []
            };
        },
        mounted(){
            // this.drawLine();
        },
        watch:{
            startChooseTime(value){
                console.log('startValue',value);
            },
            endChooseTime(value){
                console.log('endValue',value);
            },
            xName(value){
                console.log('xNameValue',value);
                this.$nextTick(()=>{
                    this.drawLine()
                })
            }
        },
        async created(){
          let queryString=(window.location.hash||" ").split('?')[1]
          let planId=(queryString || " ").split('=')[1]
            this.planId=planId;
          if(planId){
              //获取单个选课计划的信息
              let {data:{result,success}}=await this.$api.schedule.plan.schedulegetInfo({planId})
              this.planData=result.name
          }
          this.staticCourse();
          this.lookInfo();
    },
        methods: {
            //指定排课计划信息查看
            async lookInfo(){
                this.planId = window.location.href.split('?')[1].split('=')[1];
                let {data:{result,success}}=await this.$api.schedule.plan.schedulegetInfo({planId:this.planId});
                console.log(result);
                this.result=result;
                console.log(this.result);
                this.form.startChooseTime=this.result.timeLimit.split(" - ")[0];
                this.form.endChooseTime=this.result.timeLimit.split(" - ")[1];
            },
            //查询课程统计情况
            async staticCourse(){
                //查询课程组合以及选课人数
                let {data}=await this.$api.schedule.statics.getSubjectCombination({planId:this.planId});
                console.log(data);
                this.dataSource=data.result;
                //统计选课人数以及课程被选情况
                let {data:chooseCourse}=await this.$api.schedule.statics.getStudentSelectNum({planId:this.planId});
                this.chooseCourseData=chooseCourse.result;
                this.staticChooseData=chooseCourse.result.selectPerSubNumberDtos;
                console.log(this.staticChooseData);
                for(let i=0;i<this.staticChooseData.length;i++){
                    let arrDatas = {
                        name: this.staticChooseData[i].subName,
                        value: this.staticChooseData[i].number,
                    }
                    this.tipsName.push(this.staticChooseData[i].subName)
                    this.xName.push(this.staticChooseData[i].subName)
                    this.yNumber.push(this.staticChooseData[i].number)
                    this.echartsDataCircle.push(arrDatas)
                }
                console.log(this.echartsDataCircle)
            },
            disabledStartDate(startValue){
                const endValue=this.form.startChooseTime
              if(!startValue||!endValue){
                  return false;
              }
              return startValue.valueOf()>endValue.valueOf();
            },
            disabledEndDate(endValue){
              const startValue=this.form.startChooseTime;
              if(!endValue||!startValue){
                  return false;
              }
              return startValue.valueOf()>=endValue.valueOf();
            },
            handleStartOpenChange(open){
              if(!open){
                  this.endOpen=true;
              }
            },
            handleEndOpenChange(open){
                this.endOpen=open;
            },
            //画图
            drawLine(){
                let barChart = this.$echarts.init(document.getElementById('barChart'))
                let pieChart = this.$echarts.init(document.getElementById('pieChart'))
                // 绘制图表
                barChart.setOption({
                    title: { text: '单个课程统计' },
                    tooltip: {},
                    xAxis: {
                        data: this.xName
                    },
                    yAxis: {},
                    series: [{
                        name: '数量',
                        type: 'bar',
                        data: this.yNumber,
                        color: 'rgb(59 161 255)'
                    }]
                });
                // 绘制图表
                pieChart.setOption({
                    tooltip: {//提示框，可以在全局也可以在
                        trigger: 'item',  //提示框的样式
                        formatter: "{a} <br/>{b}: {c} ({d}%)",
                        color:'#000', //提示框的背景色
                        textStyle:{ //提示的字体样式
                            color:"black",
                        }
                    },
                    legend: {  //图例
                        orient: 'vertical',  //图例的布局，竖直    horizontal为水平
                        x: 'right',//图例显示在右边
                        // for(var i=0;i<this.staticChooseData.length;i++){}
                        data: this.tipsName,
                        textStyle:{    //图例文字的样式
                            color:'#333',  //文字颜色
                            fontSize:12    //文字大小
                        }
                    },
                    series: [
                        {
                            type:'pie', //环形图的type和饼图相同
                            radius: ['20%', '50%'],//饼图的半径，第一个为内半径，第二个为外半径
                            avoidLabelOverlap: false,
                            color:['#D1FBEF','#F9D858','#4CD0DD','#DF86F0','#F5A7C1'],
                            label: {
                                normal: {  //正常的样式
                                    show: true,
                                    position: 'left'
                                },
                                emphasis: { //选中时候的样式
                                    show: true,
                                    textStyle: {
                                        fontSize: '20',
                                        fontWeight: 'bold'
                                    }
                                }
                            },  //提示文字
                            labelLine: {
                                normal: {
                                    show: false
                                }
                            },
                            data: this.echartsDataCircle
                        }
                    ]
                });
            },
            //保存
            async handleOk(id){
                id=this.planId;
                this.changeChooseTimeModal=false;
              //保存选课时间
                //修改选课时间alterTime
                if (this.form.startChooseTime == null || this.form.endChooseTime == null) {
                    message.warning("请检查输入信息是否为空！")
                } else {
                    let timeLimit = this.form.startChooseTime + "—" + this.form.endChooseTime
                    let addData = {planId:id, timeLimit}
                    let {data} = await this.$api.schedule.statics.alterTime(addData);
                    console.log(data)
                    if(data&&data.success){
                        message.success("保存成功！");
                        this.lookInfo();
                    }
                    else{
                        message.error(data.message);
                    }
                }
            },
            handleCancel(){
                this.changeChooseTimeModal=false;
            },
            //修改选课结果
            changResult(){
                this.$router.push(`/schedule/detail/statistics/edit?planId=${this.planId}`);
            },
            //修改选课时间
            changeTime(){
                this.changeChooseTimeModal=true;
            },
            // //清空按钮
            // clearTable(){
            //   this.dataSource=[];
            // },
            //返回按钮
            back(){
                this.$router.go(-1)
            },
            //排课详情查看
            arrangeClass(){
                this.$router.push(`/schedule/detail/index?planId=${this.planId}`)
            }
        }
    };
</script>

<style lang="less" scoped>
    .result1{
        width: 100%;
        background-color: white;
        height:50px;
        margin: 20px 0px 10px 0px;
        padding-left: 25px;
        padding-top: 15px;
        vertical-align: top;
        border-radius: 5px;
    }
    .result{
        width: 100%;
        height: 300px;
        background-color: white;
        height: 100px;
        margin: 0px 0px 20px 0px;
        padding: 20px 25px;
        vertical-align: top;
        border-radius: 5px;
    }
    .link-font-color{
        color: #0000ff;
    }
    .info{
        height: 50px;
        width: 100%;
        margin: 0px 0px 10px 0px;
        padding: 10px 5px;
        background-color: white;
        border-radius: 5px;
    }
    .table-bg{
        background-color: white;
        margin: 0px 0px 20px 0px;
        padding: 20px 25px;
        border-radius: 5px;
    }
</style>
