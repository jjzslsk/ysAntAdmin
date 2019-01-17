<template>
  <section class="app-container">
    <el-card class="box-card">
      <!--工具条-->
      <el-form :inline="true" :model="filters" @submit.native.prevent>
        <!-- <a-button v-if="buttons.selectshow==true" type="primary" v-on:click="getKeyList">刷新</a-button> -->
        <!-- <a-button type="primary" class="addButtonClassName" :icon="ButtonIcons.edit" @click="handleAdd">编辑</a-button> -->
        <!-- <a-button type="primary" @click="allotButton">分配按钮</a-button> -->
        <a-button type="primary" v-if="isShowButton.add" @click="handleAdd" :icon="buttonList[0].Icon">{{buttonList[0].Name}}</a-button>
        <a-button type="primary" v-if="isShowButton.Refresh" :loading="loadingRefresh" :icon="buttonList[1].Icon" @click="Refresh">{{buttonList[1].Name}}</a-button>
        <a-button type="danger" v-if="isShowButton.dels" @click="start" :icon="buttonList[4].Icon" :disabled="!hasSelected" :loading="loading">{{buttonList[4].Name}}
          <template v-if="hasSelected">{{`(${selectedRowKeys.length})`}}</template>
        </a-button>

        <el-form-item style="float: right;">
          <a-button type="primary" v-if="isShowButton.query" :icon="buttonList[5].Icon" @click="getKeyList">{{buttonList[5].Name}}</a-button>
        </el-form-item>

        <el-form-item style="float: right;">
          <a-input-group compact>
            <a-select @change="this.handleSelectChange" defaultValue="按钮名称" style="width: 40%">
              <!-- <a-select-option value='Id'>Id</a-select-option> -->
              <a-select-option value="Icon">图标</a-select-option>
              <a-select-option value="ClassName">样式</a-select-option>
              <a-select-option value="Name">按钮名称</a-select-option>
            </a-select>
            <a-input style="width: 60%" defaultValue v-model="filters.data"/>
          </a-input-group>
        </el-form-item>
      </el-form>

      <!--列表-->
      <!-- <a-table defaultExpandAllRows :pagination="false" size="small" :columns="columnsTree" :dataSource="dataTree" :rowSelection="rowSelectionTree">
          <span slot="tags" slot-scope="tags">
            <a-checkbox></a-checkbox>
          </span>
          <span slot="action" slot-scope="text, record">
            <a href="javascript:;">{{record.edit}}</a>
            <a-divider type="vertical" />
            <a href="javascript:;">{{record.del}}</a>
          </span>
      </a-table>-->
      <!-- <el-table @row-dblclick='Rowdblclick' stripe :data="dataList" highlight-current-row @selection-change="selsChange" style="width: 100%;">
            <el-table-column v-for="item in tableLabel" :key="item.Label" :label="item.Label" :prop="item.prop" :width='item.width' :type='item.type'>
            </el-table-column>
            <el-table-column label="操作" width="100" fixed="right">
              <template slot-scope="scope">
                <el-button type="text"  @click="handleEdit(scope.$index, scope.row)">{{button.edit}}</el-button>
                <el-button type="text"  @click="handleDel(scope.$index, scope.row)">{{button.del}}</el-button>
              </template>
            </el-table-column>
      </el-table>-->
      <!-- 分页 -->
      <!-- <el-col :span="24" class="toolbar">
      <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10" :total="total"
        style="float:right;">
      </el-pagination>
      </el-col>-->
      <a-table
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :pagination="false"
        :dataSource="dataList"
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
          class="aIcon"
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

        <template slot="Icon" slot-scope="text,record">
          <!-- <a-badge v-if="record.Isvisiable == true" status="success" text="正常" /> -->
          <!-- {{record}} -->
          <i :class="record.Icon"></i>
        </template>

        <template slot="statu" slot-scope="text,record">
          <a-badge v-if="record.Isvisiable == true" status="success" text="正常"/>
          <a-badge v-if="record.Isvisiable == false" status="error" text="隐藏"/>
        </template>
        <template slot="action" slot-scope="text, record">
          <a href="javascript:;" v-if="isShowButton.edit" @click="onEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" v-if="isShowButton.del"  @click="onDelete(record)">删除</a>
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

      <!--图标-->
      <a-modal title="添加图标" @ok="handleOk" @click="allotIcon" v-model="dialogFormVisibleIcon">
        <ul class="anticons-list">
          <li class>
            <i @click="onsomething" class="anticon anticon-lock"></i>
            <span class="anticon-class">
              <span class="ant-badge">lock</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-unlock"></i>
            <span class="anticon-class">
              <span class="ant-badge">unlock</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-bars"></i>
            <span class="anticon-class">
              <span class="ant-badge">bars</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-book"></i>
            <span class="anticon-class">
              <span class="ant-badge">book</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-calendar"></i>
            <span class="anticon-class">
              <span class="ant-badge">calendar</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-cloud"></i>
            <span class="anticon-class">
              <span class="ant-badge">cloud</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-cloud-download"></i>
            <span class="anticon-class">
              <span class="ant-badge">cloud-download</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-code"></i>
            <span class="anticon-class">
              <span class="ant-badge">code</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-copy"></i>
            <span class="anticon-class">
              <span class="ant-badge">copy</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-credit-card"></i>
            <span class="anticon-class">
              <span class="ant-badge">credit-card</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-delete"></i>
            <span class="anticon-class">
              <span class="ant-badge">delete</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-desktop"></i>
            <span class="anticon-class">
              <span class="ant-badge">desktop</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-download"></i>
            <span class="anticon-class">
              <span class="ant-badge">download</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-ellipsis"></i>
            <span class="anticon-class">
              <span class="ant-badge">ellipsis</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file"></i>
            <span class="anticon-class">
              <span class="ant-badge">file</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-text"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-text</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-unknown"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-unknown</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-pdf"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-pdf</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-word"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-word</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-excel"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-excel</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-jpg"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-jpg</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-ppt"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-ppt</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-markdown"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-markdown</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-file-add"></i>
            <span class="anticon-class">
              <span class="ant-badge">file-add</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-folder"></i>
            <span class="anticon-class">
              <span class="ant-badge">folder</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-folder-open"></i>
            <span class="anticon-class">
              <span class="ant-badge">folder-open</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-folder-add"></i>
            <span class="anticon-class">
              <span class="ant-badge">folder-add</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-hdd"></i>
            <span class="anticon-class">
              <span class="ant-badge">hdd</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-frown"></i>
            <span class="anticon-class">
              <span class="ant-badge">frown</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-meh"></i>
            <span class="anticon-class">
              <span class="ant-badge">meh</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-smile"></i>
            <span class="anticon-class">
              <span class="ant-badge">smile</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-inbox"></i>
            <span class="anticon-class">
              <span class="ant-badge">inbox</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-laptop"></i>
            <span class="anticon-class">
              <span class="ant-badge">laptop</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-appstore"></i>
            <span class="anticon-class">
              <span class="ant-badge">appstore</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-link"></i>
            <span class="anticon-class">
              <span class="ant-badge">link</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-mail"></i>
            <span class="anticon-class">
              <span class="ant-badge">mail</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-mobile"></i>
            <span class="anticon-class">
              <span class="ant-badge">mobile</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-notification"></i>
            <span class="anticon-class">
              <span class="ant-badge">notification</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-paper-clip"></i>
            <span class="anticon-class">
              <span class="ant-badge">paper-clip</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-picture"></i>
            <span class="anticon-class">
              <span class="ant-badge">picture</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-poweroff"></i>
            <span class="anticon-class">
              <span class="ant-badge">poweroff</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-reload"></i>
            <span class="anticon-class">
              <span class="ant-badge">reload</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-search"></i>
            <span class="anticon-class">
              <span class="ant-badge">search</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-setting"></i>
            <span class="anticon-class">
              <span class="ant-badge">setting</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-share-alt"></i>
            <span class="anticon-class">
              <span class="ant-badge">share-alt</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-shopping-cart"></i>
            <span class="anticon-class">
              <span class="ant-badge">shopping-cart</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-tablet"></i>
            <span class="anticon-class">
              <span class="ant-badge">tablet</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-tag"></i>
            <span class="anticon-class">
              <span class="ant-badge">tag</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-tags"></i>
            <span class="anticon-class">
              <span class="ant-badge">tags</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-to-top"></i>
            <span class="anticon-class">
              <span class="ant-badge">to-top</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-upload"></i>
            <span class="anticon-class">
              <span class="ant-badge">upload</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-user"></i>
            <span class="anticon-class">
              <span class="ant-badge">user</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-video-camera"></i>
            <span class="anticon-class">
              <span class="ant-badge">video-camera</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-home"></i>
            <span class="anticon-class">
              <span class="ant-badge">home</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-loading"></i>
            <span class="anticon-class">
              <span class="ant-badge">loading</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-loading-3-quarters"></i>
            <span class="anticon-class">
              <span class="ant-badge">loading-3-quarters</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-cloud-upload"></i>
            <span class="anticon-class">
              <span class="ant-badge">cloud-upload</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-star"></i>
            <span class="anticon-class">
              <span class="ant-badge">star</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-heart"></i>
            <span class="anticon-class">
              <span class="ant-badge">heart</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-environment"></i>
            <span class="anticon-class">
              <span class="ant-badge">environment</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-eye"></i>
            <span class="anticon-class">
              <span class="ant-badge">eye</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-camera"></i>
            <span class="anticon-class">
              <span class="ant-badge">camera</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-save"></i>
            <span class="anticon-class">
              <span class="ant-badge">save</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-team"></i>
            <span class="anticon-class">
              <span class="ant-badge">team</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-solution"></i>
            <span class="anticon-class">
              <span class="ant-badge">solution</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-phone"></i>
            <span class="anticon-class">
              <span class="ant-badge">phone</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-filter"></i>
            <span class="anticon-class">
              <span class="ant-badge">filter</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-exception"></i>
            <span class="anticon-class">
              <span class="ant-badge">exception</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-export"></i>
            <span class="anticon-class">
              <span class="ant-badge">export</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-customer-service"></i>
            <span class="anticon-class">
              <span class="ant-badge">customer-service</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-qrcode"></i>
            <span class="anticon-class">
              <span class="ant-badge">qrcode</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-scan"></i>
            <span class="anticon-class">
              <span class="ant-badge">scan</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-like"></i>
            <span class="anticon-class">
              <span class="ant-badge">like</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-dislike"></i>
            <span class="anticon-class">
              <span class="ant-badge">dislike</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-message"></i>
            <span class="anticon-class">
              <span class="ant-badge">message</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-pay-circle"></i>
            <span class="anticon-class">
              <span class="ant-badge">pay-circle</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-calculator"></i>
            <span class="anticon-class">
              <span class="ant-badge">calculator</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-pushpin"></i>
            <span class="anticon-class">
              <span class="ant-badge">pushpin</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-bulb"></i>
            <span class="anticon-class">
              <span class="ant-badge">bulb</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-select"></i>
            <span class="anticon-class">
              <span class="ant-badge">select</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-switcher"></i>
            <span class="anticon-class">
              <span class="ant-badge">switcher</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-rocket"></i>
            <span class="anticon-class">
              <span class="ant-badge">rocket</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-bell"></i>
            <span class="anticon-class">
              <span class="ant-badge">bell</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-disconnect"></i>
            <span class="anticon-class">
              <span class="ant-badge">disconnect</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-database"></i>
            <span class="anticon-class">
              <span class="ant-badge">database</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-compass"></i>
            <span class="anticon-class">
              <span class="ant-badge">compass</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-barcode"></i>
            <span class="anticon-class">
              <span class="ant-badge">barcode</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-hourglass"></i>
            <span class="anticon-class">
              <span class="ant-badge">hourglass</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-key"></i>
            <span class="anticon-class">
              <span class="ant-badge">key</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-flag"></i>
            <span class="anticon-class">
              <span class="ant-badge">flag</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-layout"></i>
            <span class="anticon-class">
              <span class="ant-badge">layout</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-printer"></i>
            <span class="anticon-class">
              <span class="ant-badge">printer</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-sound"></i>
            <span class="anticon-class">
              <span class="ant-badge">sound</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-usb"></i>
            <span class="anticon-class">
              <span class="ant-badge">usb</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-skin"></i>
            <span class="anticon-class">
              <span class="ant-badge">skin</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-tool"></i>
            <span class="anticon-class">
              <span class="ant-badge">tool</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-sync"></i>
            <span class="anticon-class">
              <span class="ant-badge">sync</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-wifi"></i>
            <span class="anticon-class">
              <span class="ant-badge">wifi</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-car"></i>
            <span class="anticon-class">
              <span class="ant-badge">car</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-schedule"></i>
            <span class="anticon-class">
              <span class="ant-badge">schedule</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-user-add"></i>
            <span class="anticon-class">
              <span class="ant-badge">user-add</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-user-delete"></i>
            <span class="anticon-class">
              <span class="ant-badge">user-delete</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-usergroup-add"></i>
            <span class="anticon-class">
              <span class="ant-badge">usergroup-add</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-usergroup-delete"></i>
            <span class="anticon-class">
              <span class="ant-badge">usergroup-delete</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-man"></i>
            <span class="anticon-class">
              <span class="ant-badge">man</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-woman"></i>
            <span class="anticon-class">
              <span class="ant-badge">woman</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-shop"></i>
            <span class="anticon-class">
              <span class="ant-badge">shop</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-gift"></i>
            <span class="anticon-class">
              <span class="ant-badge">gift</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-idcard"></i>
            <span class="anticon-class">
              <span class="ant-badge">idcard</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-medicine-box"></i>
            <span class="anticon-class">
              <span class="ant-badge">medicine-box</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-red-envelope"></i>
            <span class="anticon-class">
              <span class="ant-badge">red-envelope</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-coffee"></i>
            <span class="anticon-class">
              <span class="ant-badge">coffee</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-copyright"></i>
            <span class="anticon-class">
              <span class="ant-badge">copyright</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-trademark"></i>
            <span class="anticon-class">
              <span class="ant-badge">trademark</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-safety"></i>
            <span class="anticon-class">
              <span class="ant-badge">safety</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-wallet"></i>
            <span class="anticon-class">
              <span class="ant-badge">wallet</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-bank"></i>
            <span class="anticon-class">
              <span class="ant-badge">bank</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-trophy"></i>
            <span class="anticon-class">
              <span class="ant-badge">trophy</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-contacts"></i>
            <span class="anticon-class">
              <span class="ant-badge">contacts</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-global"></i>
            <span class="anticon-class">
              <span class="ant-badge">global</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-shake"></i>
            <span class="anticon-class">
              <span class="ant-badge">shake</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-api"></i>
            <span class="anticon-class">
              <span class="ant-badge">api</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-fork"></i>
            <span class="anticon-class">
              <span class="ant-badge">fork</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-dashboard"></i>
            <span class="anticon-class">
              <span class="ant-badge">dashboard</span>
            </span>
          </li>
          <li class>
            <i @click="onsomething" class="anticon anticon-table"></i>
            <span class="anticon-class">
              <span class="ant-badge">table</span>
            </span>
          </li>
        </ul>

        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleIcon=false">取消</a-button>
          <a-button type="primary" @click.native="dialogFormVisibleIcon=false">确认</a-button>
        </div>
      </a-modal>

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
            :data="data"
          >
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
      <a-modal title="添加按钮" @ok="handleOkAdd" @click="createData" v-model="dialogFormVisibleAdd">
        <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
          <el-form-item label="按钮名称:" prop="Name">
            <el-input v-model="editForm.Name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="样式:" prop="ClassName">
            <el-input v-model="editForm.ClassName" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="图标:" prop>
            <el-input placeholder="请输入内容" v-model="editForm.Icon" class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="allotIcon"></el-button>
            </el-input>
          </el-form-item>
          <el-form-item label="是否显示:">
            <a-switch @change="aSwitch" v-model="editForm.Isvisiable"/>
          </el-form-item>
          <!-- <el-form-item label="排序:">
          <el-input-number v-model="editForm.Sort"></el-input-number>
          </el-form-item>-->
        </el-form>
        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleAdd=false">{{button.cancel}}</a-button>
          <a-button type="primary" @click="createData">{{button.add}}</a-button>
        </div>
      </a-modal>

      <!--编辑界面-->
      <a-modal title="编辑按钮" @ok="handleOkEdit" @click="updateData" v-model="dialogFormVisibleEdit">
        <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
          <el-form-item label="按钮名称:" prop="Name">
            <el-input v-model="editForm.Name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="样式:" prop="Classname">
            <el-input v-model="editForm.Classname" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="图标:" prop>
            <el-input placeholder="请输入内容" v-model="editForm.Icon" class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="allotIcon"></el-button>
            </el-input>
          </el-form-item>
          <el-form-item label="是否显示:">
            <a-switch @change="aSwitch" v-model="editForm.Isvisiable"/>
          </el-form-item>
          <!-- <el-form-item label="排序:">
          <el-input-number v-model="editForm.Sort"></el-input-number>
          </el-form-item>-->
        </el-form>
        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleEdit=false">{{button.cancel}}</a-button>
          <a-button type="primary" @click="updateData">{{button.modify}}</a-button>
        </div>
      </a-modal>
      <a-divider orientation="left">按钮规则</a-divider>
      <p>添加按钮组，对每个按钮进行标识</p>
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

