<template>
  <section class="app-container">
    <el-card class="box-card">
      {{buttonArs}}
      <hr>
      {{buttonArList}}
      <!--工具条-->
      <el-form :inline="true" :model="filters" @submit.native.prevent>
        <!-- allotButtons:过滤好的 {{allotButtons}}
        <hr>
        buttonList全部:{{buttonList}}
        <hr>
        buttonAr拥有的：{{buttonAr}} -->
        <span v-for="index in allotButtons" :key="index.Id">
        <a-button :disabled='buttonDisabled(index.Classname)' :class="setClass(index.Classname)" style="margin-right:.3rem"  :icon="index.Icon"  @click="defaultClick(index)" type="primary" >{{index.Name}}</a-button>
        </span>

        <a-button type="primary" v-if="isShowButton.add" @click="handleAdd" :icon="buttonList[0].Icon">{{buttonList[0].Name}}</a-button>
        <a-button type="primary" v-if="isShowButton.Refresh" :loading="loadingRefresh" :icon="buttonList[1].Icon" @click="Refresh">{{buttonList[1].Name}}</a-button>       

      <el-form-item style="float: right;">
        <span v-for="index in allotButtons" :key="index.Id">
          <a-button v-if="index.Classname==='search'" :icon="index.Icon"  @click="defaultClick(index)" type="primary" >{{index.Name}}</a-button>
        </span>
        </el-form-item>

        <el-form-item style="float: right;">
          <a-input-group compact>
            <a-select  @change="this.handleSelectChange" defaultValue="名称" style="width: 40%">
                <!-- <a-select-option value='Id'>Id</a-select-option> -->
                <a-select-option value='Name'>名称</a-select-option>
                <a-select-option value='Sort'>排序</a-select-option>
                <a-select-option value='Memo'>备注</a-select-option>
            </a-select>
          <a-input style="width: 60%" defaultValue="" v-model="filters.data"/>
        </a-input-group>
        </el-form-item>
      </el-form>


        <a-table
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :pagination="false"
        :dataSource="dataList"
        :columns="columns"
        @onSelect='onSelectTable'
      >
    <div slot="filterDropdown" slot-scope="{ setSelectedKeys, selectedKeys, confirm, clearFilters }" class='custom-filter-dropdown'>
      <a-input
        ref="searchInput"
        placeholder='请输入名称'
        :value="selectedKeys[0]"
        @change="e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
        @pressEnter="() => handleSearch(selectedKeys, confirm)"
      />
      <a-button type='primary' @click="() => handleSearch(selectedKeys, confirm)">快速定位</a-button>
      <a-button @click="() => handleReset(clearFilters)">取消</a-button>
    </div>
    <a-icon slot="filterIcon" slot-scope="filtered" type='tag' :style="{ color: filtered ? '#108ee9' : '#aaa' }" />
    <template slot="customRender" slot-scope="text">
      <span v-if="searchText">
        <template v-for="(fragment, i) in text.split(new RegExp(`(?<=${searchText})|(?=${searchText})`, 'i'))">
          <span v-if="fragment.toLowerCase() === searchText.toLowerCase()" :key="i" class="highlight">{{fragment}}</span>
          <template v-else>{{fragment}}</template>
        </template>
      </span>
      <template v-else>{{text}}</template>
    </template>
    <template slot="statu" slot-scope="text,record">
        <a-badge v-if="record.Isvisiable" status="success" text="正常" />
    </template>
    <template slot="action" slot-scope="text, record">
            <!-- <a href="javascript:;" v-if="isShowButton.edit" @click="onEdit(record)">编辑</a>
            <a href="javascript:;" v-if="isShowButton.del" @click="onDelete(record)">删除</a> -->
            <!-- <a href="javascript:;" @click="allotMent(record)">权限</a> -->
            <!-- <a href="javascript:;" @click="onDelete(record)">详情</a> -->
            <a href="javascript:;" @click="allotMent(record)">权限</a>
            <span  v-for="index in allotButtons" :key="index.Id">
            <a href="javascript:;" v-if="index.Classname==='edit'"  @click="onEdit(record.Id)">{{index.Name}}</a>
            <a href="javascript:;" v-if="index.Classname==='del'"  @click="onDelete(record)">{{index.Name}}</a>
            </span>
          </template>




  </a-table>

      <a-pagination style="margin-top:2rem;text-align: right;" 
    showSizeChanger
     showQuickJumper 
     v-model="current" 
     :total="total"
     :showTotal="(total, range) => ` 共${total}条记录 第 ${range[0]}/${range[1]}页` "
      @showSizeChange="onShowSizeChange" />



        <!--二维权限-->
    <a-modal class="allotMent" title="分配权限" @ok="handleOkData" @click="allotMent" v-model="dialogFormVisibleData">
        <a-button type="primary" @click="getRole" >获取</a-button>
        <a-button type="primary" @click="setRole" >设置</a-button>
    <!-- <br> -->
        菜单列表:{{menuList}}
      <!-- {{treeDatas}} -->
      <hr>
      获取单个角色权限:{{userRole}}

        <!-- 导航菜单：{{treeData}} -->

          <template>
            <a-tree
              checkable
              :treeData="menuList"
              @select="this.onSelect"
              @check="this.onCheck"
            >
            <span slot="title0010" style="color: #1890ff">sss</span>
            </a-tree>
          </template>
      

      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleData=false">取消</a-button>
        <a-button type="primary" @click.native="dialogFormVisibleData=false">确认</a-button>
      </div>
    </a-modal>

            <!--详情-->
    <!-- <a-modal class="allotMent" title="详情" @ok="handleOkData" @click="allotMent" v-model="dialogFormVisibleInfo">
        <a-button type="primary" @click="getRole" >获取</a-button>
        <a-button type="primary" @click="setRole" >设置</a-button>
      
          <template>
            <a-form-item v-for="i in jurisdiction" :label='i.label' :key="i.Id" :labelCol="{ span: 3 }">
                <a-checkbox-group :options="i.all" v-model="i.part" @change="onChange(i.all,i.part,i.Id,i)" />
            </a-form-item>
      
          </template>
      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleInfo=false">取消</a-button>
        <a-button type="primary" @click.native="dialogFormVisibleInfo=false">确认</a-button>
      </div>
    </a-modal> -->

    <!--按钮-->
    <el-dialog title="添加" :visible.sync="dialogFormVisibleButton" :close-on-click-modal="false">
             <div style="text-align: center" class="transferBox">
          <el-transfer
          
            style="text-align: left; display: inline-block"
            v-model="value3"
            filterable
            filter-placeholder="请输入搜索内容"
            :left-default-checked="[2, 3]"
            :right-default-checked="[1]"
            :render-content="renderFunc"
            :titles="['所有菜单', '已有菜单']"
            
            :format="{
              noChecked: '${total}',
              hasChecked: '${checked}/${total}'
            }"
            @change="handleChange"
            :data="data">
            <a-button class="transfer-footer" slot="left-footer" size="small">操作</a-button>
            <a-button class="transfer-footer" slot="right-footer" size="small">操作</a-button>
          </el-transfer>
      </div>
      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleButton=false">取消</a-button>
        <a-button type="primary" @click.native="dialogFormVisibleButton=false">确认</a-button>
      </div>
    </el-dialog>

    <!--添加界面-->
    <a-modal title="添加角色" @ok="handleOkAdd" @click="createData" v-model="dialogFormVisibleAdd">
      <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
        <el-form-item label="角色名称:" prop="Name">
          <el-input v-model="editForm.Name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="排序:">
          <el-input-number v-model="editForm.Sort"></el-input-number>
        </el-form-item>
        <el-form-item label="备注:" prop="Memo">
          <el-input v-model="editForm.Memo" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleAdd=false">{{button.cancel}}</a-button>
        <a-button type="primary" @click="createData">{{button.add}}</a-button>
      </div>
    </a-modal>

    <!--编辑界面-->
    <a-modal title="编辑角色" @ok="handleOkEdit" @click="updateData" v-model="dialogFormVisibleEdit">
      <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
        <el-form-item label="角色名称:" prop="Name">
          <el-input v-model="editForm.Name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="排序:">
          <el-input-number v-model="editForm.Sort"></el-input-number>
        </el-form-item>
        <el-form-item label="备注:" prop="Memo">
          <el-input v-model="editForm.Memo" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleEdit=false">{{button.cancel}}</a-button>        
        <a-button type="primary" @click="updateData">{{button.modify}}</a-button>
      </div>
    </a-modal>
    <a-divider orientation="left">角色规则</a-divider>
    <p>角色组可以有多个,角色有上下级层级关系,如果子角色有角色组和管理员的权限则可以派生属于自己组别的下级角色组或管理员</p>
    <a-divider dashed />
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

