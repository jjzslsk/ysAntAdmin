<template>
  <section class="app-container">
    <el-card class="box-card">
      <template>
        <template>
          <div id="components-form-demo-advanced-search">
            <a-form
              style="margin-bottom:1.6rem;"
              class="ant-advanced-search-form"
              @submit="handleSearch"
              :form="form"
            >
              <a-row :gutter="24">
                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 5 }" :wrapperCol="{ span: 18 }" label="操作人">
                    <a-input placeholder='无接口' v-model="filters.AdminName"></a-input>
                  </a-form-item>
                </a-col>

                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 5 }" :wrapperCol="{ span: 18 }" label="操作类型">
                    <a-input placeholder='输入类型' v-model="filters.OperName"></a-input>
                  </a-form-item>
                </a-col>

                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 5 }" :wrapperCol="{ span: 18 }" label="操作表">
                    <a-input placeholder='无接口'></a-input>
                  </a-form-item>
                </a-col>
              </a-row>

              <a-row :gutter="24">
                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 5 }" :wrapperCol="{ span: 18 }" label="业务">
                    <a-input placeholder='无接口'></a-input>
                  </a-form-item>
                </a-col>

                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 6 }" :wrapperCol="{ span: 18 }" label="操作时间">
                    <a-range-picker
                      :showTime="{ format: 'HH:mm' }"
                      format="YYYY-MM-DD HH:mm"
                      :placeholder="['开始时间', '结束时间']"
                      @change="onChange"
                      @ok="onOk"
                    />
                  </a-form-item>
                </a-col>
                


                <a-col :span="8">
                  <a-form-item :labelCol="{ span: 5 }" :wrapperCol="{ span: 18 }" label></a-form-item>
                </a-col>
              </a-row>

              <a-row>
                <a-col :span="24" :style="{ textAlign: 'right' }">
                  <a-button
                    type="primary"
                    @click="search"
                  >查询</a-button>
                  <a-button
                    type="primary"
                    :loading="loadingRefresh"
                    @click="Refresh"
                  >刷新</a-button>
                  <a-button @click="clickDel">
                    删除时段
                  </a-button>
                  <!-- <a-button
                    type="danger"
                    @click="start"
                    :disabled="!hasSelected"
                    :loading="loading"
                  >
                    删除
                    <template v-if="hasSelected">{{`(${selectedRowKeys.length})`}}</template>
                  </a-button> -->
                  
                </a-col>
              </a-row>
            </a-form>
          </div>
        </template>
      </template>

      <!--日志详细信息-->
    <a-modal :width="920" title="日志详细信息无接口" @ok="dialogFormVisibleAdd = true" @click="createData" v-model="dialogFormVisibleAdd">

      <a-row>
        <!-- <template slot="title" slot-scope="currentPageData"> -->
          <div class="panel-title">操作人：<font color="red">undefined</font> 操作类型：<font color="red">编辑</font> 操作表：<font color="red">操作按钮(sys_buttons)</font> 操作时间：<font color="red">2019-01-24 01:50:14</font></div>
        <!-- </template> -->
      <a-col :span="12"><a-table :pagination="false" :columns="columnsInfo" :dataSource="dataInfo" bordered>
        <template slot="name" slot-scope="text">
          {{text}}
        </template>
        <!-- <template slot="title" slot-scope="currentPageData">
          <div class="panel-title">操作人：<font color="red">undefined</font> 操作类型：<font color="red">编辑</font> 操作表：<font color="red">操作按钮(sys_buttons)</font> 操作时间：<font color="red">2019-01-24 01:50:14</font></div>
        </template> -->
        <!-- <template slot="footer" slot-scope="currentPageData">
          Footer
        </template> -->
      </a-table>
      </a-col>
        <a-col :span="12"><a-table :pagination="false" :columns="columnsInfoOutput" :dataSource="dataInfoOutput" bordered>
          <template slot="name" slot-scope="text">
            {{text}}
          </template>
          <!-- <template slot="title" slot-scope="currentPageData">
            <div class="panel-title">操作人：<font color="red">undefined</font> 操作类型：<font color="red">编辑</font> 操作表：<font color="red">操作按钮(sys_buttons)</font> 操作时间：<font color="red">2019-01-24 01:50:14</font></div>
          </template> -->
          <!-- <template slot="footer" slot-scope="currentPageData">
            Footer
          </template> -->
        </a-table></a-col>
      </a-row>

      

      

      <div slot="footer" class="dialog-footer">
        <el-button @click.native="dialogFormVisibleAdd=false">取消</el-button>
      </div>
    </a-modal>

    <!--删除界面-->
    <a-modal title="删除日志" @ok="handleOkEdit" @click="updateData" v-model="dialogFormVisibleEdit">
      <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
        <el-form-item label="删除时段:">
          <a-range-picker
                      :showTime="{ format: 'HH:mm' }"
                      format="YYYY-MM-DD HH:mm"
                      :placeholder="['开始时间', '结束时间']"
                      @change="onChangeDel"
                      @ok="onOkDel"
                    />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleEdit=false">取消</a-button>
        <a-button type="primary" @click="updateData">确定</a-button>
      </div>
    </a-modal>

      <!--工具条-->
      <a-table
        :pagination="false"
        :dataSource="dataList"
        :columns="columns"
      >
        <template slot="Issuc" slot-scope="text,record">
          <!-- {{record.Issuccess}} -->
          <a-badge v-if="record.Issuccess === 'true'" status="success" text="成功"/>
          <a-badge v-if="record.Issuccess === 'false'" status="error" text="失败"/>
        </template>
        <template slot="action" slot-scope="text, record">
          <a href="javascript:;" @click="handleAdd(record)">详情</a>
          <!-- <a href="javascript:;" @click="onDelete(record)">删除</a> -->
        </template>
      </a-table>

      <a-pagination
        style="margin-top:2rem;text-align: right;"
        showSizeChanger
        showQuickJumper
        v-model="current"
        :total="total"
        :showTotal="(total, range) => ` 共${total}条记录 第 ${range[0]}/${range[1]}页` "
        @showSizeChange="onShowSizeChange"
      />

      <a-divider orientation="left">管理员日志</a-divider>
      <p>管理员可以查看自己所拥有的权限的管理员日志</p>
      <a-divider dashed/>
    </el-card>
  </section>
