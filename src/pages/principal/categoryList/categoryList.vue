<template>
  <div class="wrapper">
    <ButtonGroup class="operation">
      <Button type="success" icon="md-refresh" @click="Refresh" size="large" ghost>
        刷新
      </Button>

    </ButtonGroup>
    <Table stripe border :columns="columns" :loading="loading" :data="data1" class="table" size="large"
           height="750"></Table>
    <Modal v-if="modal1_delay" v-model="modal1" :title="infoTitle" width="600px">
      <p>项目描述：{{data1[index].projectDescription}}</p>
      <br>
      <p>业务员手机：{{data1[index].principalPhone}}</p>
      <br>
      <p>项目大类：{{data1[index].projectType}}</p>
      <br>
      <p>经费额度：{{data1[index].maxMoney}}元</p>
      <br>
      申报人类型：<p style="display: inline-flex;" v-for="item in data1[index].applicantType">{{item}}&nbsp;</p>
      <br>
      <br>
      <p>是否上会：{{(data1[index].isExistMeetingReview===1?'是':'否')}}</p>
      <br>
      <p>是否可提交中期报告：{{(data1[index].interimReport.isReportActivated===true)?'是':'否 '}}<a
        v-if="!data1[index].interimReport.isReportActivated" @click="openInterimReport(index)">点击开通</a></p>
      <br v-if="data1[index].interimReport.isReportActivated">
      <p v-if="data1[index].interimReport.isReportActivated">中期报告开始时间：{{data1[index].interimReport.startTime}}</p>
      <br v-if="data1[index].interimReport.isReportActivated">
      <p v-if="data1[index].interimReport.isReportActivated">中期报告截止时间：{{data1[index].interimReport.deadline}}</p>
      <br>
      <p>是否可提交结题报告：{{(data1[index].concludingReport.isReportActivated===true)?'是':'否 '}}<a
        v-if="!data1[index].concludingReport.isReportActivated" @click="openConcludingReport(index)">点击开通</a></p>
      <br v-if="data1[index].concludingReport.isReportActivated">
      <p v-if="data1[index].concludingReport.isReportActivated">结题报告开始时间：{{data1[index].concludingReport.startTime}}</p>
      <br v-if="data1[index].concludingReport.isReportActivated">
      <p v-if="data1[index].concludingReport.isReportActivated">结题报告截止时间：{{data1[index].concludingReport.deadline}}</p>
    </Modal>
    <Modal v-model="modal3" title="开通提交中期报告" @on-ok="ok1(index)" @on-cancel="cancel1(index)" :z-index="1000">
      <Form ref="interimReport" :model="interimReport" :rules="interimReportRule">
        <FormItem label="中期报告开始时间" prop="startTime">
          <DatePicker type="datetime" :value="interimReport.startTime" v-model="interimReport.startTime"
                      format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                      :options="options1"
                      @on-change="setInterimReportStartTime(index,...arguments)"
                      placeholder="选择开始时间"></DatePicker>
        </FormItem>
        <FormItem label="中期报告截止时间" prop="startTime">
          <DatePicker type="datetime" :value="interimReport.endTime" v-model="interimReport.endTime"
                      format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                      :options="options2"
                      @on-change="setInterimReportEndTime(index,...arguments)"
                      placeholder="选择截止时间"></DatePicker>
        </FormItem>
        <FormItem label="上传模板" prop="uploadAddress">
          <Upload
            type="drag"
            :headers="uploadHeaders"
            :on-success="uploadSuccess1"
            ref="upload"
            action="http://itproject.shu.edu.cn/api/file/upload">
            <div style="padding: 20px 0">
              <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
              <p>点击或将文件拖拽到这里上传</p>
            </div>
          </Upload>
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="text" @click.native="cancel1(index)"
                style="">
          取消
        </Button>
        <Button type="primary" @click.native="ok1(index)"
                style="">
          确定
        </Button>
      </div>
    </Modal>
    <Modal v-model="modal4" title="开通提交结题报告" @on-ok="ok2(index)" @on-cancel="cancel2(index)" :z-index="1000">
      <Form ref="concludingReport" :model="concludingReport" :rules="concludingReportRule">
        <FormItem label="结题报告开始时间" prop="startTime">
          <DatePicker type="datetime" :value="concludingReport.startTime" v-model="concludingReport.startTime"
                      format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                      :options="options3"
                      @on-change="setConcludingReportStartTime(index,...arguments)"
                      placeholder="选择开始时间"></DatePicker>
        </FormItem>
        <FormItem label="结题报告截止时间" prop="startTime">
          <DatePicker type="datetime" :value="concludingReport.endTime" v-model="concludingReport.endTime"
                      format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                      :options="options4"
                      placeholder="选择截止时间"></DatePicker>
        </FormItem>
        <FormItem label="上传模板" prop="uploadAddress">
          <Upload
            type="drag"
            :headers="uploadHeaders"
            :on-success="uploadSuccess2"
            ref="upload"
            action="http://itproject.shu.edu.cn/api/file/upload">
            <div style="padding: 20px 0">
              <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
              <p>点击或将文件拖拽到这里上传</p>
            </div>
          </Upload>
        </FormItem>
      </Form>
      <div slot="footer">
        <Button type="text" @click.native="cancel2(index)"
                style="">
          取消
        </Button>
        <Button type="primary" @click.native="ok2(index)"
                style="">
          确定
        </Button>
      </div>
    </Modal>

    <Modal v-if="modal_change_delay" v-model="modal_change" title="修改项目详情" :z-index="1000" @on-visible-change="onVisibleChange">
      <p>项目名称：</p><Input v-model="data1[index].projectName" :placeholder="data1[index].projectName" disabled/>
      <br>
      <p>项目描述：</p><Input v-model="data1[index].projectDescription" :placeholder="data1[index].projectDescription"/>
      <br>
      <p>项目类型：</p><Input v-model="data1[index].projectType" :placeholder="data1[index].projectType" disabled/>
      <br>
      <p>修改申请开始时间:</p>
      <DatePicker type="datetime" v-model="data1[index].applicationStartTime"
                  format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                  @on-change="setAllTime(index,...arguments,1)"
                  placeholder="修改申请开始时间"></DatePicker>
      <br>
      <p>修改申请截止时间:</p>
      <DatePicker type="datetime" v-model="data1[index].applicationEndTime"
                  format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                  @on-change="setAllTime(index,...arguments,2)"
                  placeholder="修改申请截止时间"></DatePicker>
      <p>修改项目开始时间:</p>
      <DatePicker type="datetime" v-model="data1[index].projectStartTime"
                  format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                  @on-change="setAllTime(index,...arguments,3)"
                  placeholder="修改项目开始时间"></DatePicker>
      <br>
      <p>修改项目截止时间:</p>
      <DatePicker type="datetime" v-model="data1[index].projectEndTime"
                  format="yyyy年MM月dd日 HH:mm" style="width: 300px"
                  @on-change="setAllTime(index,...arguments,4)"
                  placeholder="修改项目截止时间"></DatePicker>
      <br>
      <p>项目联系方式：</p><Input v-model="data1[index].principalPhone" :placeholder="data1[index].principalPhone"/>