//列表
const dataButton = [
  {
    key: "1",
    name: "John Brown",
    age: 32,
    address: "New York No. 1 Lake Park"
  },
  {
    key: "2",
    name: "Joe Black",
    age: 42,
    address: "London No. 1 Lake Park"
  },
  {
    key: "3",
    name: "Jim Green",
    age: 32,
    address: "Sidney No. 1 Lake Park"
  },
  {
    key: "4",
    name: "Jim Red",
    age: 32,
    address: "London No. 2 Lake Park"
  }
];

const columnsTree = [
  ,
  //   {
  //   dataIndex: 'name',
  //   key: 'name',
  //   slots: { title: 'customTitle' },
  // }
  {
    title: "菜单名称",
    key: "name",
    dataIndex: "name"
  },
  {
    title: "图标",
    key: "icon",
    dataIndex: "icon"
  },
  {
    title: "标记",
    key: "code",
    dataIndex: "code"
  },
  {
    title: "链接地址",
    key: "url",
    dataIndex: "url"
  },
  {
    title: "是否显示",
    key: "show",
    dataIndex: "show",
    scopedSlots: { customRender: "tags" }
  },
  {
    title: "排序",
    key: "sort",
    dataIndex: "sort"
  },
  {
    title: "操作",
    key: "action",
    dataIndex: "action",
    scopedSlots: { customRender: "action" }
  }
];