</template>
<script>
// import script from "./script";
// export default {
//   ...script
// };
import store from "@/store/index.js"; //引入本地存储
import util from "@/utils/table.js";
import { paraHelper } from "@/utils/para.js"; //请求参数格式
import { handlePost, handleGet } from "@/api/apihelper.js";

import moment from 'moment';

const columnsInfo = [{
  title: '字段',
  dataIndex: 'name',
  scopedSlots: { customRender: 'name' },
}, {
  title: '名称',
  className: 'column-money',
  dataIndex: 'money',
}, {
  title: '旧值',
  dataIndex: 'address',
}, {
  title: '新值',
  dataIndex: 'address1',
}];

const dataInfo = [{
  key: '1',
  name: 'ButtonTag',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}, {
  key: '2',
  name: 'ButtonHTML',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}, {
  key: '3',
  name: 'Button',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}];

const columnsInfoOutput = [{
  title: '字段',
  dataIndex: 'name',
  scopedSlots: { customRender: 'name' },
}, {
  title: '名称',
  className: 'column-money',
  dataIndex: 'money',
}, {
  title: '旧值',
  dataIndex: 'address',
}, {
  title: '新值',
  dataIndex: 'address1',
}];

const dataInfoOutput = [{
  key: '1',
  name: 'ButtonTag',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}, {
  key: '2',
  name: 'ButtonHTML',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}, {
  key: '3',
  name: 'Button',
  money: '按钮标识',
  address: 'edit1',
  address1: 'edit',
}];