const treeData = [{
  title: 'parent 1',
  key: '0-0',
  children: [{
    title: 'parent 1-0',
    key: '0-0-0',
    disabled: true,
    children: [
      { title: 'leaf', key: '0-0-0-0', disableCheckbox: true },
      { title: 'leaf', key: '0-0-0-1' },
    ],
  }, {
    title: 'parent 1-1',
    key: '0-0-1',
    children: [
      { key: '0-0-1-0', slots: { title: 'title0010' }},
      { key: '0-0-1-1', slots: { title: 'title0010' }},
    ],
  }],
}]

//模拟数据
const treeDatas = 

[ { key: "1", Icon: "anticon anticon-inbox", Code: "sys", Url: "sys_manage", Sort: "0", Show: "√", children: 
[ { key: "3", Icon: "anticon anticon-inbox", Code: "button", Url: "/views/button/index", Sort: "1", Show: "√",

 children: [//此菜单下拥有的权限按钮
  { key: "90",title: "添加" },
  { key: "91",title: "删除" },
  { key: "92",title: "编辑" },
  { key: "93",title: "查询" }, 
  ],


  Edit: "编辑", Del: "删除", title: "1操作按钮" }, 
{ key: "17", Icon: "anticon anticon-desktop", Code: "rolemanagement", Url: "/views/rolemanagement/index", Sort: "1", Show: "√", 

children: [//此菜单下拥有的权限按钮
  { key: "90",title: "添加" },
  { key: "91",title: "删除" },
  { key: "92",title: "编辑" },
  { key: "93",title: "查询" }, 
  ],


Edit: "编辑", Del: "删除", title: "角色管理" },
 { key: "18", Icon: "anticon anticon-user", Code: "usermanagement", Url: "/views/usermanagement/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "用户管理" }, 
 { key: "19", Icon: "anticon anticon-solution", Code: "divisionmanage", Url: "/views/divisionmanage/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "部门管理" }, 
 { key: "20", Icon: "anticon anticon-exception", Code: "datadictionary", Url: "/views/datadictionary/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "数据字典" },
  { key: "21", Icon: "anticon anticon-save", Code: "systemsetup", Url: "/views/systemsetup/index", Sort: "1", Show: "X", children: null,Edit: "编辑", Del: "删除", title: "参数设置" },
   { key: "38", Icon: "anticon anticon-export", Code: "operationlog", Url: "/views/operationlog/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "操作日志" }, 
   { key: "40", Icon: "anticon anticon-setting", Code: "configure", Url: "/views/configure/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "系统配置" }, 
   { key: "1064", Icon: "anticon anticon-file-unknown", Code: "test", Url: "/views/test/index", Sort: "1", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "test1" },
    { key: "2", Icon: "anticon anticon-database", Code: "navigationmenu", Url: "/views/navigationmenu/index", Sort: "2", Show: "√", children: null,Edit: "编辑", Del: "删除", title: "导航菜单" } ],
    Edit: "编辑", Del: "删除", title: "系统设置" } ]

//树形选择

const treeData123 = [{
  title: '系统配置',
  key: '0-0',
  children: [{
    title: '导航菜单',
    key: '0-0-0',
    children: [
      { title: '添加', key: '0-0-0-0' },
      { title: 'del', key: '0-0-0-1' },
      { title: 'edit', key: '0-0-0-2' },
    ],
  }, {
    title: '操作按钮',
    key: '0-0-1',
    children: [
      { title: 'add', key: '0-0-1-0' },
      { title: 'DEL', key: '0-0-1-1' },
      { title: 'DEIT', key: '0-0-1-2' },
    ],
  }],
}, ]

const columnsTree = [
//   {
//   dataIndex: 'name',
//   key: 'name',
//   slots: { title: 'customTitle' },
// }
,{
  title: '菜单名称',
  key: 'name',
  dataIndex: 'name',
},{
  title: '浏览',
  key: 'icon',
  dataIndex: 'icon',
  scopedSlots: { customRender: 'tags' },
},{
  title: '添加',
  key: 'code',
  dataIndex: 'code',
  scopedSlots: { customRender: 'tags' },
},{
  title: '下话单',
  key: 'url',
  dataIndex: 'url',
  scopedSlots: { customRender: 'tags' },
},{
  title: '重发通知',
  key: 'show',
  dataIndex: 'show',
  scopedSlots: { customRender: 'tags' },
},{
  title: '重置',
  key: 'sort',
  dataIndex: 'sort',
  scopedSlots: { customRender: 'tags' },
},{
  title: '编辑',
  key: 'action',
  dataIndex: 'action',
  scopedSlots: { customRender: 'tags' },
  // scopedSlots: { customRender: 'action' },
},{
  title: '删除',
  key: 'del',
  dataIndex: 'del',
  scopedSlots: { customRender: 'tags' },
  // scopedSlots: { customRender: 'action' },
},{
  title: '查询',
  key: 'query',
  dataIndex: 'query',
  scopedSlots: { customRender: 'tags' },
  // scopedSlots: { customRender: 'action' },
},{
  title: '导出',
  key: 'export',
  dataIndex: 'export',
  scopedSlots: { customRender: 'tags' },
  // enable:
  // scopedSlots: { customRender: 'action' },
}];

const dataTree = [{
  key: 1,
  name: '系统设置',
  icon:'icon',
  code:'Button',
  url:'sys/ButtonList',
  sort:'1',
  age: 60,
  edit:'编辑',
  del:'删除',
  children: [{
    key: 11,
    name: '导航菜单',
    icon:'icon',
    code:'Button',
    url:'sys/ButtonList',
    sort:'1',
    age: 42,
    edit:'编辑',
    del:'删除',
  }, {
    key: 12,
    name: '用户管理',
    icon:'icon',
    code:'Button',
    url:'sys/ButtonList',
    sort:'1',
    age: 30,
    edit:'编辑',
    del:'删除',
  }, {
    key: 13,
    name: '部门管理',
    icon:'icon',
    code:'Button',
    url:'sys/ButtonList',
    sort:'1',
    age: 72,
    edit:'编辑',
    del:'删除',
  }],
}, {
  key: 2,
  name: '财务管理',
  icon:'icon',
  code:'Button',
  url:'sys/ButtonList',
  sort:'1',
  age: 32,
  edit:'编辑',
  del:'删除',
  children: [{
    key: 22,
    name: '奖金明细',
    icon:'icon',
    code:'Button',
    url:'sys/ButtonList',
    sort:'1',
    age: 42,
    edit:'编辑',
    del:'删除',
  }
  ]
}];

const rowSelectionTree = {
  onChange: (selectedRowKeys, selectedRows) => {
    console.log(`selectedRowKeys: ${selectedRowKeys}`, 'selectedRows: ', selectedRows);
  },
  onSelect: (record, selected, selectedRows) => {
    console.log(record, selected, selectedRows);
  },
  onSelectAll: (selected, selectedRows, changeRows) => {
    console.log(selected, selectedRows, changeRows);
  },
};

//多选初始化数组
// const plainOptions = ['Apple', 'Pear', 'Orange']
const plainOptions = [
  { label: 'Apple', value: 'Apple1' },
  { label: 'Pear', value: 'Pear1' },
  { label: 'Orange', value: 'Orange1' },
]
const defaultCheckedList = []

export default {
  //分页触发
      watch:{
      pageSize(val) {
        console.log('pageSize',val);
      },
      current(val) {
        console.log('current',val);
        this.page = val;
        this.getDataList();
      }
    },
  data() {
        // 穿梭框
        const generateData = _ => {
        const data = [];
        
        for (let i = 1; i <= 10; i++) {
          data.push({
            key: i,
            label: `备选项 ${ i }`,
            disabled: i % 4 === 0
          });
        }
        return data;
      };
    return {
      //权限按钮
      roleBotton:[],
      treeData,
      treeDatas,
      menuList:[],
      //按钮
      ButtonData:[],
      ButtonIcons: {},
      ButtonNames: {},
      buttonList: [],
      buttonAr:[],
      buttonArs:[],
      buttonArList:[],
      allotButtons:[],
      //树形选择
      expandedKeys: ['0-0-0', '0-0-1'],
      autoExpandParent: true,
      checkedKeys: ['0-0-0'],
      selectedKeys: [],
      treeData123,

      //获取到权限数组
      userRole:[],
      //按钮显示隐藏
      isShowButton:{},
      //批量选择
      selectedRowKeys: [], // Check here to configure the default column
      selectedRows:[],
      loading: false,
      loadingRefresh: false,

      //分页
      current:1,
      //列表
      // dataButton,
      searchText: '',
                  columns: [{
        title: '名称',
        dataIndex: 'Name',
        key: 'Name',
        scopedSlots: {
          filterDropdown: 'filterDropdown',
          filterIcon: 'filterIcon',
          customRender: 'customRender',
        },
        onFilter: (value, record) => record.name.toLowerCase().includes(value.toLowerCase()),
        onFilterDropdownVisibleChange: (visible) => {
          if (visible) {
            setTimeout(() => {
              this.$refs.searchInput.focus()
            })
          }
        },
      }, {
        title: '编号',
        dataIndex: 'Id',
        key: 'Id',
      },{
        title: '排序',
        dataIndex: 'Sort',
        key: 'Sort',
      },
      {
        title: '备注',
        dataIndex: 'Memo',
        key: 'Memo',
      },

      // { title: '显示状态', dataIndex: 'Isvisiable', key: 'Isvisiable', scopedSlots: { customRender: 'statu' } },
      {
        title: '操作',
        Key: 'action',
        dataIndex: 'action',
        scopedSlots: { customRender: 'action' },
        width: 200
      }, 
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
      
      checkboxModel:false,
      selectValue:'Name',
            //穿梭框
        data: generateData(),
        value3: [1],
        value4: [1],
        renderFunc(h, option) {
          return <span>{ option.key } - { option.label }</span>;
        },
      //搜索
      input3: '',
      input4: '',
      input5: '',
      select: '',

      // tree列表
      dataTree,
      columnsTree,
      rowSelectionTree,
      
      bllCode: {
        //接口标识，由后端提供
        add: "AddYsdatabaseYsRole", //添加
        edit: "UpdateYsdatabaseYsRole", //修改
        del: "DelYsdatabaseYsRole", //删除
        getList: "GetListYsdatabaseYsRole", //获取列表
        getObj: "GetYsdatabaseYsRole", //获取对象（单个）
        getRolesList: "GetListYsdatabaseYsRole" //获取角色
      },
      tableLabel: [
        { type: "selection", width: "50" },
        { Label: "ID", prop: "id", width: "50", type: "index" },
        { Label: "名称", prop: "Name", width: "100" },
        // { Label: "上级菜单", prop: "Pid", width: "80" },
        // { Label: "链接地址", prop: "Url"},
        // { Label: "页面标识", prop: "Param",width: "150"},
        // { Label: "图标", prop: "Icon", width: "50" },
        { Label: "排序", prop: "Sort", width: "50" },
        // {Label:'是否管理后台',prop:"shifouguanlihoutai",width:'150'},
        // {Label:'是否需要登录',prop:"shifouxuyaodenglu",width:'150'},
        // {Label:'创建时间',prop:"beizhu",width:'150'},
        {Label:'备注',prop:"Memo"},
      ],
      filtersName: "角色名称",
      button: {
        query: "查询",
        add: "添加",
        batchRemove: "批量删除",
        edit: "编辑",
        del: "删除",
        cancel: "取消",
        modify: "修改"
      },

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
      dialogFormVisibleData:false,
      dialogFormVisibleButton:false,
      dialogFormVisibleIcon:false,
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleInfo:false,
      ListsuperiorMenu: [],
      dataList: [], //主页数据
      //分页初始化
      total: 0,
      page: 1,
      size:10,

      sels: [], // 列表选中列
      editFormRules: {
        Name: [
          {
            required: true,
            message: "名称必填",
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
        Url:'',
        id: "",
        shangjiid: "",
        lianjie: "",
        shifouxianshi: "",
        paixu: "",
        tubiao: ""
      },

      filterdataListData: [],
    //搜索
    handleSelectChange (value) {
      console.log (value)
      this.selectValue = value
      console.log (this.selectValue)
      // this.form.setFieldsValue({
      //   note: `Hi, ${value === 'male' ? 'man' : 'lady'}!`,
      // })
    },
      //查询条件
      filters: {
      },
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
    computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0
    }
  },


  watch: {
    checkedKeys(val) {
      console.log('onCheck', val)
    }
  },

  methods: {
    //按钮权限
    buttonDisabled(index){
      if(index === 'inport'){
      return true
      }
    },

        //转列表对象值
    // carButton() {
    //   var buttonkeyMap = {
    //     Name: "title",
    //   };

    //   for (var i = 0; i < this.menuList.length; i++) {
    //     var obj = this.menuList[i];
    //     for (var key in obj) {
    //       var newKey = buttonkeyMap[key];
    //       if (newKey) {
    //         obj[newKey] = obj[key];
    //         delete obj[key];
    //       }
    //     }
    //   }

    // },

    setClass(index) {
      if(index === 'edit'){
      return 'p1'
      }
      if(index === 'search'){
      return 'p1'
      }

    },
    onSelectTable(){
      alert (1)
    },
    //过滤按钮
    allotButton() {
      this.allotButtons = []
      var ButtonDatas = [];
      this.buttonAr.Data[0].ButtonIds.map((car)=>{//拥有的按钮ID集
      const allotButton = this.buttonList.find((i)=>{//和全部的筛选对象
        return i.Id === car
      })
        ButtonDatas.push(allotButton)
    })
    console.log ('ButtonDatas00000::',ButtonDatas)
    this.allotButtons = ButtonDatas.filter((e)=>{
            return e
          })
            console.log ('eeeeeeeee',this.allotButtons)
    },
        //默认点击事件
    defaultClick(index){
      console.log (index)
      switch(index.Classname){
            case 'add':
            this.handleAdd()
            break;
            case 'refresh':
            this.Refresh()
            break;
            case 'search':
            this.getKeyList()
            break;
            case 'del':
            this.start()
            break;
            case 'edit':
            if(this.selectedRowKeys >0){
            this.onEdit(this.selectedRows[0])
            }else {
          this.$message({
            message: '请选择一个需要编辑的按钮',
            type: "warning"
          });
        }
            break;
                 
          }
},
      //树形选择
      onSelect (selectedKeys, info) {
      console.log('selected', selectedKeys, info)
    },
    onCheck (checkedKeys, info) {

      console.log('onCheck', checkedKeys, info)

      this.roleBotton = []
      info.checkedNodes.forEach((i)=>{
        // console.log ('lsk',i.data.props.Id)
        this.roleBotton.push(i.data.props.Id)
      })
      console.log ('rrrrr',this.roleBotton)
      // console.log('info', info.halfCheckedKeys)
    },

    getRole(i){
      this.userRole = []
        const paraId = [
          {
            Id: i,
            // Power: [{
            //   MenuId: 19,	//菜单id
            //   Buttons: [1,38,39]	//按钮id数组
            // }]
          }
        ];
        this.para.Code = "GetYsRolePower";
        this.para.Data = JSON.stringify(paraId[0]);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true ) {
            console.log (res)

            this.userRole = res.Data.Power
          }
        //   // if(typeof res.Data === "null"){
        //   //   alert(1)
        //   // }
        //   // if(typeof res.Data === null){
        //   //   alert(1)
        //   // }
        //   // if(typeof res.Data == "null"){
        //   //   alert(1)
        //   // }
        });
    },
    setRole(){
      this.$confirm("确认为该菜单分配按钮吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      }).then(() => {
        const paraId = [
          {
            Pid: 1,
            Id: 1030,
            Power: [{
              MenuId: 19,	//菜单id
              Buttons: [1,38,39]	//按钮id数组
            },
            {
              MenuId: 20,	//菜单id
              Buttons: [1,38]	//按钮id数组
            },]
          }
        ];
        this.para.Code = "SetYsRolePower";
        this.para.Data = JSON.stringify(paraId[0]);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.dialogFormVisibleData = false;
            this.getDataList()
            this.$message({
              message: "分配成功！",
              type: "success"
            });
          }
        });
      });
    },
        //分页操作
    onShowSizeChange(current, pageSize) {
        console.log('111',current, pageSize);
        // this.page = val;
        this.page = current;
        this.size = pageSize;
        this.getDataList();
      },
        //批量选择
    start () {
      this.loading = true;
      // ajax request after empty completing
      this.$confirm("确认执行删除操作吗？", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
      .then(() => {
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
            this.selectedRowKeys = []
            }
          });

      }, 1000);
        })
    },
        //列表查询
    handleSearch (selectedKeys, confirm) {
      confirm()
      this.searchText = selectedKeys[0]
    },

    handleReset (clearFilters) {
      clearFilters()
      this.searchText = ''
    },
                // 显示编辑界面
    onEdit(row) {
      // ----------
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row,
      }; 
      this.para.Code = 'GetYsdatabaseYsRole';
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
      this.editForm = Object.assign({}, res.Data);


        }
      });
    },
    //表记录删除
      onDelete (data) {
      console.log (data)
        this.$confirm("确认删除该记录吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Id: data.Id
          };
          this.para.Code = 'DelYsdatabaseYsRole';
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
        //列表
    handleSearch (selectedKeys, confirm) {
      confirm()
      this.searchText = selectedKeys[0]
    },

    handleReset (clearFilters) {
      clearFilters()
      this.searchText = ''
    },
     onSelectChange (selectedRowKeys,selectedRows) {
      this.selectedRows = []
      console.log('selectedRowKeys changed: ', selectedRowKeys,selectedRows);
      this.selectedRowKeys = selectedRowKeys
       selectedRows.forEach(car =>{
        this.selectedRows.push(car.Id)
      })
      console.log (this.selectedRows)
      
    },
    //表开关
    onChangeSwitch(text, record, index){
      console.log (text, record, index)
      // console.log (text, record, index)
      // console.log(`checked = ${e.target.checked}`)

            // console.log('setIsCheckInterface--rowdata===',rowdata)
      // this.paraSwitch.Code = this.bllCode.isCheck;
      // let data = {
      //   id: rowdata.id,
      //   shifouyanzheng: rowdata.shifouyanzheng
      // };
      // this.paraSwitch.Data = JSON.stringify(data);
      // handlePost(this.paraSwitch).then(res => {
      //   if (res.IsSuccess == true) {
      //     this.getDataList();
      //   }
      // });
    },
    onChangeClick(e){
      console.log(`checked = ${e.target.checked}`)
      console.log (this.checkboxModel)
    },
    //窗口事件
    handleOk() {
      this.dialogFormVisibleIcon = false;
    },
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    handleOkAdd(){
      this.dialogFormVisibleAdd = false;
    },
    handleOkData(){
      this.dialogFormVisibleData = false;
      this.dialogFormVisibleInfo = false;
    },
    Refresh() {
        this.filters = {}
        this.loadingRefresh = true;
        setTimeout(() => {
        this.loadingRefresh = false;
        this.page = 1
        this.current = 1
        this.getDataList();
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
    //修树KEY
    // trans (data) {
    //   data.forEach(item => {
    //     item.title = item.Name // 把title属性赋值给name属性
    //     delete item.Name // 删除title属性
    //     item.children && this.trans(item.children) // 如果有children，递归调用
    //   })
    // },

    //修树KEY
    trans (data) {

      // var buttonkeyMap = {
      //   children: "childrens",
      // };

      // for (var i = 0; i < data.length; i++) {
      //   var obj = data[i];
      //   for (var key in obj) {
      //     var newKey = buttonkeyMap[key];
      //     if (newKey) {
      //       obj[newKey] = obj[key];
      //       delete obj[key];
      //     }
      //   }
      // }


      var isFirst = true;

      if(isFirst){
      data.forEach(item => {
        item.title = item.Name 
        delete item.Name 
        item.children.forEach(i=>{
        delete i.children
        i.children = i.Buttons
        delete i.Buttons 
        i.title = i.Name 
        delete i.Name 
        i.children.forEach((index)=>{
          index.title = index.Name 
          delete index.Name
          index.value = index.Id
          // index.key = JSON.stringify(index.Id)
          // index.key = index.Id.toString() 
          // delete index.Id
        })
        })
        // item.title = item.Name // 把title属性赋值给name属性
        // delete item.Name // 删除title属性
        // item.children && this.trans(item.children) // 如果有children，递归调用
      })
      isFirst = false;
      }

    console.log ('data',data)
    },


    allotMent(i) {
      this.dialogFormVisibleData = true;
      const paraId = {
        ButtonShow:true
      }; 
      this.para.Code = 'GetListYsdatabaseYsMenu';
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res =>{
        this.menuList = res.Data
        this.trans(this.menuList)//修树KEY
        this.getRole(i.Id)//获取单个角色权限
      })
      // const paraId = {}; 
      // this.para.Code = 'GetListYsdatabaseYsMenu';
      // this.para.Data = JSON.stringify(paraId);
      // handlePost(this.para).then(res => {
      //   if (res.IsSuccess == true) {
      //     this.treeData = res.Data
      //     console.log ('res',this.treeData)

      //   }
      // });

    },
    //行点击事件
    Rowdblclick(row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id,
      }; 
      this.para.Code = 'GetYsdatabaseYsRole';
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
      this.editForm = Object.assign({}, res.Data);
        }
      });
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

    // 删除
    // handleDel(index, row) {
    //   this.$confirm("确认删除该记录吗?", "提示", {
    //     type: "warning",
    //     confirmButtonText: "确定",
    //     cancelButtonText: "取消"
    //   })
    //     .then(() => {
    //       const paraId = {
    //         Id: row.id
    //       };
    //       this.para.Code = this.bllCode.del;
    //       this.para.Data = JSON.stringify(paraId);
    //       console.log(this.para.Data);
    //       handlePost(this.para).then(res => {
    //         if (res.IsSuccess == true) {
    //           this.getDataList();
    //         }
    //       });
    //     })
    //     .catch(() => {});
    // },
    handleCurrentChange(val) {
      this.page = val;
      this.getDataList();
    },
        //转按钮列表对象值
    carButton() {
      var buttonkeyMap = {
        Id: "value",
        Name: "label"
      };

      for (var i = 0; i < this.ButtonData.length; i++) {
        var obj = this.ButtonData[i];
        for (var key in obj) {
          var newKey = buttonkeyMap[key];
          if (newKey) {
            obj[newKey] = obj[key];
            delete obj[key];
          }
        }
      }
    },
    // 获取列表
    getDataList() {

          //初始化图标
          const paras = {};
          this.para.Code = 'GetListYsdatabaseYsButton';
          this.para.Data = JSON.stringify(paras);
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
            this.buttonList = res.Data.List;
            this.carButton()//转按钮列表对象值
          }

      this.selectedRowKeys = []
      var dataSource = this.selectValue
      const paraId = [{
        Page: this.page,
        Data: this.filters.data,
        Size: this.size
      }];

      var keyMap = {
            "Data" : dataSource,
        };

        for(var i = 0;i < paraId.length;i++){
                var obj = paraId[i];
                for(var key in obj){
                          var newKey = keyMap[key];
                          if(newKey){
                                    obj[newKey] = obj[key];
                                    delete obj[key];
                            }
                    }
        }

        
        
      this.para.Code = this.bllCode.getList;
      this.para.Data = JSON.stringify(paraId[0]);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.total = res.Data.Count;
          this.dataList = res.Data.List;
          // //添加对象 属性
          this.dataList.map((car)=>{
              // var obj = {};
              var key = "description";
              var value = "添加"
              eval("car." + key + "='" + value + "'");
            })

            //获取单菜单按钮
            const paraId = {
              MenuId:17,
            };
            this.para.Code = "GetYsMenuButton";
            this.para.Data = JSON.stringify(paraId);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.buttonAr = res;
                this.allotButton()
                this.GetYsMenuButtons()
                // this.GetListButton()
              }
            }); 
        } else {
            this.$message({
              message: res.Code + ":" + res.Message,
              type: "warning"
            });
          }
      });
    });
    },

    //获取多菜单按钮
    GetYsMenuButtons(){
      const paraIds = {

            };
            this.para.Code = "GetYsMenuButton";
            this.para.Data = JSON.stringify(paraIds);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.buttonArs = res;
                // this.allotButton()
                this.GetListButton()

              }
            }); 
    },

    //获取操作按钮列表
    GetListButton(){
      const paraIds = {

            };
            this.para.Code = "GetListYsdatabaseYsButton";
            this.para.Data = JSON.stringify(paraIds);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.buttonArList = res;
                // this.allotButton()
              }
            }); 
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
      this.editForm = {};
      const paraId = {
        Id: row.Id,
      }; 
      this.para.Code = 'GetYsdatabaseYsRole';
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
      this.editForm = Object.assign({}, res.Data);
        }
      });
    },

    // 显示添加界面
    handleAdd() {
      this.dialogStatus = "create";
      this.dialogFormVisibleAdd = true;
      // this.$refs["editForm"].resetFields(); //重置editForm
      this.editForm = {

      };

      let paert = {
        Pid: -1
      };
      this.para.Data = JSON.stringify(paert);
      this.para.Code = this.bllCode.getList;
      console.log(this.para);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.ListsuperiorMenu = res.Data.List;
          let top = {
            Id: 0,
            Name: "无"
          };
          this.ListsuperiorMenu.push(top);
        }
      });
    },
    // 编辑
    updateData() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消"
          }).then(() => {
            let paert = {
              shangjiid: -1
            };
            this.para.Data = JSON.stringify(this.paert);
            this.para.Code = this.bllCode.edit;
            this.para.Data = JSON.stringify(this.editForm);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.$refs["editForm"].resetFields(); //重置editForm
                this.dialogFormVisibleEdit = false;
                this.getDataList();
                this.$message({
                  message: "修改成功！",
                  type: "success"
                });
              } else {
                this.$refs["editForm"].resetFields();
                this.dialogFormVisibleEdit = false;
              }
            });
          });
        }
      });
    },
    //查询
    getKeyList() {
      this.page = 1;
      this.getDataList();
    },
    // 添加
    createData: function() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消"
          })
            .then(() => {
              this.para.Code = this.bllCode.add;
              this.para.Data = JSON.stringify(this.editForm);
              handlePost(paraHelper).then(res => {
                if (res.IsSuccess == true) {
                  this.$refs["editForm"].resetFields();
                  this.dialogFormVisibleAdd = false;
                  this.getDataList();
                  this.$message({
                    message: "添加成功！",
                    type: "success"
                  });
                } else {
                  this.$refs["editForm"].resetFields();
                  this.dialogFormVisibleAdd = false;
                }
              });
            })
            .catch(e => {});
        }
      });
    },
    // 全选单选多选
    selsChange(sels) {
      this.sels = sels;
    },
    // 批量删除
    batchRemove() {
      // this.idData = this.sels.map(item => item.id).toString();//转换为字符串
      var Ids = this.sels.map(item => item.Id);
      this.$confirm("确认删除选中记录吗？", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Ids: Ids
          };
          this.para.Code = this.bllCode.del;
          this.para.Data = JSON.stringify(paraId);
          handlePost(this.para).then(res => {
            this.getDataList();
            this.$message({
              message: "删除成功！",
              type: "success"
            });
          });
        })
        .catch(() => {});
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
  background: #F0F2F5;
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
    -webkit-transition: -webkit-transform .3s ease-in-out;
    transition: -webkit-transform .3s ease-in-out;
    transition: transform .3s ease-in-out;
    transition: transform .3s ease-in-out,-webkit-transform .3s ease-in-out;
    will-change: transform;
}
.allotMent {
  width: 600px !important;
}
.custom-filter-dropdown {
  padding: 8px;
  border-radius: 6px;
  background: #fff;
  box-shadow: 0 1px 6px rgba(0, 0, 0, .2);
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
        color: blue
    }
</style>