const dataTree = [
  {
    key: 1,
    name: "系统设置",
    icon: "icon",
    code: "Button",
    url: "sys/ButtonList",
    sort: "1",
    age: 60,
    edit: "编辑",
    del: "删除",
    children: [
      {
        key: 11,
        name: "导航菜单",
        icon: "icon",
        code: "Button",
        url: "sys/ButtonList",
        sort: "1",
        age: 42,
        edit: "编辑",
        del: "删除"
      },
      {
        key: 12,
        name: "用户管理",
        icon: "icon",
        code: "Button",
        url: "sys/ButtonList",
        sort: "1",
        age: 30,
        edit: "编辑",
        del: "删除",
        tags: ["nice", "developer", "111"]
      },
      {
        key: 13,
        name: "部门管理",
        icon: "icon",
        code: "Button",
        url: "sys/ButtonList",
        sort: "1",
        age: 72,
        edit: "编辑",
        del: "删除"
      }
    ]
  },
  {
    key: 2,
    name: "财务管理",
    icon: "icon",
    code: "Button",
    url: "sys/ButtonList",
    sort: "1",
    age: 32,
    edit: "编辑",
    del: "删除",
    children: [
      {
        key: 22,
        name: "奖金明细",
        icon: "icon",
        code: "Button",
        url: "sys/ButtonList",
        sort: "1",
        age: 42,
        edit: "编辑",
        del: "删除"
      }
    ]
  }
];