export default {
  data() {
    // 穿梭框
    const generateData = _ => {
      const data = [];

      for (let i = 1; i <= 10; i++) {
        data.push({
          key: i,
          label: `备选项 ${i}`,
          disabled: i % 4 === 0
        });
      }
      return data;
    };
    return {
      dataInfo,
      columnsInfo,
      form: this.$form.createForm(this),
      // 按时间查询
      dateString:[],
      //按时间删除
      dateStringDel:[],
      //分页初始化
      total: 0,
      page: 1,
      size:10,
      //按钮显示隐藏
      isShowButton: {},
      //按钮
      buttonList: [],
      ButtonIcons: {},
      ButtonNames: {},
      columns: [
        ,
        //               {
        //   title: '名称',
        //   dataIndex: 'Name',
        //   key: 'Name',
        //   scopedSlots: {
        //     filterDropdown: 'filterDropdown',
        //     filterIcon: 'filterIcon',
        //     customRender: 'customRender',
        //   },
        //   onFilter: (value, record) => record.name.toLowerCase().includes(value.toLowerCase()),
        //   onFilterDropdownVisibleChange: (visible) => {
        //     if (visible) {
        //       setTimeout(() => {
        //         this.$refs.searchInput.focus()
        //       })
        //     }
        //   },
        // }
        {
          title: "Id",
          dataIndex: "Id",
          key: "Id",
          width: '5%',
        },
        {
          title: "输入参数",
          dataIndex: "Input",
          key: "Input"
          // width: 200
        },
        {
          title: "返回结果",
          dataIndex: "Result",
          key: "Result"
          // width: 200
        },
        {
          title: "操作类型",
          dataIndex: "OperName",
          key: "OperName",
          width: 100
        },
        {
          title: "IP",
          dataIndex: "IP",
          key: "IP",
          width: 100
        },
        {
          title: "业务名称无接口",
          dataIndex: "oaName",
          key: "oaName",
          width: 100
        },
        {
          title: "操作时间",
          dataIndex: "Createdate",
          key: "Createdate",
          width: 200
        },
        {
          title: "操作人",
          dataIndex: "AdminName",
          key: "AdminName",
          width: '8%',          
        },
        {
          title: "是否成功",
          width: 100,
          dataIndex: "Issuccess",
          key: "Issuccess",
          scopedSlots: { customRender: "Issuc" }
        },
        {
          title: "操作",
          Key: "action",
          dataIndex: "action",
          scopedSlots: { customRender: "action" },
          width: 160
        }
      ],
      //搜索
      input3: "",
      input4: "",
      input5: "",
      select: "",

      dialogFormVisibleEdit: false,
      dialogFormVisibleAdd: false,


      bllCode: {
        //接口标识，由后端提供
        add: "AddYsdatabaseYsDepartment", //添加
        edit: "UpdateYsdatabaseYsDepartment", //修改
        del: "DelYsdatabaseYsLog", //删除
        getList: "GetListYsdatabaseYsLog", //获取列表
        getObj: "GetYsdatabaseYsLog", //获取对象（单个）
        getRolesList: "GetListYsdatabaseYsRole" //获取角色
      },
      filtersName: "请输入",
      para: paraHelper, //列表
      buttons: {
        //按钮去权限控制
        selectshow: false,
        addshow: false,
        updateshow: false,
        delshow: false
      },
      // para: paraHelper,
      dialogStatus: "",
      filters: {
        AdminName:'',
        OperName:'',
        Time:[],
      },
      ListsuperiorMenu: [],
      dataList: [], //主页数据
      total: 0,
      page: 1,
      sels: [], // 列表选中列
      editFormRules: {
        Name: [
          {
            required: true,
            message: "菜单名称必填",
            trigger: "blur"
          }
        ],
        code: [
          {
            required: true,
            message: "标识必填",
            trigger: "blur"
          }
        ],
        shangjiid: [
          {
            required: true,
            message: "上级菜单必填",
            trigger: "blur"
          }
        ],
        lianjie: [
          {
            required: true,
            message: "链接必填",
            trigger: "blur"
          }
        ]
      },

      // 编辑界面数据
      editForm: {
        Name: "",
        Url: "",
        id: "",
        shangjiid: "",
        lianjie: "",
        shifouxianshi: "",
        paixu: "",
        tubiao: ""
      },
      //批量选择
      selectedRowKeys: [], // Check here to configure the default column
      selectedRows: [],
      loading: false,
      loadingRefresh: false,
      //分页
      current: 1,

      filterdataListData: [],
      ids: [],
      page: 1,
      addFormVisible: false, // 添加界面是否显示
      addFormRules: {
        name: [
          {
            required: true,
            message: "请输入姓名",
            trigger: "blur"
          }
        ]
      }
    };
  },
  watch: {
    checkedKeys(val) {
      console.log("onCheck", val);
    },
    filterText(val) {
      this.$refs.tree2.filter(val);
    },

    pageSize(val) {
      console.log("pageSize", val);
    },
    current(val) {
      console.log("current", val);
      this.page = val;
      this.getDataList();
    }
  },
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0;
    }
  },
  methods: {
    //清空时间
    defaultTime(){
      return []
    },
    moment,
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    clickDel(row) {
      // ----------
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.filters = {}
    },
    // 编辑
    updateData() {
      const paraId = {
          Page: this.page,
          StartTime: this.filters.Time[0],
          EndTime: this.filters.Time[1],
          Size: this.size,
        };
        this.para.Code = 'DelYsdatabaseYsLog';
        this.para.Data = JSON.stringify(paraId);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.dataList = res.Data.List;
            this.$message({
                message: "删除完成",
                type: "success"
              });
      this.dialogFormVisibleEdit = false;
          }
        });
    },
    //分页操作
    onShowSizeChange(current, pageSize) {
      console.log("111", current, pageSize);
      // this.page = val;
      this.page = current;
      this.size = pageSize;
      this.getDataList();
    },
    //批量选择
    start() {
      this.loading = true;
      // ajax request after empty completing
      this.$confirm("确认执行删除操作吗？", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      }).then(() => {
        setTimeout(() => {
          this.loading = false;
          // this.selectedRowKeys = [];

          // this.idData = this.sels.map(item => item.id).toString();//转换为字符串
          // var Ids = this.sels.map(item => item.Id);
          const paraId = {
            Ids: this.selectedRows
          };
          this.para.Code = this.bllCode.del;
          this.para.Data = JSON.stringify(paraId);
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.getDataList();
              this.$message({
                message: "删除成功！",
                type: "success"
              });
              this.selectedRowKeys = [];
            } else {
              this.$message({
                message: res.Code + ":" + res.Message,
                type: "warning"
              });
            }
          });
        }, 1000);
      });
    },
    onSelectChange(selectedRowKeys, selectedRows) {
      this.selectedRows = [];
      console.log("selectedRowKeys changed: ", selectedRowKeys, selectedRows);
      this.selectedRowKeys = selectedRowKeys;
      selectedRows.forEach(car => {
        this.selectedRows.push(car.Id);
      });
      console.log(this.selectedRows);
    },

    //提交查询表单
    handleSearch(e) {
      e.preventDefault();
      this.form.validateFields((error, values) => {
        console.log("error", error);
        console.log("Received values of form: ", values);
      });
    },
    //查询时间选择
    onChange(value, dateString) {
      this.dateString = dateString
      this.filters.Time = dateString
      console.log("Selected Time: ", value);
      console.log("Formatted Selected Time: ", dateString);
    },
    onOk(value) {
      console.log("onOk: ", value);
    },
    //删除时间选择
    onChangeDel(value, dateString) {
      this.dateStringDel = dateString
      this.filters.Time = dateString
      console.log("Selected TimeDEL: ", value);
      console.log("Formatted Selected TimeDEL: ", dateString);
    },
    onOkDel(value) {
      console.log("onOk: ", value);
    },
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
    },
    search(){
        const paraId = {
          Page: this.page,
          StartTime: this.filters.Time[0],
          EndTime: this.filters.Time[1],
          Size: this.size,
          // AdminName:this.filters.AdminName,
          OperName:this.filters.OperName
        };
        this.para.Code = this.bllCode.getList;
        this.para.Data = JSON.stringify(paraId);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.dataList = res.Data.List;
            this.$message({
                message: "查询完成",
                type: "success"
              });
          }
        });
        console.log (paraId)
    },
    //刷新页面
    Refresh() {
      this.onChange()
      this.filters = {},
      this.filters.Time = [],
      this.dateString = [];
      this.loadingRefresh = true;
      setTimeout(() => {
        this.loadingRefresh = false;
        this.page = 1;
        this.current = 1;

        const paraId = {};
        this.para.Code = this.bllCode.getList;
        this.para.Data = JSON.stringify(paraId);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.dataList = res.Data.List;
            this.$message({
                message: "刷新成功！",
                type: "success"
              });
          }
        });
        

      }, 1000);
    },
    //穿梭框
    handleChange(value, direction, movedKeys) {
      console.log(value, direction, movedKeys);
    },
    //图标
    allotIcon() {
      this.dialogFormVisibleIcon = true;
    },
    allotButton() {
      this.dialogFormVisibleButton = true;
    },
    //行点击事件
    Rowdblclick() {
      // this.handleAdd();
    },
    //加载按钮
    loadButton(data) {
      if (data && data.length > 0) {
        this.interfaces = data.map(item => item.bllcode);
        this.buttons.addshow =
          this.interfaces.indexOf(this.bllCode.add) > 0 ? true : false;
        this.buttons.selectshow =
          this.interfaces.indexOf(this.bllCode.getList) > 0 ? true : false;
        this.buttons.updateshow =
          this.interfaces.indexOf(this.bllCode.edit) > 0 ? true : false;
        this.buttons.delshow =
          this.interfaces.indexOf(this.bllCode.del) > 0 ? true : false;
      }
    },

    handleCurrentChange(val) {
      this.page = val;
      this.getDataList();
    },
    // 获取列表
    getDataList() {
      const paras = {};
      this.para.Code = "GetListYsdatabaseYsButton";
      this.para.Data = JSON.stringify(paras);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.buttonList = res.Data.List;
        }

        this.selectedRowKeys = [];

        const paraId = {
          Page: this.page,
          // Createdate: this.dateString,
          // Createdate: '2018-12-30 16:33:44',
          // StartTime: this.filters.Time[0],
          // EndTime: this.filters.Time[1],
          Size: this.size
        };
        // this.dataList = [];
        // console.log (this.dateString,'1111')
        // console.log (paraId.StartTime,'StartTime11')
        this.para.Code = this.bllCode.getList;
        this.para.Data = JSON.stringify(paraId);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.dataList = res.Data.List;

          }
        });
      });
    },
    //删除
    onDelete(data) {
      console.log(data);
      this.$confirm("确认删除该记录吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Id: data.Id
          };
          this.para.Code = this.bllCode.del;
          this.para.Data = JSON.stringify(paraId);
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.getDataList();
              this.$message({
                message: "删除成功！",
                type: "success"
              });
            } else {
              this.$message({
                message: res.Code + ":" + res.Message,
                type: "warning"
              });
            }
          });
        })
        .catch(() => {});
    },
    // 删除
    handleDel(index, row) {
      this.$confirm("确认删除该记录吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Id: row.Id
          };
          this.para.Code = this.bllCode.del;
          this.para.Data = JSON.stringify(paraId);
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.getDataList();
              this.$message({
                message: "删除成功！",
                type: "success"
              });
            }
          });
        })
        .catch(() => {});
    },
    // 显示编辑界面
    handleEdit(index, row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;

    },

    // 新增
    createData: function() {
      this.dialogFormVisibleAdd = false;
    },

    // 显示添加界面
    handleAdd(row) {
      this.dialogStatus = "create";
      this.dialogFormVisibleAdd = true;

      const paraId = {
            Id: row.Id
          };
      this.para.Code = this.bllCode.getObj;
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          
          // this.getDataList();
          // this.$message({
          //   message: "删除成功！",
          //   type: "success"
          // });
        }
      });
    },
    //查询
    getKeyList() {
      this.page = 1;
      this.getDataList();
    }
  },
  mounted() {
    this.loadButton(store.getters.interfaces); //按权限加载按钮
    this.getDataList();
  }
};
</script>
<style scoped>
.panel-heading {
  padding: 15px;
  padding-bottom: 0;
  background: #e8edf0;
  border-color: #e8edf0;
  position: relative;
  border-bottom: 1px solid transparent;
  border-top-right-radius: 2px;
  border-top-left-radius: 2px;
}
.panel-lead {
  margin-bottom: 15px;
}
.app-container {
  background: #f0f2f5;
}
.el-select .el-input {
  width: 130px;
}
.input-with-select .el-input-group__prepend {
  background-color: #fff;
}
.tablesSyl td,
.tablesSyl th {
  padding: 5px 0;
}
.anticons-list {
  overflow: hidden;
  padding-left: 0px;
}
.anticons-list li {
  float: left;
  /* width: 16.66%; */
  text-align: center;
  list-style: none;
  cursor: pointer;
  /* height: 100px; */
  color: #555;
  -webkit-transition: color 0.3s ease-in-out, background-color 0.3s ease-in-out;
  transition: color 0.3s ease-in-out, background-color 0.3s ease-in-out;
  position: relative;
  /* margin: 3px 0; */
  /* border-radius: 4px; */
  background-color: #fff;
  overflow: hidden;
  /* padding: 10px 0 0; */
}
.anticons-list li .anticon {
  font-size: 36px;
  /* margin: 12px 0 8px; */
  -webkit-transition: -webkit-transform 0.3s ease-in-out;
  transition: -webkit-transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out, -webkit-transform 0.3s ease-in-out;
  will-change: transform;
}
/* ----------------------表单 */
.ant-advanced-search-form {
  padding: 24px;
  background: #fbfbfb;
  border: 1px solid #d9d9d9;
  border-radius: 6px;
}

.ant-advanced-search-form .ant-form-item {
  display: flex;
}

.ant-advanced-search-form .ant-form-item-control-wrapper {
  flex: 1;
}

#components-form-demo-advanced-search .ant-form {
  max-width: none;
}
#components-form-demo-advanced-search .search-result-list {
  margin-top: 16px;
  border: 1px dashed #e9e9e9;
  border-radius: 6px;
  background-color: #fafafa;
  min-height: 200px;
  text-align: center;
  padding-top: 80px;
}
th.column-money,
td.column-money {
  text-align: right !important;
}
</style>
