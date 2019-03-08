<template>
  <div class="hello">
    <div class="nav">
      <h1>Table</h1>
      <p>
        这是一款多功能表格，有搜索、筛选、排序、分页、新增、删除、编辑（表格行内编辑）、详情（跳转到详情页面）及render函数的用法
      </p>
    </div>
    <div class="content">
      <div class="title">
        <!--搜索框-->
        <Input class="search-key"
               icon="ios-search"
               v-model.trim="searchKey"
               @on-change="searchDevice()"
               placeholder="请输入机构名称或客户名称进行搜索"/>
        <Button type="primary" style="float:left;" @click="addLineModal">弹窗新增</Button>
        <Button type="primary" style="float:right;" @click="addLine">行内新增</Button>
      </div>
      <!--表格-->
      <Table stripe class="lightBlue-header-table" :columns="deviceColumns" :data="deviceData">
        <template slot-scope="{ row, index }" slot="Jname">
          <Input type="text" v-model="editJname" v-if="editIndex === index" />
          <span v-else>{{ row.Jname }}</span>
        </template>

        <template slot-scope="{ row, index }" slot="Kname">
          <Input type="text" v-model="editKname" v-if="editIndex === index" />
          <span v-else>{{ row.Kname }}</span>
        </template>

        <template slot-scope="{ row, index }" slot="time">
          <Input type="text" v-model="editTime" v-if="editIndex === index" />
          <span v-else>{{ row.time }}</span>
        </template>

        <template slot-scope="{ row, index }" slot="day">
          <Input type="text" v-model="editDay" v-if="editIndex === index" />
          <span v-else>{{ row.day }}</span>
        </template>

        <template slot-scope="{ row, index }" slot="number">
          <Input type="text" v-model="editNumber" v-if="editIndex === index" />
          <span v-else>{{ row.number }}</span>
        </template>

        <template slot-scope="{ row, index }" slot="action">
          <div v-if="editIndex === index">
            <Button type="success" size="small" @click="handleSave(index)">保存</Button>
            <Button size="small" @click="editIndex = -1">取消</Button>
          </div>
          <div v-else>
            <Button size="small" type="info" @click="handleEdit(row, index)">编辑</Button>
          </div>
        </template>
      </Table>
      <div style="float:right;padding-top:20px;">
        <Page :total="dataCount" :page-size="pageSize" :current="current" ref="page" show-total show-elevator @on-change="handleChange" style="float:left;padding-right:30px;" />
        <Button type="primary" style="float:right; "  @click="confirm">确定</Button>
      </div>
      <!--新增-->
      <modal v-model="addModal" :title="'新增'" :mask-closable="false" @on-cancel="Cancel">
        <Row type="flex" justify="center" align="top">
          <i-col span="20">
            <Form ref="formCustom" :model="formCustom" :label-width="100">
              <FormItem label="机构名称" prop="organName">
                <i-input type="text" v-model.trim="formCustom.organName" :maxlength="20"
                         placeholder="请输入机构名称"/>
              </FormItem>
              <FormItem label="客户名称" prop="customerName">
                <i-input type="text" v-model.trim="formCustom.customerName" :maxlength="20"
                         placeholder="请输入客户名称"/>
              </FormItem>
            </Form>
          </i-col>
        </Row>
        <div class="footer" slot="footer" style="text-align: center">
          <Button  @click="Cancel">取消</Button>
          <Button type="primary" @click="saveInfo" :loading="saveDisable">保存</Button>
        </div>
      </modal>
    </div>

  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      //表格信息
      editIndex: -1,  // 当前聚焦的输入框的行数
      editJname: '',  // 第一列输入框，当然聚焦的输入框的输入内容，与 data 分离避免重构的闪烁
      editKname: '',  // 第二列输入框
      editTime: '',  // 第三列输入框
      editDay: '',  // 第四列输入框
      editNumber: '',  // 第五列输入框
      deviceColumns: [
        {
          title: '序号',
          minWidth: 20,
          align: 'center',
          type: 'index',
        },
        {
          title: '机构名称',
          align: 'center',
          minWidth: 50,
          key: 'Jname',
          slot: 'Jname',
          filters: [
            {
              label: 'ABC',
              value: 1
            },
            {
              label: 'HUANGPENG',
              value: 2
            }
          ],
          filterMultiple: false,
          filterMethod (value, row) {
            if (value === 1) {
              return row.Jname === 'ABC';
            } else if (value === 2) {
              return row.Jname === 'HUANGPENG';
            }
          }
        },
        {
          title: '客户名称',
          align: 'center',
          minWidth: 50,
          key: 'Kname',
          slot: 'Kname'
        },
        {
          title: '使用截止时间',
          align: 'center',
          minWidth: 80,
          key: 'time',
          slot: 'time'
        },
        {
          title: '剩余时间（天）',
          align: 'center',
          minWidth: 80,
          key: 'day',
          slot: 'day'
        },
        {
          title: '桥检数量',
          align: 'center',
          minWidth: 50,
          key: 'number',
          sortable: true,
          slot: 'number'
        },
        {
          title: '图标',
          align: 'center',
          key: 'status ',
          minWidth: 30,
          render: (h, params) => {
            return h('div', [
              h('Icon', {
                props: {
                  type: 'ios-person',
                  size: 32,
                }
              }),
            ])
          }
        },
        {
          title: '设备状态',
          align: 'center',
          minWidth: 50,
          key: 'status',
          render: (h, params) => {
            let statusColor = '';
            let statusStr = "在线";
            if (params.row.status === 1) {//正常
              statusColor = '#63be34'
            } else {
              statusColor = 'grey';
              statusStr = "离线"
            }
            return h('div', [
              h('Icon', {
                props: {
                  type: 'ios-radio-button-on',
                  color: statusColor
                }
              }),
              h('span', {
                style: {
                  color: 'grey',
                  marginRight: '10px',
                  marginLeft: '5px'
                }
              }, statusStr)
            ])
          },
        },
        {
          title: '开关',
          align: 'center',
          minWidth: 30,
          key: 'switch ',
          render: (h, params) => {
            return h('div', [
              h('i-switch', { //数据库1是打开，0是关闭
                props: {
                  type: 'primary',
                  value: params.row.treatment === 1  //控制开关的打开或关闭状态，官网文档属性是value
                },
                style: {
                  //这里写样式
                },
                on: {
                  'on-change': (value) => {//触发事件是on-change,用双引号括起来，
                    //参数value是回调值，并没有使用到
                    this.changeSwitch(params.index,value) //params.index是拿到table的行序列，可以取到对应的表格值
                  }
                }
              }, )
            ])
          }
        },
        {
          title: '操作',
          width: 150,
          align: 'center',
          render: (h, params) => {
            let that = this;
            return h('div', [
              h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    // this.show(params.index)
                    sessionStorage.deviceId = params.row.id;  //传ID
                    that.$router.push({
                      path: `/list`
                    });
                    sessionStorage.Jname = params.row.Jname;  //机构名称
                    sessionStorage.Kname = params.row.Kname;  //客户名称
                  }
                }
              }, '详情'),
              h('Button', {
                props: {
                  type: 'error',
                  size: 'small'
                },
                on: {
                  click: () => {
                    this.remove(params.index)
                  }
                }
              }, '删除')
            ]);
          }
        },
        {
          title:'编辑',
          align: 'center',
          slot: 'action',
          minWidth: 80,
        }
      ],
      tableData: [
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:1,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:2,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:3,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:4,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:5,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:6,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:7,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:8,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:9,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:10,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:11,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:12,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:13,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:14,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:15,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:16,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:17,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:18,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:19,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:20,
          status:0,
          treatment:0
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:21,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:22,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:23,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:24,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:25,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:26,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:27,
          status:0,
          treatment:0
        },
        {
          id:1,
          Jname:'ABC',
          Kname:'林美',
          time:'2025-02-26',
          day:2001,
          number:28,
          status:1,
          treatment:1
        },
        {
          id:2,
          Jname:'HUANGPENG',
          Kname:'黄鹏',
          time:'2025-02-26',
          day:2451,
          number:29,
          status:0,
          treatment:0
        },
      ],
      deviceData:[],
      deviceDataCopy:[],//表格所有信息副本
      searchKey:'',//搜索关键字
      // 初始化信息总条数
      dataCount:0,
      // 每页显示多少条
      pageSize:10,
      current:1,
      addModal:false,
      formCustom: {
        organName: '',
        customerName: '',
      },
      // 按钮的禁用状态
      saveDisable: false,
    }
  },
  methods:{
    //初始化
    init(){
      let res = this.tableData; //res是后台返回的数据，我这里没有后台返回的数据，只是举个例子
      this.deviceData=res;
      this.deviceDataCopy=res;
    },
    // 搜索
    searchDevice(){
      let deviceArr = this.deviceDataCopy;
      let searchArr = [];
      let key = this.searchKey;
      if (key === ''){
        this.deviceData = this.deviceDataCopy;
        return;
      }
      for (let i = 0; i < deviceArr.length; i++){
        let _name = `${deviceArr[i].Jname}${deviceArr[i].Kname}`;
        if (_name.includes(key)){
          searchArr.push(deviceArr[i])
        }
      }
      this.deviceData = searchArr;
    },
    //删除功能
    remove (index) {
      this.deviceData.splice(index, 1);
    },
    //通过开关状态判断值然后传值进行更新
    changeSwitch(index,value) {
      //打开是true
      console.log(index);
      console.log(value);
    },
    //编辑
    handleEdit (row, index) {
      this.editJname = row.Jname;
      this.editKname = row.Kname;
      this.editTime = row.time;
      this.editDay = row.day;
      this.editNumber = row.number;
      this.editIndex = index;
    },
    //保存
    handleSave (index) {
      this.deviceData[index].Jname = this.editJname;
      this.deviceData[index].Kname = this.editKname;
      this.deviceData[index].number = this.editNumber;
      this.deviceData[index].time = this.editTime;
      this.deviceData[index].day = this.editDay;
      this.editIndex = -1;
    },
    //行内新增
    addLine(){
      let ID = this.tableData.length+1;
      let newRow = {
        id: ID,
        Jname:'',
        Kname:'',
        time:'',
        day: '',
        number:'',
        status:0,
        treatment:0
      }
      this.tableData.push(newRow);

      this.editIndex = ID;



      // 循环数组的两种方法
      this.tableData.forEach((item) => {
        // console.log(item.id);

      });
      for (let i = 0; i<this.deviceData.length; i++){
        // console.log(this.deviceData[i].id);
      }
    },
    //弹窗新增
    addLineModal(){
      this.addModal = true;
    },
    // 获取历史记录信息
    handleListApproveHistory(){
      // 保存取到的所有数据
      // this.tableData = response.data.data; //获取后台接口的数据
      this.dataCount = this.tableData.length;
      // 初始化显示，小于每页显示条数，全显，大于每页显示条数，取前每页条数显示
      if(this.dataCount < this.pageSize){
        this.deviceData = this.tableData;
      }else{
        this.deviceData = this.tableData.slice(0,this.pageSize);
      }
    },
    handleChange(index){
      let _start = ( index - 1 ) * this.pageSize;
      let _end = index * this.pageSize;
      this.deviceData = this.tableData.slice(_start,_end);
    },
    confirm (){
      //:total="dataCount"   数据总数
      // :page-size="pageSize"  每页条数
      //:current="current"  当前页码

      let t_value = this.$refs.page.$el.lastElementChild.lastElementChild.childNodes[1].value;  //获取跳转至第几页页数

      if(t_value===""){
        this.$Message.info('请填写页数');
        return false;
      }
      if(t_value>Math.ceil(this.dataCount/this.pageSize)){
        this.$Message.info('超过总页数，无法跳转');
        this.$refs.page.$el.lastElementChild.lastElementChild.childNodes[1].value = this.current;
        return false;
      }
      this.current = parseInt(t_value);
      this.handleChange(this.current);//请求后台数据方法

    },
    //取消弹窗新增
    Cancel() {
      this.addModal = false;
      this.$refs['formCustom'].resetFields();
      this.saveDisable = false
    },
    //保存新增
    saveInfo(){

    }
  },
  mounted() {
    this.init();
    this.handleListApproveHistory();  //初始化表格信息
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
.hello{
  margin:20px;
  .nav{
    margin-bottom:20px;
  }
  .content{
    .title{
      height:50px;
      width:100%;
      .search-key{
        float:left;
        width:240px;
        margin-right:10px;
      }
    }
  }
}
</style>
