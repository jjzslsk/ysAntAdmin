<template>
  <section class="app-container">
    <el-card class="box-card">
      <!--工具条-->
      <el-form :inline="true" :model="filters" @submit.native.prevent>
        <span v-for="index in MenuButtonsData" :key="index.Id">
          <a-button
            :class="setClass(index.Classname)"
            style="margin-right:.3rem"
            :icon="index.Icon"
            @click="defaultClick(index)"
            type="primary"
          >{{index.Name}}</a-button>
        </span>
        <el-form-item style="float: right;">
          <span v-for="index in MenuButtonsData" :key="index.Id">
            <a-button
              v-if="index.Classname==='search'"
              :icon="index.Icon"
              @click="defaultClick(index)"
              type="primary"
            >{{index.Name}}</a-button>
          </span>
        </el-form-item>

        <el-form-item style="float: right;">
          <a-input-group compact>
            <a-select @change="this.handleSelectChange" defaultValue="名称" style="width: 40%">
              <!-- <a-select-option value='Id'>Id</a-select-option> -->
              <a-select-option value="Name">名称</a-select-option>
              <a-select-option value="Username">账号</a-select-option>
              <a-select-option value="Mobile">电话</a-select-option>
              <!-- <a-select-option value='Memo'>备注</a-select-option> -->
              <!-- <a-select-option value='Name'>名称</a-select-option> -->
            </a-select>
            <a-input style="width: 60%" defaultValue v-model="filters.data"/>
          </a-input-group>
        </el-form-item>
      </el-form>

      <!-- 部门树形 -->
      <a-col style="height:100%;margin-top: 2.5rem; width:14rem;float: left;padding-bottom:20px;">
        <el-card class="box-card">
          <div class="text item">
            <template>
              <a-tree
                style="height: 54.1rem;"
                defaultExpandAll
                :defaultSelectedKeys="['0']"
                @select="onSelect"
                :treeData="dataList"
              />
            </template>
          </div>
        </el-card>
      </a-col>

      <a-col class="UserTable" style="margin-top: 2.5rem;">
        <el-card class="box-card">
          <a-table
            @click="expandRowByClick()"
            :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
            :scroll="{ x: 1300 }"
            :pagination="false"
            :dataSource="users"
            :columns="columns"
          >
            <div
              slot="filterDropdown"
              slot-scope="{ setSelectedKeys, selectedKeys, confirm, clearFilters }"
              class="custom-filter-dropdown"
            >
              <a-input
                ref="searchInput"
                placeholder="请输入名称"
                :value="selectedKeys[0]"
                @change="e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
                @pressEnter="() => handleSearch(selectedKeys, confirm)"
              />
              <a-button type="primary" @click="() => handleSearch(selectedKeys, confirm)">快速定位</a-button>
              <a-button @click="() => handleReset(clearFilters)">取消</a-button>
            </div>
            <a-icon
              slot="filterIcon"
              slot-scope="filtered"
              type="tag"
              :style="{ color: filtered ? '#108ee9' : '#aaa' }"
            />
            <template slot="customRender" slot-scope="text">
              <span v-if="searchText">
                <template
                  v-for="(fragment, i) in text.split(new RegExp(`(?<=${searchText})|(?=${searchText})`, 'i'))"
                >
                  <span
                    v-if="fragment.toLowerCase() === searchText.toLowerCase()"
                    :key="i"
                    class="highlight"
                  >{{fragment}}</span>
                  <template v-else>{{fragment}}</template>
                </template>
              </span>
              <template v-else>{{text}}</template>
            </template>
            <template slot="Issuper" slot-scope="text,record">
              <a-badge v-if="record.Issuper == true" status="success" text="超管"/>
              <a-badge v-if="record.Issuper == false" status="error" text="普通"/>
            </template>
            <template slot="State" slot-scope="text,record">
              <a-badge v-if="record.State == true" status="success" text="启用"/>
              <a-badge v-if="record.State == false" status="error" text="停用"/>
            </template>
            <template slot="action" slot-scope="text, record">
              <span v-for="index in MenuButtonsData" :key="index.Id">
                <a
                  href="javascript:;"
                  v-if="index.Classname==='edit'"
                  @click="onEdit(record.Id)"
                >{{index.Name}}</a>
                <a
                  href="javascript:;"
                  v-if="index.Classname==='del'"
                  @click="onDelete(record)"
                >{{index.Name}}</a>
              </span>
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
        </el-card>
      </a-col>

      <a-modal
        title="分配角色"
        class="amodalButton"
        v-model="dialogFormVisibleRoles"
        @ok="dialogFormVisibleRoles = true"
        @click="dialogFormVisibleRoles = true"
      >
        <a-form-item :labelCol="{ span: 3 }">
          <a-checkbox-group :options="options" v-model="value" @change="onChangeCheckbox"/>
        </a-form-item>

        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleRoles=false">取消</a-button>
          <a-button type="primary" @click.native="dialogFormVisibleRoles=false">确认</a-button>
        </div>
      </a-modal>

      <!--二维权限-->
      <a-modal
        class="allotMent"
        title="分配权限"
        @ok="handleOkData"
        @click="allotMent"
        v-model="dialogFormVisibleData"
      >
        <template>
          <a-form-item
            v-for="i in jurisdiction"
            :label="i.label"
            :key="i.Id"
            :labelCol="{ span: 3 }"
          >
            <a-checkbox-group
              :options="i.all"
              v-model="i.part"
              @change="onChange(i.all,i.part,i.Id,i)"
            />
          </a-form-item>
        </template>

        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleData=false">取消</a-button>
          <a-button type="primary" @click.native="dialogFormVisibleData=false">确认</a-button>
        </div>
      </a-modal>

      <!--添加界面-->
      <a-modal
        title="添加用户"
        @ok="dialogFormVisibleAdd = true"
        @click="createData"
        v-model="dialogFormVisibleAdd"
      >
        <template>
          <a-tabs @change="callback" type="card">
            <a-tab-pane tab="基本信息" key="1">
              <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称:" prop="Name">
                  <el-input v-model="editForm.Name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="账号:" prop="Username">
                  <el-input
                    v-model="editForm.Username"
                    auto-complete="off"
                    :disabled="disabledZhangHao"
                    placeholder="设置后不可更改"
                  ></el-input>
                </el-form-item>
                <el-form-item label="密码:" prop="PasWord">
                  <el-input
                    type="password"
                    :disabled="disabledMima"
                    v-model="editForm.PasWord"
                    auto-complete="off"
                  ></el-input>
                </el-form-item>

                <el-form-item label="部门:">
                  <el-select v-model="editForm.DepartmentId" placeholder="请选择">
                    <el-option
                      v-for="item in departments"
                      :key="item.Name"
                      :label="item.Name"
                      :value="item.Id"
                    >{{item.Name}}</el-option>
                  </el-select>
                </el-form-item>
                <a-form-item label="角色" :labelCol="{ span: 5 }" :wrapperCol="{ span: 12 }">
                  <a-select
                    size="large"
                    v-model="editForm.RoleIds"
                    mode="multiple"
                    style="width: 100%"
                    @change="handleChangeSelect"
                    v-decorator="['RoleIds',  {rules: [{ required: true, message: '请选择角色' }]} ]"
                    placeholder="选择角色"
                  >
                    <a-select-option v-for="item in userRoles" :key="item.Id">{{item.Name}}</a-select-option>
                  </a-select>
                </a-form-item>

                <a-row>
                  <a-col :span="12">
                    <el-form-item label="是否超管:">
                      <a-switch @change="aSwitch" v-model="editForm.Issuper"/>
                    </el-form-item>
                  </a-col>
                  <a-col :span="12">
                    <el-form-item label="是否启用:">
                      <a-switch @change="aState" v-model="editForm.State"/>
                    </el-form-item>
                  </a-col>
                </a-row>
              </el-form>
            </a-tab-pane>
            <a-tab-pane tab="联系方式" key="20">
              <el-form label-width="100px">
                <el-form-item label="手机:" prop="Mobile">
                  <el-input v-model="editForm.Mobile"></el-input>
                </el-form-item>
                <el-form-item label="邮箱:" prop="Email">
                  <el-input v-model="editForm.Email"></el-input>
                </el-form-item>
                <el-form-item label="备注:">
                  <el-input type="textarea" v-model="editForm.Memo"></el-input>
                </el-form-item>
              </el-form>
            </a-tab-pane>

            <a-tab-pane tab="其他" key="3">
              <span>暂无其他信息</span>
            </a-tab-pane>
          </a-tabs>
        </template>

        <div slot="footer" class="dialog-footer">
          <el-button @click.native="dialogFormVisibleAdd=false">取消</el-button>
          <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">添加</el-button>
          <el-button v-else type="primary" @click="updateData">修改</el-button>
        </div>
      </a-modal>

      <!--编辑界面-->
      <a-modal
        :width="920"
        title="编辑用户"
        @ok="dialogFormVisibleEdit = true"
        @click="updateData"
        v-model="dialogFormVisibleEdit"
      >
        <div>
          <a-row :gutter="24">
            <a-col :span="12">
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>用户管理：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">
                <a-tag color="cyan">无接口</a-tag>
                <a-tag color="cyan">查询</a-tag>
                <a-tag color="cyan">详情</a-tag>
                <a-tag color="cyan">修改</a-tag>
                <a-tag color="cyan">删除</a-tag>
              </a-col>
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>菜单管理：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">
                <a-tag color="cyan">无接口</a-tag>
                <a-tag color="cyan">查询</a-tag>
                <a-tag color="cyan">详情</a-tag>
                <a-tag color="cyan">修改</a-tag>
                <a-tag color="cyan">导入</a-tag>
              </a-col>
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>权限管理：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">
                <a-tag color="cyan">无接口</a-tag>
                <a-tag color="cyan">查询</a-tag>
                <a-tag color="cyan">详情</a-tag>
                <a-tag color="cyan">删除</a-tag>
              </a-col>
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>测试权限</span>
              </a-col>
              <a-col :span="20" style="margin-bottom: 12px;">-</a-col>
            </a-col>

            <a-col :span="12" style="float: right;">
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>角色管理：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">
                <a-tag color="cyan">无接口</a-tag>
                <a-tag color="cyan">查询</a-tag>
                <a-tag color="cyan">详情</a-tag>
                <a-tag color="cyan">修改</a-tag>
                <a-tag color="cyan">删除</a-tag>
              </a-col>
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>会员管理：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">
                <a-tag color="cyan">无接口</a-tag>
                <a-tag color="cyan">查询</a-tag>
                <a-tag color="cyan">详情</a-tag>
                <a-tag color="cyan">修改</a-tag>
                <a-tag color="cyan">删除</a-tag>
              </a-col>
              <a-col :lg="4" :md="24" style="margin-bottom: 12px;">
                <span>测试权限：</span>
              </a-col>
              <a-col :lg="20" :md="24" style="margin-bottom: 12px;">-</a-col>
            </a-col>
          </a-row>
        </div>

        <template>
          <a-tabs @change="callback" type="card">
            <a-tab-pane tab="基本信息" key="1">
              {{editForm}}
              <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称:" prop="Name">
                  <el-input v-model="editForm.Name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="账号:" prop="Username">
                  <el-input
                    v-model="editForm.Username"
                    auto-complete="off"
                    :disabled="disabledZhangHao"
                    placeholder="设置后不可更改"
                  ></el-input>
                </el-form-item>
                <el-form-item label="密码:" prop="PasWord-">
                  <el-input
                    type="password"
                    :disabled="disabledMima"
                    v-model="editForm.PasWord"
                    auto-complete="off"
                  ></el-input>
                </el-form-item>
                <el-form-item label="部门:">
                  <el-select v-model="editForm.Departmentid" placeholder="请选择">
                    <el-option
                      v-for="item in departments"
                      :key="item.Id"
                      :label="item.Name"
                      :value="item.Id"
                    ></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="角色:">
                  <template>
                    <a-select
                      size="large"
                      mode="multiple"
                      style="width: 100%"
                      @change="handleChangeSelect"
                      placeholder="选择角色"
                      :defaultValue="RoleIdsInfo"
                    >
                      <a-select-option v-for="item in userRoles" :key="item.Id">{{item.Name}}</a-select-option>
                    </a-select>
                  </template>
                </el-form-item>
                <a-row>
                  <a-col :span="12">
                    <el-form-item label="是否超管:">
                      <a-switch @change="aSwitch" v-model="editForm.Issuper"/>
                    </el-form-item>
                  </a-col>
                  <a-col :span="12">
                    <el-form-item label="是否启用:">
                      <a-switch @change="aState" v-model="editForm.State"/>
                    </el-form-item>
                  </a-col>
                </a-row>
              </el-form>
            </a-tab-pane>
            <a-tab-pane tab="联系方式" key="2">
              <el-form label-width="100px">
                <el-form-item label="手机:" prop="Mobile">
                  <el-input v-model="editForm.Mobile"></el-input>
                </el-form-item>
                <el-form-item label="邮箱:" prop="Email">
                  <el-input v-model="editForm.Email"></el-input>
                </el-form-item>
                <el-form-item label="备注:">
                  <el-input type="textarea" v-model="editForm.Memo"></el-input>
                </el-form-item>
              </el-form>
            </a-tab-pane>

            <a-tab-pane tab="其他" key="3">
              <span>暂无其他信息</span>
            </a-tab-pane>
          </a-tabs>
        </template>

        <div slot="footer" class="dialog-footer">
          <el-button @click.native="dialogFormVisibleEdit=false">取消</el-button>
          <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">添加</el-button>
          <el-button v-else type="primary" @click="updateData">修改</el-button>
        </div>
      </a-modal>

      <!--导出界面-->
      <a-modal
        :width="600"
        title="导出Excel"
        @ok="dialogFormVisibleExport = true"
        @click="exportData"
        v-model="dialogFormVisibleExport"
      >
        <template>
          <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
            <div>
              <a-checkbox-group :options="plainOptions" v-model="checkedList" @change="onChange"/>
            </div>
          </el-form>
        </template>

        <div slot="footer" class="dialog-footer">
          <el-button @click.native="dialogFormVisibleExport=false">取消</el-button>
          <el-button type="primary" @click="exportData">确定</el-button>
        </div>
      </a-modal>

      <a-divider orientation="left">用户管理</a-divider>
      <p>一个管理员可以有多个角色组,左侧的菜单根据管理员所拥有的权限进行生成</p>
      <a-divider dashed/>
    </el-card>
  </section>