const rowSelectionTree = {
  onChange: (selectedRowKeys, selectedRows) => {
    console.log(
      `selectedRowKeys: ${selectedRowKeys}`,
      "selectedRows: ",
      selectedRows
    );
  },
  onSelect: (record, selected, selectedRows) => {
    console.log(record, selected, selectedRows);
  },
  onSelectAll: (selected, selectedRows, changeRows) => {
    console.log(selected, selectedRows, changeRows);
  }
};

export default {
  watch: {
    pageSize(val) {
      console.log("pageSize", val);
    },
    current(val) {
      console.log("current", val);
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
          label: `备选项 ${i}`,
          disabled: i % 4 === 0
        });
      }
      return data;
    };

    return {
      //按钮显示隐藏
      isShowButton:{},
      //按钮
      ButtonIcons: {},
      ButtonNames: {},
      buttonList: [],

      buttonClassName: "",
      //批量选择
      selectedRowKeys: [], // Check here to configure the default column
      selectedRows: [],
      loading: false,
      loadingRefresh: false,
      //分页
      current: 1,
      //列表
      dataButton,
      searchText: "",
      columns: [
        {
          title: "按钮名称",
          dataIndex: "Name",
          key: "Name",
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
          title: "图标",
          dataIndex: "Icon",
          key: "Icon",
          scopedSlots: { customRender: "Icon" }
        },
        {
          title: "样式",
          dataIndex: "Classname",
          key: "Classname"
        },
        {
          title: "显示状态",
          dataIndex: "Isvisiable",
          key: "Isvisiable",
          scopedSlots: { customRender: "statu" }
        },
        {
          title: "操作",
          Key: "action",
          dataIndex: "action",
          scopedSlots: { customRender: "action" },
          width: 200
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
      //穿梭框
      data: generateData(),
      value3: [1],
      value4: [1],
      renderFunc(h, option) {
        return (
          <span>
            {option.key} - {option.label}
          </span>
        );
      },
      //搜索
      input3: "",
      input4: "",
      input5: "",
      select: "",

      // tree列表
      dataTree,
      columnsTree,
      rowSelectionTree,

      bllCode: {
        //接口标识，由后端提供
        add: "AddYsdatabaseYsButton", //添加
        edit: "UpdateYsdatabaseYsButton", //修改
        del: "DelYsdatabaseYsButton", //删除
        getList: "GetListYsdatabaseYsButton", //获取列表
        getObj: "GetYsdatabaseYsButton", //获取对象（单个）
        getRolesList: "GetListYsdatabaseYsRole" //获取角色
      },
      tableLabel: [
        { type: "selection", width: "50" },
        { Label: "ID", prop: "Id", width: "50", type: "index" },
        { Label: "图标", prop: "Icon", width: "100" },
        { Label: "按钮名称", prop: "Name", width: "200" },
        // { Label: "权限标识", prop: "Url"},
        // { Label: "排序", prop: "Sort", width: "50" },
        { Label: "说明", prop: "beizhu" }
        // {Label:'是否管理后台',prop:"shifouguanlihoutai",width:'150'},
        // {Label:'是否需要登录',prop:"shifouxuyaodenglu",width:'150'},
        // {Label:'创建时间',prop:"beizhu",width:'150'},
        // {Label:'备注',prop:"beizhu",width:'150'},
      ],
      filtersName: "按钮名称",
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
      dialogFormVisibleButton: false,
      dialogFormVisibleIcon: false,
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      filters: {},
      ListsuperiorMenu: [],
      dataList: [], //主页数据
      //分页初始化
      total: 0,
      page: 1,
      size: 10,

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
        Url: "",
        id: "",
        shangjiid: "",
        lianjie: "",
        shifouxianshi: "",
        paixu: "",
        tubiao: ""
      },
      //初始化搜索字段
      selectValue: "Name",
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
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0;
    }
  },
  methods: {
    // 图标选择
    onsomething(event) {
      console.log(event);
      // this.editForm.Icon = event.target.parentElement.className
      this.editForm.Icon = event.target.className;
      // console.log (this.editForm.Icon)
      this.dialogFormVisibleIcon = false;
    },
    // getIconDefaultevent(event){
    //    this.buttonClassName = event.srcElement.classList[0]
    //    if (this.buttonClassName == 'addButtonClassName'){
    //      this.dataList.map ((car)=>{
    //      if(car.Name == '添加'){
    //      this.ButtonIcons.add = car.Icon
    //      };
    //    })
    //    }
    // },

    //  linIcon(){

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
    //是否显示
    aSwitch(checked) {
      this.editForm.Isvisiable = checked;
    },
    // 显示编辑界面
    onEdit(row) {
      // ----------
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id
      };
      this.para.Code = "GetYsdatabaseYsButton";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);
        } else {
          this.$message({
            message: res.Code + ":" + res.Message,
            type: "warning"
          });
        }
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
          this.para.Code = "DelYsdatabaseYsButton";
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
    //窗口事件
    handleOk() {
      this.dialogFormVisibleIcon = false;
    },
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    handleOkAdd() {
      this.dialogFormVisibleAdd = false;
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
    Rowdblclick(row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id
      };
      this.para.Code = "GetYsdatabaseYsButton";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);
        } else {
          this.$message({
            message: res.Code + ":" + res.Message,
            type: "warning"
          });
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
    //分页操作
    onShowSizeChange(current, pageSize) {
      console.log("111", current, pageSize);
      // this.page = val;
      this.page = current;
      this.size = pageSize;
      this.getDataList();
    },
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
      // this.form.setFieldsValue({
      //   note: `Hi, ${value === 'male' ? 'man' : 'lady'}!`,
      // })
    },
    // 获取列表
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

        this.para.Code = this.bllCode.getList;
        this.para.Data = JSON.stringify(paraId[0]);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.total = res.Data.Count;
            this.dataList = res.Data.List;

            //初始化按钮
            this.isShowButton = {
              add:false,
              Refresh:false,
              edit:false,
              del:false,
              dels:false,
              query:false,  
            };
            //获取多菜单按钮
            const paraId = {
              MenuId:3,
            };
            this.para.Code = "GetYsMenuButton";
            this.para.Data = JSON.stringify(paraId);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                const buttonAr = res.Data[0].ButtonIds;
                buttonAr.forEach((i)=>{
                  switch(i){
                    case 1:
                    this.isShowButton.add = true;
                    break;                   
                    case 38:
                    this.isShowButton.Refresh = true;
                    break;                   
                    case 39:
                    this.isShowButton.edit = true;
                    break;                   
                    case 40:
                    this.isShowButton.del = true;
                    break;                   
                    case 43:
                    this.isShowButton.dels = true;
                    break;                   
                    case 45:
                    this.isShowButton.query = true;
                    break;                   
                  }
                  // if(typeof i == 'null'){
                })
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
    // 显示编辑界面
    handleEdit(index, row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row.Id
      };
      this.para.Code = "GetYsdatabaseYsButton";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);
        } else {
          this.$message({
            message: res.Code + ":" + res.Message,
            type: "warning"
          });
        }
      });
    },

    // 显示添加界面
    handleAdd() {
      this.dialogStatus = "create";
      this.dialogFormVisibleAdd = true;
      // this.$refs["editForm"].resetFields(); //重置editForm
      this.editForm = {
        Isvisiable: true
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
        } else {
          this.$message({
            message: res.Code + ":" + res.Message,
            type: "warning"
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
                  this.$message({
                    message: res.Code + ":" + res.Message,
                    type: "warning"
                  });
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

    // this.getIconDefaultevent();
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
  font-size: 26px;
  /* margin: 12px 0 8px; */
  -webkit-transition: -webkit-transform 0.3s ease-in-out;
  transition: -webkit-transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out, -webkit-transform 0.3s ease-in-out;
  will-change: transform;
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
.anticons-list {
  margin: 0;
  list-style: none;
  overflow: hidden;
}
.anticons-list li {
  float: left;
  width: 16.66%;
  /* width: 8.66%; */
  text-align: center;
  list-style: none;
  cursor: pointer;
  height: 100px;
  color: #555;
  transition: color 0.3s ease-in-out, background-color 0.3s ease-in-out;
  position: relative;
  margin: 0;
  border-radius: 4px;
  background-color: #fff;
  overflow: hidden;
  padding: 0;
}
.anticons-list li .anticon {
  font-size: 24px;
  margin: 12px 0 8px;
  transition: -webkit-transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out;
  transition: transform 0.3s ease-in-out, -webkit-transform 0.3s ease-in-out;
  will-change: transform;
}
.anticon {
  display: inline-block;
  font-style: normal;
  vertical-align: -0.125em;
  text-align: center;
  text-transform: none;
  line-height: 0;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.anticons-list li .anticon-class {
  display: block;
  text-align: center;
  -webkit-transform: scale(0.83);
  transform: scale(0.83);
  font-family: Lucida Console, Consolas;
  white-space: nowrap;
}
.anticons-list li .anticon-class .ant-badge {
  transition: color 0.3s ease-in-out;
}
.ant-badge {
  font-family: Chinese Quote, -apple-system, BlinkMacSystemFont, Segoe UI,
    PingFang SC, Hiragino Sans GB, Microsoft YaHei, Helvetica Neue, Helvetica,
    Arial, sans-serif;
  font-size: 14px;
  font-variant: tabular-nums;
  line-height: 1.5;
  color: rgba(0, 0, 0, 0.65);
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  list-style: none;
  position: relative;
  display: inline-block;
  line-height: 1;
  color: unset;
}
.anticons-list .anticon:before {
  /* content: none !important; */
}
.aIcon {
  top: -0.6rem;
}
</style>