<!--      <br>-->
<!--      项目经费:<span style="font-weight: bold">{{this.data2[index].maxMoney}}元</span>-->
<!--      如需修改：-->
<!--      <br>-->
<!--      <InputNumber v-model="ChangeMoney" :max="10000" :min="0"></InputNumber>-->
<!--      <br>-->
      <!--<span>上会状态：{{(data2[index].isExistMeetingReview===1?'是':'否')}}</span>
      如需修改：<br><Select v-model="ChangeIsExistMeetingReview" style="width: 100px">
        <Option v-for="item in trueOrfalse" :value="item.value" :key="item.value" >{{item.label}}</Option>
      </Select>
      <br>
      <span>中期报告提交状态：{{(data1[index].interimReport.isReportActivated===1?'是':'否')}}</span>如需修改：
      <Select v-model="ChangeIsReportActivated">
        <Option v-for="item in trueOrfalse" :value="item.value" :key="item.value" >{{item.label}}</Option>
      </Select>
      <br>-->

<!--      <p>修改申报人类型：</p>-->
<!--      <CheckboxGroup v-model="data1[index].applicantType">-->
<!--        <Checkbox label="本科生"></Checkbox>-->
<!--        <Checkbox label="研究生"></Checkbox>-->
<!--        <Checkbox label="博士生"></Checkbox>-->
<!--      </CheckboxGroup>-->
      <br>

      <div slot="footer">
        <Button type="text" @click.native="cancel3()"
                style="">
          取消
        </Button>
        <Button type="primary" @click.native="finish(index)"
                style="">
          确定
        </Button>
      </div>
    </Modal>
  </div>
</template>