</template>

<script>
import store from "@/store/index.js"; //引入本地存储
import util from "@/utils/table.js";
import { handlePost, handleGet } from "@/api/apihelper.js";
import { paraHelper } from "@/utils/para.js"; //请求参数格式

const defaultCheckedList = [];

const options = [
  { label: "Apple", value: "Apple" },
  { label: "Pear", value: "Pear" },
  { label: "Orange", value: "Orange" }
];
// const plainOptions = ['Apple', 'Pear', 'Orange']
const plainOptions = [
  { label: "名称", value: "Name as 名称" },
  { label: "Id", value: "Id as Id" },
  { label: "账号", value: "Username as 账号" },
  // { label: '部门名称', value: 'DepartmentName as 部门名称' },
  { label: "是否启用", value: "State as 是否启用" },
  { label: "是否超管", value: "Issuper as 是否超管" },
  { label: "邮箱", value: "Email as 邮箱" },
  { label: "手机", value: "Mobile as 手机" }
];

const treeData = [
  {
    title: "0-0",
    key: "0-0",
    children: [
      {
        title: "0-0-0",
        key: "0-0-0",
        children: [
          { title: "0-0-0-0", key: "0-0-0-0" },
          { title: "0-0-0-1", key: "0-0-0-1" },
          { title: "0-0-0-2", key: "0-0-0-2" }
        ]
      },
      {
        title: "0-0-1",
        key: "0-0-1",
        children: [
          { title: "0-0-1-0", key: "0-0-1-0" },
          { title: "0-0-1-1", key: "0-0-1-1" },
          { title: "0-0-1-2", key: "0-0-1-2" }
        ]
      },
      {
        title: "0-0-2",
        key: "0-0-2"
      }
    ]
  },
  {
    title: "0-1",
    key: "0-1",
    children: [
      { title: "0-1-0-0", key: "0-1-0-0" },
      { title: "0-1-0-1", key: "0-1-0-1" },
      { title: "0-1-0-2", key: "0-1-0-2" }
    ]
  },
  {
    title: "0-2",
    key: "0-2"
  }
];
export default {
  data() {
    return {
      MenuButtonsData: [],
      MenuButtons: [],
      roleButtonItem: null,

      RoleIdsInfo: [],
      // 导出选择
      checkedList: defaultCheckedList,
      indeterminate: true,
      checkAll: false,
      plainOptions,

      options,
      value: [],
      ButtonData: [],
      // 多选
      // checkedList: defaultCheckedList,
      //模拟权限数组
      jurisdiction: [
        {
          Id: 1,
          label: "导航菜单",
          all: ["添加", "删除", "编辑", "批量删除", "查询"],
          part: ["添加", "删除"]
        },
        {
          Id: 2,
          label: "部门管理",
          all: ["添加", "删除", "编辑", "批量删除", "查询"],
          part: ["添加", "删除", "编辑"]
        },
        {
          Id: 3,
          label: "用户管理",
          all: ["添加", "删除", "编辑", "批量删除", "查询"],
          part: ["添加", "删除", "编辑"]
        },
        {
          Id: 4,
          label: "角色管理",
          all: ["添加", "删除", "编辑", "批量删除", "查询"],
          part: ["添加", "删除", "编辑"]
        },
        {
          Id: 5,
          label: "菜单按钮",
          all: ["添加", "删除", "编辑", "批量删除", "查询"],
          part: ["添加", "删除", "编辑"]
        }
      ],
      //按钮显示隐藏
      isShowButton: {},
      //按钮
      ButtonData: [],
      ButtonIcons: {},
      ButtonNames: {},
      buttonList: [],
      buttonAr: [],
      allotButtons: [],
      //初始化搜索字段
      selectValue: "Name",
      //批量选择
      selectedRowKeys: [], // Check here to configure the default column
      selectedRows: [],
      loading: false,
      loadingRefresh: false,
      //分页
      current: 1,
      //列表
      // dataButton,
      searchText: "",
      columnsTitle: [],
      columns: [
        {
          title: "名称",
          dataIndex: "Name",
          key: "Name",
          width: 180,
          scopedSlots: {
            filterDropdown: "filterDropdown",
            filterIcon: "filterIcon",
            customRender: "customRender"
          },
          onFilter: (value, record) =>
            record.name.toLowerCase().includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.$refs.searchInput.focus();
              });
            }
          }
        },
        {
          title: "Id",
          dataIndex: "Id",
          key: "Id",
          width: 60
        },
        {
          title: "账号",
          dataIndex: "Username",
          key: "Username",
          width: 120
        },
        {
          title: "部门名称",
          dataIndex: "DepartmentName",
          key: "DepartmentName",
          width: 100
        },
        {
          width: 100,
          title: "是否启用",
          dataIndex: "State",
          key: "State",
          scopedSlots: { customRender: "State" }
        },
        {
          width: 100,
          title: "是否超管",
          dataIndex: "Issuper",
          key: "Issuper",
          scopedSlots: { customRender: "Issuper" }
        },
        {
          title: "邮箱",
          dataIndex: "Email",
          key: "Email",
          width: 180
        },
        {
          title: "手机",
          dataIndex: "Mobile",
          key: "Mobile",
          width: 180
        },
        // {
        //   title: '备注',
        //   dataIndex: 'Memo',
        //   key: 'Memo',
        // },
        {
          title: "操作",
          fixed: "right",
          Key: "action",
          dataIndex: "action",
          scopedSlots: { customRender: "action" },
          width: 130
        }
        // {
        //   title: 'Address',
        //   dataIndex: 'address',
        //   key: 'address',
        //   filters: [{
        //     text: 'London',
        //     value: 'London',
        //   }, {
        //     text: 'New York',
        //     value: 'New York',
        //   }],
        //   onFilter: (value, record) => record.address.indexOf(value) === 0,
        // }
      ],
      //表单
      formLayout: "horizontal",
      form: this.$form.createForm(this),
      //部门树查询
      TreeId: 0,

      //按钮KEY
      buttonKey: "",

      mockData: [],
      targetKeys: [],

      dataList: [], //主页数据
      //tree
      expandedKeys: ["0-0-0", "0-0-1"],
      autoExpandParent: true,
      checkedKeys: [],
      selectedKeys: [],
      treeData,

      data9: null,
      TreeData: [
        {
          label: "所有部门",
          children: [
            {
              // label: this.departments.departments,
            }
          ]
        }
      ],
      filterText: "",

      bllCode: {
        //接口标识，由后端提供
        add: "AddAdmin", //新增
        edit: "UpdateAdmin", //修改
        del: "DelYsdatabaseYsAdmin", //删除
        getList: "GetYsdatabaseYsAdmin", //获取列表
        getObj: "GetSource", //获取对象（单个）
        getRolesList: "GetListYsdatabaseYsRole", //获取角色
        getListDepartment: "GetYsdatabaseYsDepartment", //获取部门
        GetTreeDepartment: "GetTreeDepartment" //获取部门树
      },
      filtersName: "用户名称",
      button: {
        query: "查询",
        add: "添加",
        batchRemove: "批量删除"
      },
      buttons: {
        //按钮去权限控制
        selectshow: false,
        addshow: false,
        updateshow: false,
        delshow: false
      },
      // shifouchaoguan: false,
      // shifouqiyong: true,
      disabledZhangHao: false,
      disabledMima: false,
      para: paraHelper,
      dialogStatus: "",
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleExport: false,
      dialogFormVisibleData: false,
      dialogFormVisibleButton: false,
      dialogFormVisibleRoles: false,
      filters: {},
      users: [],
      userRoles: [],
      //分页初始化
      total: 0,
      page: 1,
      size: 10,

      sels: [], // 列表选中列
      editFormRules: {
        mingcheng: [
          {
            required: true,
            message: "名称必填",
            trigger: "blur"
          }
        ],
        zhanghao: [
          {
            required: true,
            message: "账号必填",
            trigger: "blur"
          }
        ],
        mima: [
          {
            required: true,
            message: "密码必填",
            trigger: "blur"
          }
        ],
        zhenshixingming: [
          {
            required: true,
            message: "真实姓名必填",
            trigger: "blur"
          }
        ],
        youxiang: [
          {
            type: "email",
            required: true,
            message: "邮箱格式不正确",
            trigger: "blur"
          }
        ],
        shoujihaoma: [
          {
            required: true,
            message: "手机必填",
            trigger: "blur"
          }
          // {
          //   type: "number",
          //   message: "手机号码格式不正确",
          //   trigger: ["blur", "change"]
          // }
        ]
      },
      // 选中用户信息
      selectUserForm: {
        id: 0,
        mingcheng: "",
        shifouchaoguan: "",
        zhanghao: "",
        mima: "",
        zhenshixingming: "",
        youxiang: "",
        shifoushanchu: "",
        shifouqiyong: "",
        beizhu: ""
      },
      // 编辑界面数据
      editForm: {
        RoleIds: []
      },
      Roles: [],

      resData: {},

      departments: [],

      // 编辑界面数据
      RolesForm: {
        id: 0,
        mingcheng: "",
        shifouchaoguan: "",
        zhanghao: "",
        mima: "",
        zhenshixingming: "",
        youxiang: "",
        shifoushanchu: "",
        shifouqiyong: "",
        beizhu: ""
      },

      addFormVisible: false, // 新增界面是否显示
      addFormRules: {
        UserName: [
          {
            required: true,
            message: "请输入用户名",
            trigger: "blur"
          }
        ]
      }
    };
  },
  watch: {
    checkedKeys(val) {},
    filterText(val) {
      this.$refs.tree2.filter(val);
    },

    pageSize(val) {},
    current(val) {
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
    onChange(checkedList) {
      this.indeterminate =
        !!checkedList.length && checkedList.length < plainOptions.length;
      this.checkAll = checkedList.length === plainOptions.length;
    },
    onCheckAllChange(e) {
      Object.assign(this, {
        checkedList: e.target.checked ? plainOptions : [],
        indeterminate: false,
        checkAll: e.target.checked
      });
    },

    setClass(index) {
      if (index === "edit") {
        return "p1";
      }
      if (index === "search") {
        return "p1";
      }
    },
    allotButton() {
      this.allotButtons = [];
      var ButtonDatas = [];
      this.buttonAr.Data[0].ButtonIds.map(car => {
        //拥有的按钮ID集
        const allotButton = this.buttonList.find(i => {
          //和全部的筛选对象
          return i.Id === car;
        });
        ButtonDatas.push(allotButton);
      });
      this.allotButtons = ButtonDatas.filter(e => {
        return e;
      });
    },
    //默认点击事件
    defaultClick(index) {
      switch (index.Classname) {
        case "add":
          this.handleAdd();
          break;
        case "refresh":
          this.Refresh();
          break;
        case "search":
          this.getKeyList();
          break;
        case "del":
          this.start();
          break;
        case "export":
          this.export();
          break;
        case "edit":
          if (this.selectedRowKeys > 0) {
            this.onEdit(this.selectedRows[0]);
          } else {
            this.$message({
              message: "请选择一个需要编辑的按钮",
              type: "warning"
            });
          }
          break;
      }
    },
    //expandRowByClick
    expandRowByClick() {},
    //导出
    exportData() {
      const ColumnsData = [];

      const paraId = {
        // Columns: ["Id as id", "Name as 名称"],
        Columns: this.checkedList,
        IsExport: "True"
      };
      this.para.Code = "GetListYsdatabaseYsAdmin";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          const hrefUrl = res.Data.Url;
          // this.$router.push("www.baidu.com");
          window.location.href = process.env.BASE_URL + hrefUrl;
          this.getDataList();
          this.$message({
            message: "导出成功！",
            type: "success"
          });
        }
      });
    },
    // 多选框
    onChangeCheckbox(checkedList, checkedValues) {},

    //分页操作
    onShowSizeChange(current, pageSize) {
      // this.page = val;
      this.page = current;
      this.size = pageSize;
      this.getDataList();
    },
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
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
      this.selectedRowKeys = selectedRowKeys;
      selectedRows.forEach(car => {
        this.selectedRows.push(car.Id);
      });
    },
    //是否显示
    aSwitch(checked) {
      this.editForm.Isvisiable = checked;
    },
    // 显示导出界面
    export(row) {
      this.dialogFormVisibleExport = true;
      // this.columnsTitle = this.columns.map((i)=>{
      //   return i.dataIndex;
      // })
    },
    // 显示编辑界面
    onEdit(row) {
      this.RoleIdsInfo = [];

      // ----------
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.disabledZhangHao = true;
      // this.disabledMima = true;
      this.editForm = {};
      const paraId = {
        Id: row
      };
      this.para.Code = "GetYsdatabaseYsAdmin";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.resData = res.Data;
          this.editForm = Object.assign({}, this.resData);
          var RolejsonData = JSON.parse(res.Data.Rolejson);
          RolejsonData.forEach(item => {
            this.RoleIdsInfo.push(item);
          });
          this.para.Data = "";
          this.para.Code = "GetListYsdatabaseYsDepartment";
          handlePost(this.para).then(res => {
            //获取部门列表
            if (res.IsSuccess == true) {
              this.departments = res.Data.List;

              this.para.Code = "GetListYsdatabaseYsRole";
              this.para.Data = "{}";
              handlePost(this.para).then(res => {
                //获取角色列表
                if (res.IsSuccess == true) {
                  this.roles = res.Data.List;
                }
              });
            }
          });
        }
      });
    },
    //删除
    onDelete(data) {
      this.$confirm("确认删除该记录吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Ids: [data.Id]
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
    //列表查询
    handleSearch(selectedKeys, confirm) {
      confirm();
      this.searchText = selectedKeys[0];
    },

    handleReset(clearFilters) {
      clearFilters();
      this.searchText = "";
    },
    //表单
    handleSubmit(e) {
      e.preventDefault();
      this.form.validateFields((err, values) => {
        if (!err) {
          this.editForm = values;
        }
      });
    },
    handleSelectChangeDepartment(value) {
      this.form.setFieldsValue({
        note: `Hi, ${value === "male" ? "man" : "lady"}!`
      });
    },
    //是否显示
    aSwitch(checked) {
      this.editForm.Issuper = checked;
    },
    //是否启用
    aState(checked) {
      this.editForm.State = checked;
    },
    //tabs
    callback(key) {},
    filterOption(inputValue, option) {
      return option.description.indexOf(inputValue) > -1;
    },
    handleChange(targetKeys, direction, moveKeys) {
      this.targetKeys = targetKeys;
    },
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
      // this.form.setFieldsValue({
      //   note: `Hi, ${value === 'male' ? 'man' : 'lady'}!`,
      // })
    },
    //窗口事件
    handleOk() {
      this.dialogFormVisibleIcon = false;
    },
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    handleOkButton() {
      this.dialogFormVisibleButton = false;
    },
    handleOkData() {
      this.dialogFormVisibleData = false;
    },
    //刷新页面
    Refresh() {
      this.filters = {};
      this.loadingRefresh = true;
      setTimeout(() => {
        this.loadingRefresh = false;
        this.page = 1;
        this.current = 1;
        this.getDataList();
      }, 1000);
    },
    //穿梭框
    handleChange(value, direction, movedKeys) {},
    //图标
    allotIcon() {
      this.dialogFormVisibleIcon = true;
    },
    allotRoles() {
      // this.buttonKey = data
      this.getMock();
      this.dialogFormVisibleRoles = true;
    },
    //分配二维权限
    allotMent() {
      this.dialogFormVisibleData = true;
    },
    //
    onExpand(expandedKeys) {
      this.expandedKeys = expandedKeys;
      this.autoExpandParent = false;
    },
    onCheck(checkedKeys) {
      this.checkedKeys = checkedKeys;
    },
    // onSelect (selectedKeys, info) {
    //   this.selectedKeys = selectedKeys
    // },
    onSelect(selectedKeys, info) {
      this.TreeId = selectedKeys[0];
      this.getTreeList(this.TreeId);
    },
    //
    nodeClick(data9) {},
    changeClick(data) {},
    handleNodeClick(data) {},
    //行点击事件
    Rowdblclick(row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.disabledZhangHao = true;
      this.disabledMima = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id
      };
      this.para.Code = "GetYsdatabaseYsAdmin";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);

          this.para.Data = "";
          this.para.Code = "GetListYsdatabaseYsDepartment";
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.departments = res.Data.List;
              this.para.Code = "GetListYsdatabaseYsRole";
              handlePost(this.para).then(res => {
                if (res.IsSuccess == true) {
                  this.roles = res.Data.List;
                }
              });
            }
          });
        }
      });
    },
    //窗口控制
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    //多选角色
    handleChangeSelect(value) {
      this.RoleIdsInfo = value;
      // this.editForm.RoleIds = value
    },
    popupScroll() {},

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
    // 超管显示转换
    formatIsAdmin: function(row, column) {
      return row.Issuper === true ? "是" : "否";
    },
    // 状态显示转换
    formatIsDisabled: function(row, column) {
      return row.State === false ? "禁用" : "启用";
    },
    handleCurrentChange(val) {
      this.page = val;
      this.getDataList();
    },
    //查询
    getKeyList() {
      this.page = 1;
      this.getDataList();
    },

    YsAdminGetInfo() { //获取按钮
      this.para.Code = "YsAdminGetInfo";
      this.para.Data = "";
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.roleButtonItem = res.Data;
          this.MenuButtons = this.roleButtonItem.MenuButtons;
          this.MenuButtons.forEach(item => {
            if (item.MenuId == 18) {
              this.MenuButtonsData = item.Button;
            }
          });
        }
      });
    },

    // 获取用户列表
    getDataList() {
          const paras = {};
          this.para.Code = "GetListYsdatabaseYsButton";
          this.para.Data = JSON.stringify(paras);
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.buttonList = res.Data.List;
              this.buttonList.map(car => {
                if (car.Name == "添加") {
                  this.ButtonIcons.add = car.Icon;
                  this.ButtonNames.add = car.Name;
                }
                if (car.Name == "编辑") {
                  this.ButtonIcons.edit = car.Icon;
                  this.ButtonNames.edit = car.Name;
                }
                if (car.Name == "批量删除") {
                  this.ButtonIcons.del = car.Icon;
                  this.ButtonNames.del = car.Name;
                }
                if (car.Name == "刷新") {
                  this.ButtonIcons.refresh = car.Icon;
                  this.ButtonNames.refresh = car.Name;
                }
                if (car.Name == "查询") {
                  this.ButtonIcons.query = car.Icon;
                  this.ButtonNames.query = car.Name;
                }
              });
            }

            this.selectedRowKeys = [];
            var dataSource = this.selectValue;
            //取列表
            const paraId = [
              {
                Page: this.page,
                Data: this.filters.data,
                Size: this.size
              }
            ];

            var keyMap = {
              Data: dataSource
            };

            for (var i = 0; i < paraId.length; i++) {
              var obj = paraId[i];
              for (var key in obj) {
                var newKey = keyMap[key];
                if (newKey) {
                  obj[newKey] = obj[key];
                  delete obj[key];
                }
              }
            }

            this.para.Code = "GetListYsdatabaseYsAdmin";
            this.para.Data = JSON.stringify(paraId[0]);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                const description = {
                  description: "123"
                };
                this.total = res.Data.Count;
                this.users = res.Data.List;
                this.users.map(car => {
                  // var obj = {};
                  var key = "description";
                  var value = "添加";
                  eval("car." + key + "='" + value + "'");
                });
              }
              this.para.Data = "";
              this.para.Code = "GetListYsdatabaseYsDepartment";
              handlePost(this.para).then(res => {
                if (res.IsSuccess == true) {
                  this.departments = res.Data.List;
                  this.para.Code = "GetListYsdatabaseYsRole";
                  handlePost(this.para).then(res => {
                    if (res.IsSuccess == true) {
                      this.userRoles = res.Data.List;
                      //获取多菜单按钮
                      const paraId = {
                        MenuId: 18
                      };
                      this.para.Code = "GetYsMenuButton";
                      this.para.Data = JSON.stringify(paraId);
                      handlePost(this.para).then(res => {
                        if (res.IsSuccess == true) {
                          this.buttonAr = res;
                          this.allotButton();
                        }
                      });
                    }
                  });
                }
              });
              //取部门树
              const paraTree = {
                // Page: this.page,
                // Size: 10
              };
              // this.dataList = [];
              this.para.Code = "GetTreeYsdatabaseYsDepartment";
              this.para.Data = JSON.stringify(paraTree);
              handlePost(this.para).then(res => {
                if (res.IsSuccess == true) {
                  this.dataList = res.Data;
                  const paraData = {
                    Del: "删除",
                    Edit: "编辑",
                    Key: "",
                    title: "全部"
                  };
                  this.dataList.unshift(paraData);
                  this.YsAdminGetInfo() //获取按钮
                }
              });
            });
          });  
          

          //获取按钮END
      //   }
      // });
    },
    // 按部门查询
    getTreeList(data) {
      //取列表
      var numberData = parseInt(data);
      if (data.length < 1) {
        const paraSelect = {
          Page: this.page,
          Size: 10
        };
        this.para.Code = "GetListYsdatabaseYsAdmin";
        this.para.Data = JSON.stringify(paraSelect);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.users = res.Data.List;
          }
        });
      } else {
        const paraSelect = {
          DepartmentId: numberData,
          Page: this.page,
          Size: 10
        };
        this.para.Code = "GetListYsdatabaseYsAdmin";
        this.para.Data = JSON.stringify(paraSelect);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.users = res.Data.List;
          } else {
            this.$message({
              message: res.Code + ":" + "该部门用户" + res.Message,
              type: "warning"
            });
          }
        });
      }
    },

    // 删除
    handleDel(index, row) {
      this.$confirm("确认删除该记录吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          const paraId = {
            id: row.id
          };
          this.para.Code = this.bllCode.del;
          this.para.Data = JSON.stringify(paraId);
          handlePost(this.para)
            .then(res => {
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
            })
            .catch(err => {});
        })
        .catch(err => {});
    },
    // 显示编辑界面
    handleEdit(index, row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.disabledZhangHao = true;
      this.disabledMima = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id
      };
      this.para.Code = "GetYsdatabaseYsAdmin";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);
          // this.dataList = res.Data;

          this.para.Data = "";
          this.para.Code = "GetListYsdatabaseYsDepartment";
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.departments = res.Data.List;
              this.para.Code = "GetListYsdatabaseYsRole";
              this.para.Data = "{}";
              handlePost(this.para).then(res => {
                if (res.IsSuccess == true) {
                  this.roles = res.Data.List;
                } else {
                  this.$message({
                    message: res.Code + ":" + res.Message,
                    type: "warning"
                  });
                }
              });
            }
          });
        }
      });
    },
    // 显示新增界面
    handleAdd() {
      this.dialogStatus = "create";
      this.dialogFormVisibleAdd = true;
      this.disabledZhangHao = false;
      this.disabledMima = false;
      this.editForm = {
        Issuper: true,
        State: true
      };
      this.para.Code = "GetListYsdatabaseYsDepartment";
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.departments = res.Data.List;
          this.para.Code = "GetListYsdatabaseYsRole";
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.roles = res.Data.List;
            } else {
              this.$message({
                message: res.Code + ":" + res.Message,
                type: "warning"
              });
            }
          });
        }
      });
    },
    // 编辑
    updateData() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          })
            .then(() => {
              this.editForm.DepartmentId = this.editForm.Departmeid;
              delete this.editForm.RoleIds;
              this.editForm.RoleIds = this.RoleIdsInfo;
              const paraObj = Object.assign({}, this.editForm);
              this.para.Data = JSON.stringify(paraObj);
              this.para.Code = "UpdateYsdatabaseYsAdmin";
              handlePost(this.para)
                .then(res => {
                  if (res.IsSuccess == true) {
                    this.$refs["editForm"].resetFields();
                    this.dialogFormVisibleEdit = false;
                    this.getDataList(); //刷新列表
                    this.$message({
                      message: "修改成功！",
                      type: "success"
                    });
                  } else {
                    this.$message({
                      message: res.Code + ":" + res.Message,
                      type: "warning"
                    });
                  }
                })
                .catch(err => {});
            })
            .catch(e => {
              // 打印一下错误
            });
        }
      });
    },
    // 新增
    createData: function() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }).then(() => {
            const paraObj = Object.assign({}, this.editForm);
            this.para.Data = JSON.stringify(paraObj);
            this.para.Code = "AddYsdatabaseYsAdmin";
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.$refs["editForm"].resetFields();
                this.dialogFormVisibleAdd = false;
                this.getDataList();
                this.$message({
                  message: "添加成功！",
                  type: "success"
                });
              } else {
                this.$message({
                  message: res.Code + ":" + res.Message,
                  type: "warning"
                });
              }
            });
          });
        }
      });
    },
    // 全选单选多选
    selsChange(sels) {
      this.sels = sels;
    },
    // 批量删除
    batchRemove() {
      var ids = this.sels.map(item => item.id);
      this.$confirm("确认删除选中记录吗？", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            ids: ids
          };
          this.para.Data = JSON.stringify(paraId);
          this.para.Code = this.bllCode.del;
          handlePost(this.para)
            .then(res => {
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
            })
            .catch(err => {});
        })
        .catch(err => {});
    }
  },
  mounted() {
    this.loadButton(store.getters.interfaces); //按权限加载按钮
    this.getDataList();
  }
};
</script>

<style scoped>
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}
/* .el-card__header{
    padding: 12px 14px;
} */
.box-card {
  /* width: 480px; */
  /* height: 68rem; */
}
/* ----------------------------------- */
.custom-filter-dropdown {
  padding: 8px;
  border-radius: 6px;
  background: #fff;
  box-shadow: 0 1px 6px rgba(0, 0, 0, 0.2);
}

.custom-filter-dropdown input {
  width: 130px;
  margin-right: 8px;
}

.custom-filter-dropdown button {
  margin-right: 8px;
}

.highlight {
  color: #f50;
}
/* ----------------条件绑定 */
.p1 {
  display: none;
}
.p {
  color: blue;
}
</style>