<script>

  import axios from 'axios'
  import {formatDate} from "./formatDate";

  export default {
    name: "categoryList",
    watch: {
      modal1(val) {
        if (val) {
          this.modal1_delay = true;
        }
      },
      modal_change(val) {
        if (val) {
          this.modal_change_delay = true;
        }
      }
    },
    data() {
      return {
        trueOrfalse: [
          {
            value: '1',
            label: '是'
          },
          {
            value: '0',
            label: '否'
          }
        ],
        modal1_delay: false,
        modal_change_delay: false,
        loading: false,
        modal1: false,
        modal3: false,
        modal4: false,
        modal_change: false,
        infoTitle: null,
        ChangeMoney: 1000,
        ChangeIsExistMeetingReview: 1,
        ChangeIsReportActivated: 1,
        index: 0,
        projectName: '',
        applicationEndTime: [],
        interimReportStartTime: [],
        interimReportEndTime: [],
        concludingReportStartTime: [],
        projectEndTime: [],
        options1: {
          disabledDate: date => {         //这里使用箭头函数是因为这样才能指向vue对象，也就是说使用this.applicationEndTime才不会报错
            if ((date.valueOf() < this.applicationEndTime[this.index]) || (date.valueOf() > this.projectEndTime[this.index])) {
              return true;
            } else {
              return false;
            }
          }
        },
        options2: {
          disabledDate: date => {
            if ((date.valueOf() <= this.interimReportStartTime[this.index]) || (date.valueOf() > this.projectEndTime[this.index])) {
              return true;
            } else {
              return false;
            }
          }
        },
        options3: {
          disabledDate: date => {
            if ((date.valueOf() < this.interimReportEndTime[this.index]) || (date.valueOf() > this.projectEndTime[this.index])) {
              return true;
            } else {
              return false;
            }
          }
        },
        options4: {
          disabledDate: date => {
            if ((date.valueOf() <= this.concludingReportStartTime[this.index]) || (date.valueOf() > this.projectEndTime[this.index])) {
              return true;
            } else {
              return false;
            }
          }
        },
        interimReport: {
          startTime: '',
          endTime: '',
          uploadAddress: ''
        },
        concludingReport: {
          startTime: '',
          endTime: '',
          uploadAddress: ''
        },
        interimReportRule: {
          startTime: [
            {required: true, type: 'date', message: '请选择开始时间', trigger: 'blur'}
          ],
          endTime: [
            {required: true, type: 'date', message: '请选择截止时间', trigger: 'blur'}
          ],
          uploadAddress: [
            {required: true, message: '请上传模板', trigger: 'blur'}
          ]
        },
        concludingReportRule: {
          startTime: [
            {required: true, type: 'date', message: '请选择开始时间', trigger: 'blur'}
          ],
          endTime: [
            {required: true, type: 'date', message: '请选择截止时间', trigger: 'blur'}
          ],
          uploadAddress: [
            {required: true, message: '请上传模板', trigger: 'blur'}
          ]
        },
        uploadHeaders: {
          Authorization: localStorage.getItem('token')
        },
        data1: [],
        columns: [
          {
            title: '项目名称',
            key: 'projectName',
            align: 'center'
          },
          {
            title: '项目简介',
            key: 'projectDescription',
            align: 'center',
            tooltip: true
          },
          {
            title: '项目类别',
            key: 'projectType',
            align: 'center'
          },
          {
            title: '申报开始时间',
            key: 'applicationStartTime',
            align: 'center'
          },
          {
            title: '申报截止时间',
            key: 'applicationEndTime',
            align: 'center'
          },
          {
            title: '项目开始时间',
            key: 'projectStartTime',
            align: 'center'
          },
          {
            title: '项目截止时间',
            key: 'projectEndTime',
            align: 'center'
          },
          {
            title: '操作',
            key: 'operation',
            width: 230,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'success'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      //alert(this.data1[index].projectName);
                      this.index = params.index;
                      console.log(this.index)
                      this.modal_change = true;
                    }
                  }
                }, '修改'),
                h('Button', {
                  props: {
                    type: 'info'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.infoTitle = this.data1[params.index].projectName + ' 详情'
                      this.index = params.index
                      console.log(this.index)
                      this.modal1 = true
                    }
                  }
                }, '详情'),
                h('Button', {
                  props: {
                    type: 'error'
                  },
                  on: {
                    click: () => {
                      this.$Modal.confirm({
                        title: '请再次确认',
                        content: '确认删除 ' + this.data1[params.index].projectName + ' 吗?',
                        okText: '确定',
                        cancelText: '点错了',
                        onOk: () => {
                          axios({
                            url: apiRoot + '/admin/deleteProjectCategory',
                            method: 'post',
                            data: {
                              projectCategoryId: this.data1[params.index].projectCategoryId
                            }
                          }).then((res) => {
                            if (res.data.code === 'SUCCESS') {
                              this.data1.splice(params.index, 1);
                              this.$Message.success("删除成功！")
                            } else {
                              this.$Message.error(res.data.message)
                            }
                          })
                        }
                      })
                    }
                  }
                }, '删除')
              ])
            }
          }
        ]
      }
    },
    mounted() {
      this.initData('初始化成功！');
    },
    methods: {
      Refresh() {
        this.initData("刷新成功！")
      },
      initData(msg) {
        this.loading = true
        axios({
          url: apiRoot + '/admin/findMyProjectCategory',
          method: 'get'
        }).then((res) => {
          if (res.data.code === 'SUCCESS') {
            this.data1 = res.data.data
            this.data2 = res.data.data
            for (let i = 0; i < res.data.data.length; i++) {       //将申请截止时间由string变成date
              if (this.data1[i].applicationEndTime) {
                var arr1 = res.data.data[i].applicationEndTime.split(" ");
                var sdate = arr1[0].split('-');
                var date = new Date(sdate[0], sdate[1] - 1, sdate[2]);
                this.applicationEndTime[i] = date
              }
            }
            for (let i = 0; i < res.data.data.length; i++) {      //将项目截止时间由string变成date
              if (this.data1[i].projectEndTime) {
                var arr1 = res.data.data[i].projectEndTime.split(" ");
                var sdate = arr1[0].split('-');
                var date = new Date(sdate[0], sdate[1] - 1, sdate[2]);
                this.projectEndTime[i] = date
              }
            }
            this.$Message.success(msg)
            this.loading = false
          } else {
            this.$Message.warning(res.data.message)
            this.loading = false
          }
        }).catch((err) => {
          console.error(err)
          this.$Message.error("请检查网络连接！")
          this.loading = false
        })
      },
      openInterimReport(index) {
        this.modal3 = true
        this.index = index
      },
      openConcludingReport(index) {
        console.log(this.interimReportEndTime)
        if (this.data1[index].interimReport.deadline || this.interimReportEndTime[index]) { //判断中期报告是否已经开通，否则不准开通结题报告
          this.modal4 = true;
          this.index = index;
        } else {
          this.$Message.warning("请先开通中期报告！")
        }
      },
      uploadSuccess1(response) {
        this.interimReport.uploadAddress = response.data
        this.$Message.success("上传成功！");
        console.log('uploadAddress:', this.interimReport.uploadAddress)
      },
      uploadSuccess2(response) {
        this.concludingReport.uploadAddress = response.data
        this.$Message.success("上传成功！");
        console.log('uploadAddress:', this.concludingReport.uploadAddress)
      },
      ok1(index) {
        this.$refs['interimReport'].validate((valid) => {
          if (valid) {
            axios({
              url: apiRoot + '/admin/createReport',
              method: 'post',
              data: {
                projectCategoryId: this.data1[index].projectCategoryId,
                type: 1,
                reportAddress: this.interimReport.uploadAddress,
                startTime: this.interimReport.startTime.Format("yyyy年MM月dd日 hh:mm"),
                deadline: this.interimReport.endTime.Format("yyyy年MM月dd日 hh:mm")
              }
            }).then((res) => {
              if (res.data.code === 'SUCCESS') {
                this.data1[index].interimReport.isReportActivated = true;
                this.data1[index].interimReport.startTime = res.data.startTime;
                this.data1[index].interimReport.deadline = res.data.endTime;
                this.$forceUpdate()
                this.$Message.success('开通中期报告成功！');
                console.log(this.data1[index].interimReport)
                this.$refs['interimReport'].resetFields()
                this.modal3 = false;
              } else {
                this.$Message.error('开通失败！')
              }
            })
          } else {
            this.$Message.error('请将有关字段补充完整！');
          }
        })
      },
      cancel1() {
        this.modal3 = false
        this.$refs['interimReport'].resetFields()
      },
      ok2(index) {
        this.$refs['concludingReport'].validate((valid) => {
          if (valid) {
            axios({
              url: apiRoot + '/admin/createReport',
              method: 'post',
              data: {
                projectCategoryId: this.data1[index].projectCategoryId,
                type: 2,
                reportAddress: this.concludingReport.uploadAddress,
                startTime: this.concludingReport.startTime.Format("yyyy年MM月dd日 hh:mm"),
                deadline: this.concludingReport.endTime.Format("yyyy年MM月dd日 hh:mm")
              }
            }).then((res) => {
              if (res.data.code === 'SUCCESS') {
                this.data1[index].concludingReport.isReportActivated = true;
                //TODO startTime deadline undifined??
                this.data1[index].concludingReport.startTime = res.data.startTime;
                this.data1[index].concludingReport.deadline = res.data.endTime;
                this.$Message.success('开通结题报告成功！');
                this.$refs['concludingReport'].resetFields()
                this.modal4 = false;
                this.initData('更新成功！')
              } else {
                this.$Message.error('开通失败！')
              }
            })
          } else {
            this.$Message.error('请将有关字段补充完整！');
          }
        })
      },
      cancel2() {
        this.modal4 = false
        this.$refs['concludingReport'].resetFields()
      },
      finish(index) {
        var status = true;
        if (this.ChangeIsExistMeetingReview === 1) {
          status = true;
        } else if (this.ChangeIsExistMeetingReview === 0) {
          status = false;
        }
        axios({
          url: apiRoot + '/admin/projectCategory/update',
          method: 'post',
          data: {
            pid: this.data1[index].projectCategoryId,
            info: {
              // projectName: this.data1[index].projectName,
              projectDescription: this.data1[index].projectDescription,
              // projectTypeName: this.data1[index].projectType,
              applicationStartTime: typeof(this.data1[index].applicationStartTime.valueOf())!=='string'?(formatDate(this.data1[index].applicationStartTime)):(this.data1[index].applicationStartTime.valueOf()),
              applicationEndTime: typeof(this.data1[index].applicationEndTime.valueOf())!=='string'?(formatDate(this.data1[index].applicationEndTime)):(this.data1[index].applicationEndTime.valueOf()),
              projectStartTime: typeof(this.data1[index].projectStartTime.valueOf())!=='string'?(formatDate(this.data1[index].projectStartTime)):(this.data1[index].projectStartTime.valueOf()),
              projectEndTime: typeof(this.data1[index].projectEndTime.valueOf())!=='string'?(formatDate(this.data1[index].projectEndTime)):(this.data1[index].projectEndTime.valueOf()),
              principalPhone: this.data1[index].principalPhone,
              // maxMoney: this.ChangeMoney,
              //isMeetingReviewNum: this.status,
              //application: this.data1[index].applicantType,
            },
          }
        }).then((res) => {
          if (res.data.code === 'SUCCESS') {
            this.$Message.success('修改成功！')
            this.$router.go(0)
          } else {
            this.$Message.warning(res.data.message)
          }
        }).catch(() => {
          this.$Message.error('请检查网络连接！')
        })
        this.modal_change = false;
      },
      cancel3() {
        this.modal_change = false;
      },
      setInterimReportStartTime(index, date, type) {
        // if (date) {       //将格式化的时间变成标准date格式
        //   date = date.replace('年', '-');
        //   date = date.replace('月', '-');
        //   date = date.replace('日', '');
        //   var arr1 = date.split(" ");
        //   var sdate = arr1[0].split('-');
        //   var newdate = new Date(sdate[0], sdate[1] - 1, sdate[2]);
        // }
        this.interimReportStartTime[index] = date
      },
      setInterimReportEndTime(index, date, type) {
        this.interimReportEndTime[index] = date;
      },
      setConcludingReportStartTime(index, date, type) {
        // if (date) {       //将格式化的时间变成标准date格式
        //   date = date.replace('年', '-');
        //   date = date.replace('月', '-');
        //   date = date.replace('日', '');
        //   var arr1 = date.split(" ");
        //   var sdate = arr1[0].split('-');
        //   var newdate = new Date(sdate[0], sdate[1] - 1, sdate[2]);
        //   console.log(date, "!!")
        // }
        this.concludingReportStartTime[index] = date
      },
      setAllTime(index, date, type, flag) {
        console.log(date, type, flag)
        if (flag === 1) {
          this.data1[index].applicationStartTime = date;
        } else if (flag === 2) {
          this.data1[index].applicationEndTime = date;
        } else if (flag === 3) {
          this.data1[index].projectStartTime = date;
        } else if (flag === 4) {
          this.data1[index].projectEndTime = date;
        }
        if (typeof (this.data1[index].applicationStartTime) == 'date') {
          this.data1[index].applicationStartTime = formatDate(this.data1[index].applicationStartTime);
        }
        if (typeof (this.data1[index].applicationEndTime) == 'date') {
          this.data1[index].applicationEndTime = formatDate(this.data1[index].applicationEndTime);
        }
        if (typeof (this.data1[index].projectStartTime) == 'date') {
          this.data1[index].projectStartTime = formatDate(this.data1[index].projectStartTime);
        }
        if (typeof (this.data1[index].projectEndTime) == 'date') {
          this.data1[index].projectEndTime = formatDate(this.data1[index].projectEndTime);
        }
      },
      onVisibleChange(status){
        if (status == false) {
          this.Refresh()
        }
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "categoryList";
</style>
