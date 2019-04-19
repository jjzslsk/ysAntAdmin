<template>
  <section class="app-container">
    <el-card class="box-card">
      <!--工具条-->
      <el-form :inline="true" :model="filters" @submit.native.prevent>
        <!-- allotButtons:过滤好的 {{allotButtons}}
        <hr>
        buttonList全部:{{buttonList}}
        <hr>
        buttonAr拥有的：{{buttonAr}} -->
        <span v-for="index in MenuButtonsData" :key="index.Id">
        <a-button  :class="setClass(index.Classname)" style="margin-right:.3rem"  :icon="index.Icon"  @click="defaultClick(index)" type="primary" >{{index.Name}}</a-button>
        </span>
        <!-- <a-button type="primary" v-if="isShowButton.add" @click="handleAdd" :icon="buttonList[0].Icon">{{buttonList[0].label}}</a-button>
        <a-button type="primary" v-if="isShowButton.Refresh" :loading="loadingRefresh" :icon="buttonList[1].Icon" @click="Refresh">{{buttonList[1].label}}</a-button> -->
      </el-form>
      <a-table
        style="margin-top:2rem"
        defaultExpandAllRows
        :pagination="false"
        :columns="columnsTree"
        :dataSource="dataList"
      >
        <!-- <a slot="name" slot-scope="text" href="javascript:;">{{text}}</a> -->
        <!-- <span slot="customTitle"><a-icon type="smile-o" /> Name</span> -->
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
          class="aIcon"
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
          <a-badge v-if="record.Show==='√'" status="success" text="正常"/>
          <a-badge v-if="record.Show==='X'" status="error" text="隐藏"/>
        </template>

        <template slot="action" slot-scope="text, record">
          <a href="javascript:;" @click="allotButtonInfo(record.key)">分配按钮</a>
          <!-- <a-divider   type="vertical"/>
          <a  href="javascript:;" @click="onEdit(record)">{{record.Edit}}</a>
          <a-divider  type="vertical"/>
          <a  href="javascript:;" @click="onDelete(record)">{{record.Del}}</a> -->

          <span  v-for="index in MenuButtonsData" :key="index.Id">
          <a href="javascript:;" v-if="index.Classname==='edit'"  @click="onEdit(record)">{{index.Name}}</a>
          <a href="javascript:;" v-if="index.Classname==='del'"  @click="onDelete(record)">{{index.Name}}</a>
          </span>
        </template>
      </a-table>

      <!-- <a-pagination
        style="margin-top:2rem;text-align: right;"
        showSizeChanger
        showQuickJumper
        v-model="current"
        :total="total"
        :showTotal="(total, range) => ` 共${total}条记录 第 ${range[0]}/${range[1]}页` "
        @showSizeChange="onShowSizeChange"
      /> -->

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
      <a-modal
        title="分配按钮"
        :width="800"
        class="amodalButton"
        v-model="dialogFormVisibleButton"
        @ok="handleOkButton"
        @click="allotIcon"
      >

          <!-- <a-button type="primary" @click="GetYsMenuButton()">获取单个菜单按钮</a-button> -->
          <!-- <a-button type="primary" @click="GetYsMenuButtons">获取所有菜单按钮</a-button> -->
          <!-- <a-button type="primary" @click="SetButton">设置按钮</a-button> -->
          <!-- {{buttonKey}}编辑ID : -->
          {{GetYsMenuButtonInfo.Name}}：
          <!-- <span v-for="i in GetYsMenuButtonInfo" :key="i.Id"> -->
          <!-- 为"{{i.Name}}"分配按钮 -->
          <!-- </span> -->

          <!-- <hr>
          ==============
          选中： {{checkedList}}
          {{selButtons}}
          <hr>
          ==============
          <hr>
            所有按钮信息：{{GetYsMenuButtonsData}} -->
          <!-- <template>
        <a-transfer
          :dataSource="mockData"
          :filterOption="filterOption"
          :targetKeys="targetKeys"
          @change="handleChange"
          :render="item=>item.title"
        >
        </a-transfer>
          </template>-->
          <!-- ==============
          <hr>
         获取操作按钮数据，已改为多选内容： {{ButtonData}} -->
         <!-- <template v-for="item in GetYsMenuButtonsData" > -->
          <a-form-item  :labelCol="{ span: 3 }">

                <!-- <a-checkbox
                 :indeterminate="indeterminate"
                 @change="onCheckAllChange"
                  :checked="checkAll" >
                  全选
                  </a-checkbox> -->


                <a-checkbox-group
                  :options="ButtonData"
                  v-model='checkedList'
                  @change="onChangeCheckbox"
                />


          </a-form-item>
         <!-- </template> -->


        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleButton=false">取消</a-button>
          <a-button type="primary" @click="allotMenuButton">确认</a-button>
        </div>
      </a-modal>

      <!-- <a-modal class="amodalButton" title="分配按钮" v-model="dialogFormVisibleButton" @ok="handleOkButton" @click="allotMenuButton">
      <template>
        <a-transfer
          :titles="['未选按钮', '已选按钮']"
          :dataSource="mockData"
          :filterOption="filterOption"
          :targetKeys="targetKeys"
          @change="handleChange"
          :render="item=>item.title"
        >
        </a-transfer>
      </template>

      <div slot="footer" class="dialog-footer">
        <a-button @click.native="dialogFormVisibleButton=false">取消</a-button>
        <a-button type="primary" @click="allotMenuButton">确认</a-button>
      </div>
      </a-modal>-->
      <!--添加界面-->
      <a-modal :afterClose='afterCloseModal' title="添加按钮" @ok="handleOkAdd" @click="createData" v-model="dialogFormVisibleAdd">
        <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
          <!-- <a-form-item label='菜单名称' :labelCol="{ span: 5 }" :wrapperCol="{ span: 12 }">
          <a-input v-decorator="['note',{rules: [{ required: true, message: 'Please input your note!' }]} ]" />
        </a-form-item>

        <a-form-item
            label='上级菜单'
            :labelCol="{ span: 5 }"
            :wrapperCol="{ span: 12 }"
          >
            <a-select
              v-decorator="[
                'gender',
                {rules: [{ required: true, message: 'Please select your gender!' }]}
              ]"
              placeholder='Select a option and change input text above'
              @change="this.handleSelectChange"
            >
              <a-select-option value='male'>male</a-select-option>
              <a-select-option value='female'>female</a-select-option>
            </a-select>
          </a-form-item>-->
          <el-form-item label="菜单名称:" prop="Name">
            <el-input v-model="editForm.Name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="参数:" prop="Param">
            <el-input v-model="editForm.Param" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="备注:" prop="Memo">
            <el-input v-model="editForm.Memo" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="图标:" prop>
            <el-input placeholder="请输入内容" v-model="editForm.Icon" class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="allotIcon"></el-button>
            </el-input>
          </el-form-item>
          <el-form-item label="上级菜单:" prop="Pid">
            <el-select v-model="editForm.Pid" placeholder="请选择">
              <el-option
                v-for="item in ListsuperiorMenu"
                :key="item.Id"
                :label="item.Name"
                :value="item.Id"
              ></el-option>
            </el-select>
            <!-- {{editForm.Pid}} -->
          </el-form-item>
          <el-form-item label="链接地址:">
            <el-input v-model="editForm.Url" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="排序:">
            <el-input-number v-model="editForm.Sort"></el-input-number>
          </el-form-item>
          <a-row>
            <a-col :span="12">
              <el-form-item label="显示菜单:">
                <a-switch @change="aSwitch" v-model="editForm.Isvisiable"/>
              </el-form-item>
            </a-col>
            <a-col :span="12">
              <el-form-item label="是否启用:">
                <a-switch @change="aState" v-model="editForm.State"/>
              </el-form-item>
            </a-col>
          </a-row>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleAdd=false">{{button.cancel}}</a-button>
          <a-button type="primary" @click="createData">{{button.add}}</a-button>
        </div>
      </a-modal>

      <!--编辑界面-->
      <a-modal :afterClose='afterCloseModal' title="编辑按钮" @ok="handleOkEdit" @click="updateData" v-model="dialogFormVisibleEdit">
        <el-form :model="editForm" label-width="100px" :rules="editFormRules" ref="editForm">
          <el-form-item label="菜单名称:" prop="Name">
            <el-input v-model="editForm.Name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="图标:" prop>
            <el-input placeholder="请输入内容" v-model="editForm.Icon" class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="allotIcon"></el-button>
            </el-input>
          </el-form-item>
          <el-form-item label="参数:" prop="Param">
            <el-input v-model="editForm.Param" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="备注:" prop="Memo">
            <el-input v-model="editForm.Memo" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="上级菜单:" prop="Pid">
            <el-select v-model="editForm.Pid" placeholder="请选择">
              <el-option
                v-for="item in ListsuperiorMenu"
                :key="item.Id"
                :label="item.Name"
                :value="item.Id"
              ></el-option>
            </el-select>
            <!-- {{editForm.Pid}} -->
          </el-form-item>
          <el-form-item label="链接地址:">
            <el-input v-model="editForm.Url" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="排序:">
            <el-input-number v-model="editForm.Sort"></el-input-number>
          </el-form-item>
          <a-row>
            <a-col :span="12">
              <el-form-item label="显示菜单:">
                <a-switch @change="aSwitch" v-model="editForm.Isvisiable"/>
              </el-form-item>
            </a-col>
            <a-col :span="12">
              <el-form-item label="是否启用:">
                <a-switch @change="aState" v-model="editForm.State"/>
              </el-form-item>
            </a-col>
          </a-row>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <a-button @click.native="dialogFormVisibleEdit=false">{{button.cancel}}</a-button>
          <a-button type="primary" @click="updateData">{{button.modify}}</a-button>
        </div>
      </a-modal>
      <a-divider orientation="left">菜单规则</a-divider>
      <p>规则通常对应一个控制器的方法,同时左侧的菜单栏数据也从规则中体现,通常建议通过命令行进行生成规则节点</p>
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
import { paraHelper } from "@/utils/para.js"; //请求参数格式
import { handlePost, handleGet } from "@/api/apihelper.js";

// 多选
// const plainOptions = ['查询','新增', '删除','导入','修改', '批量删除','5','6']
const plainOptions = [
  { label: "查询", value: 1 },
  { label: "新增", value: 2 },
  { label: "新增", value: 3 },
  { label: "新增", value: 5 },
  { label: "删除", value: 6 }
];

// const plainOptions = ['1',2, 3,'4','修改', '批量删除','5','6']
const defaultCheckedList = [];

//表头部
const columnsTree = [
  // {
  //   title: '菜单名称',
  //   key: 'Name',
  //   dataIndex: 'Name',
  //   scopedSlots: { customRender: 'name' },
  //   width: 160
  // }
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
    title: "图标",
    key: "Icon",
    dataIndex: "Icon",
    scopedSlots: { customRender: "Icon" }
  },
  {
    title: "标记",
    key: "Code",
    dataIndex: "Code"
  },
  {
    title: "链接地址",
    key: "Url",
    dataIndex: "Url"
  },
  {
    title: "显示状态",
    dataIndex: "Show",
    key: "Show",
    scopedSlots: { customRender: "statu" }
  },
  ,
  {
    title: "排序",
    key: "Sort",
    dataIndex: "Sort"
  },
  {
    title: "操作",
    key: "action",
    dataIndex: "action",
    scopedSlots: { customRender: "action" },
    width: 200
  }
];

//表主体数据
// const dataTree = [{
//   key: 1,
//   name: '系统设置',
//   icon:'icon',
//   code:'Button',
//   url:'sys/ButtonList',
//   sort:'1',
//   edit:'编辑',
//   del:'删除',
//   show:'√',
//   children: [{
//     key: 11,
//     name: '导航菜单',
//     icon:'icon',
//     code:'Button',
//     url:'sys/ButtonList',
//     sort:'1',
//     edit:'编辑',
//     del:'删除',
//     show:'√',
//   }, {
//     key: 12,
//     name: '用户管理',
//     icon:'icon',
//     code:'Button',
//     url:'sys/ButtonList',
//     sort:'1',
//     edit:'编辑',
//     del:'删除',
//     show:'√',
//   }, {
//     key: 13,
//     name: '部门管理',
//     icon:'icon',
//     code:'Button',
//     url:'sys/ButtonList',
//     sort:'1',
//     edit:'编辑',
//     del:'删除',
//     show:'√',
//   }],
// }, {
//   key: 2,
//   name: '财务管理',
//   icon:'icon',
//   code:'Button',
//   url:'sys/ButtonList',
//   sort:'1',
//   edit:'编辑',
//   del:'删除',
//   show:'√',
//   children: [{
//     key: 22,
//     name: '奖金明细',
//     icon:'icon',
//     code:'Button',
//     url:'sys/ButtonList',
//     sort:'1',
//     edit:'编辑',
//     del:'删除',
//     show:'x',
//   }
//   ]
// }];

const dataTrees = [{
  key: 1,
   Name: "系统设置", 
   Icon: "anticon anticon-tool",
   Code: "sys",
   Url: "sys_manage",
  Sort: "0",
    Show: "√",
     Edit: "编辑",
     Del: "删除",
     children:
         [
          { key: 3, Name: "操作按钮", Icon: "anticon anticon-inbox", Code: "button", Url: "/views/button/index", chidren: null,Edit: "编辑", Del: "删除", },
          { key: 17, Name: "角色管理", Icon: "anticon anticon-desktop", Code: "rolemanagement", Url: "/views/rolemanagement/index", chidren: null,Edit: "编辑", Del: "删除", }, 
          { key: 18, Name: "用户管理", Icon: "anticon anticon-user", Code: "usermanagement", Url: "/views/usermanagement/index", chidren: null,Edit: "编辑", Del: "删除", },
          { key: 19, Name: "部门管理", Icon: "anticon anticon-solution", Code: "divisionmanage", Url: "/views/divisionmanage/index", chidren: null,Edit: "编辑", Del: "删除", },
          { key: 20, Name: "数据字典", Icon: "anticon anticon-exception", Code: "datadictionary", Url: "/views/datadictionary/index", chidren: null,Edit: "编辑", Del: "删除", }, 
          { key: 21, Name: "参数设置", Icon: "anticon anticon-save", Code: "systemsetup", Url: "/views/systemsetup/index", chidren: null,Edit: "编辑", Del: "删除", }, 
          { key: 38, Name: "操作日志", Icon: "anticon anticon-export", Code: "operationlog", Url: "/views/operationlog/index", chidren: null,Edit: "编辑", Del: "删除", }, 
          { key: 40, Name: "系统配置", Icon: "anticon anticon-setting", Code: "configure", Url: "/views/configure/index", chidren: null,Edit: "编辑", Del: "删除", },
          { key: 2, Name: "导航菜单", Icon: "anticon anticon-database", Code: "navigationmenu", Url: "/views/navigationmenu/index", chidren: null,Edit: "编辑", Del: "删除", }
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
  inject:['reload'],
  data() {
    return {
      //获取按钮
      MenuButtonsData: [],
      MenuButtons: [],
      roleButtonItem: null,

      //按钮
      ButtonData:[],
      ButtonIcons: {},
      ButtonNames: {},
      buttonList: [],
      buttonAr:[],
      allotButtons:[],
      // 多选
      checkedList: defaultCheckedList,
      // checkedList: [],

      indeterminate: true,
      checkAll: false,
      plainOptions,
      // checkedLists: [],

      searchText: "",
      //初始化搜索字段
      selectValue: "Name",
      //批量选择
      selectedRowKeys: [], // Check here to configure the default column
      selectedRows: [],
      loading: false,
      loadingRefresh: false,
      //分页
      current: 1,

      //按钮KEY
      buttonKey: "",
      //菜单按钮
      GetYsMenuButtonData: [], //获取单按钮
      GetYsMenuButtonInfo: '', //获取单按钮
      GetYsMenuButtonsData: [], //获取多按钮
      selectValue: "Name",
      //穿梭框
      dataListButton: [], //按钮列表数组

      mockData: [],
      targetKeys: [],

      //搜索
      input3: "",
      input4: "",
      input5: "",
      select: "",

      // tree列表
      // dataTree,
      dataTrees,
      columnsTree,
      rowSelectionTree,

      bllCode: {
        //接口标识，由后端提供
        getButton: "GetListYsdatabaseYsButton",
        add: "AddYsdatabaseYsMenu", //添加
        edit: "UpdateYsdatabaseYsMenu", //修改
        del: "DelYsdatabaseYsMenu", //删除
        getList: "GetListYsdatabaseYsMenu", //获取列表
        getObj: "GetYsdatabaseYsMenu", //获取对象（单个）
        getRolesList: "GetListYsdatabaseYsRole" //获取角色
      },
      tableLabel: [
        { type: "selection", width: "50" },
        { Label: "ID", prop: "Id", width: "50", type: "index" },
        { Label: "名称", prop: "Name" },
        { Label: "上级菜单", prop: "Pid", width: "80" },
        { Label: "链接地址", prop: "Url" },
        { Label: "页面标识", prop: "Param", width: "150" },
        { Label: "图标", prop: "Icon", width: "50" },
        { Label: "排序", prop: "Sort", width: "50" }
        // {Label:'是否管理后台',prop:"shifouguanlihoutai",width:'150'},
        // {Label:'是否需要登录',prop:"shifouxuyaodenglu",width:'150'},
        // {Label:'创建时间',prop:"beizhu",width:'150'},
        // {Label:'备注',prop:"beizhu",width:'150'},
      ],
      filtersName: "菜单名称",
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
        tubiao: "",
        Isvisiable: ""
      },

      filterdataListData: [],
      //查询条件
      filters: {},
      ids: [],
      //分页初始化
      total: 0,
      page: 1,
      size: 10,

      addFormVisible: false, // 添加界面是否显示
      addFormRules: {
        name: [
          {
            required: true,
            message: "请输入姓名",
            trigger: "blur"
          }
        ]
      },

      mockDatas: [
        { chosen: true, key: "0", title: "添加" },
        {
          chosen: false,
          key: "1",
          title: "编辑"
        },
        { chosen: true, key: "2", title: "删除" },
        {
          chosen: false,
          key: "3",
          title: "查询"
        },
        { chosen: true, key: "4", title: "重设" },
        {
          chosen: false,
          key: "5",
          title: "导入"
        },
        {
          chosen: false,
          key: "8",
          title: "导出"
        },
        { chosen: true, key: "6", title: "导出" },
        {
          chosen: false,
          key: "7",
          title: "批量删除"
        },
        {
          chosen: false,
          key: "9",
          title: "批量添加"
        }
      ]
    };
  },
  selectedKeys: [],
  selectedRowKeys: [], // Check here to configure the default column
  loading: false,
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0;
    }
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
    setClass(index) {
      if(index === 'edit'||index === 'del'||index === 'search' ){
      return 'p1'
      }
    },
    //过滤按钮
    allotButton() {
      this.allotButtons = []
      var ButtonDatas = [];
      this.buttonAr.Data[0].ButtonIds.map((car)=>{//拥有的按钮ID集
      const allotButton = this.buttonList.find((i)=>{//和全部的筛选对象
        return i.value === car
      })
        ButtonDatas.push(allotButton)
    })
    console.log ('ButtonDatas::',ButtonDatas)
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
    //刷新页面
    afterCloseModal (){
      this.reload();
    },
    // 图标选择
    onsomething(event) {
      this.editForm.Icon = event.toElement.className;
      this.dialogFormVisibleIcon = false;
    },
    // 多选框
    onChangeCheckbox(checkedList, checkedValues) {//单选
      this.checkedLists = checkedList
      console.log("checked = ", checkedList, checkedValues);
      this.indeterminate =
        !!checkedList.length && checkedList.length < this.ButtonData.length;
      this.checkAll = checkedList.length === this.ButtonData.length;
    },
    onCheckAllChange(e) {//全选
      // console.log (this.checkedLists)
      Object.assign(this, {
        checkedList: e.target.checked ? this.ButtonData : [],
        indeterminate: false,
        checkAll: e.target.checked
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
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
      // this.form.setFieldsValue({
      //   note: `Hi, ${value === 'male' ? 'man' : 'lady'}!`,
      // })
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
    start() {
      this.loading = true;
      // ajax request after empty completing
      setTimeout(() => {
        this.loading = false;
        this.selectedRowKeys = [];
      }, 1000);
    },
    onSelectChange(selectedRowKeys) {
      console.log("selectedRowKeys changed: ", selectedRowKeys);
      this.selectedRowKeys = selectedRowKeys;
    },
    //
    handleChangePid(value) {
      console.log(`selected ${value}`);
    },
    onDelete(data) {
      console.log(data);
      this.$confirm("确认删除该记录吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          const paraId = {
            Id: data.key
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
      // const dataSource = [...this.dataSource]
      // this.dataSource = dataSource.filter(item => item.key !== key)
    },
    //

    // 显示编辑界面
    onEdit(row) {
      // ----------
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      this.editForm = {};
      const paraId = {
        Id: row.key
      };
      this.para.Code = "GetYsdatabaseYsMenu";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.editForm = Object.assign({}, res.Data);
          // this.dataList = res.Data;
          // -------------
          //获取上级菜单
          let paert = {
            IsList: true
          };
          this.para.Data = JSON.stringify(paert);
          this.para.Code = "GetListYsdatabaseYsMenu";
          handlePost(this.para).then(res => {
            if (res.IsSuccess == true) {
              this.ListsuperiorMenu = res.Data.List;
              let top = {
                Id: 0,
                Name: "一级菜单"
              };
              this.ListsuperiorMenu.push(top);
            }
          });
        }
      });
    },
    //搜索
    handleSelectChange(value) {
      this.selectValue = value;
      // this.form.setFieldsValue({
      //   note: `Hi, ${value === 'male' ? 'man' : 'lady'}!`,
      // })
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
    //图标
    allotIcon() {
      this.dialogFormVisibleIcon = true;
    },
    handleOk() {
      this.dialogFormVisibleIcon = false;
    },
    handleOkEdit() {
      this.dialogFormVisibleEdit = false;
    },
    handleOkAdd() {
      this.dialogFormVisibleAdd = false;
    },
    handleOkButton() {
      this.dialogFormVisibleButton = false;
    },
    allotButtonInfo(data) {
      this.GetYsMenuButtonData = [];
      this.GetYsMenuButtonsData = [];
      this.buttonKey = data;
      this.dialogFormVisibleButton = true;
      this.GetYsMenuButton();//获取单个
    },
    //

    //是否显示
    aSwitch(checked) {
      this.editForm.Isvisiable = checked;
    },
    //是否启用
    aState(checked) {
      this.editForm.State = checked;
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

        //转按钮列表对象值
    carButtons() {
      var buttonkeyMap = {
        Id: "value",
        ButtonIds: "label"
      };

      for (var i = 0; i < this.GetYsMenuButtonsData.length; i++) {
        var obj = this.GetYsMenuButtonsData[i];
        for (var key in obj) {
          var newKey = buttonkeyMap[key];
          if (newKey) {
            obj[newKey] = obj[key];
            delete obj[key];
          }
        }
      }
    },

    YsAdminGetInfo() { //获取按钮
      this.para.Code = "YsAdminGetInfo";
      this.para.Data = "";
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.roleButtonItem = res.Data;
          this.MenuButtons = this.roleButtonItem.MenuButtons;
          this.MenuButtons.forEach(item => {
            if (item.MenuId == 2) {
              this.MenuButtonsData = item.Button;
            }
          });
        }
      });
    },

    // 获取列表
    getDataList() {
      //获取操作按钮
      const paras = {};
      this.para.Code = "GetListYsdatabaseYsButton";
      this.para.Data = JSON.stringify(paras);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.buttonList = res.Data.List;
          this.ButtonData = res.Data.List;
          this.carButton()//转按钮列表对象值
        }
        //-------------------------------------

        var dataSource = this.selectValue;
        const paraId = [
          {
            Page: this.page,
            Data: this.filters.data,
            Size: 10
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
            // this.total = res.Data.Count;
            this.dataList = res.Data;
            // this.getIcon()

            //获取多菜单按钮
            const paraId = {
              MenuId:2,
            };
            this.para.Code = "GetYsMenuButton";
            this.para.Data = JSON.stringify(paraId);
            handlePost(this.para).then(res => {
              if (res.IsSuccess == true) {
                this.buttonAr = res;
                this.allotButton()
                this.setClass()
                this.YsAdminGetInfo()
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
    // 查询列表
    getQueryList() {
      const paraId = {
        Id: 1
      };

      this.para.Code = "GetYsdatabaseYsMenu";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.dataList = res.Data;
        }
      });
    },

    GetYsMenuButton() {
      //编辑获取单菜单按钮
      this.checkedList = [];
      const paraId = {
        MenuId: this.buttonKey
      };
      this.para.Code = "GetYsMenuButton";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        this.GetYsMenuButtonInfo = ''
        this.GetYsMenuButtonInfo = res.Data[0]
        if (res.IsSuccess == true) {
          if (res.Data[0].ButtonIds.length > 0) {
            this.GetYsMenuButtonData = res.Data[0].ButtonIds;
            this.checkedList = this.GetYsMenuButtonData;
          }

          
        }
      });
    },
        //提交按钮
    allotMenuButton() {
      this.$confirm("确认为该菜单分配按钮吗?", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      }).then(() => {
        const paraId = [
          {
            MenuId: this.buttonKey,
            ButtonIds: this.checkedList
          }
        ];
        this.para.Code = "SetYsMenuButton";
        this.para.Data = JSON.stringify(paraId);
        handlePost(this.para).then(res => {
          if (res.IsSuccess == true) {
            this.dialogFormVisibleButton = false;
            this.getDataList()
            this.$message({
              message: "分配按钮-成功！",
              type: "success"
            });
          }
        });
      });
    },

    GetYsMenuButtons() {
      //获取多菜单按钮
      const paraId = {
        // MenuId:this.sels[0].key,
      };
      this.para.Code = "GetYsMenuButton";
      this.para.Data = JSON.stringify(paraId);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.GetYsMenuButtonsData = res.Data;
          // this.carButtons()
        }
      });
    },
    //穿梭框
    getMock() {
      const targetKeys = [];
      const mockData = [];
      console.log("1111", this.mockDatas);
      // console.log ('this.GetYsMenuButtonData.length:',this.GetYsMenuButtonData)

      for (let i = 0; i < this.mockDatas.length; i++) {
        // console.log (this.GetYsMenuButtonData[i])
        // console.log ('iiiiiii',this.mockDatas[i])
        const data = {
          key: this.mockDatas[i].key,
          title: this.mockDatas[i].title,
          chosen: this.mockDatas[i].chosen
        };
        if (data.chosen) {
          targetKeys.push(data.key);
        }
        mockData.push(data);
      }
      this.mockData = mockData;

      this.targetKeys = targetKeys;
    },
    filterOption(inputValue, option) {
      return option.description.indexOf(inputValue) > -1;
    },
    handleChange(targetKeys, direction, moveKeys) {
      console.log(targetKeys, direction, moveKeys);
      this.targetKeys = targetKeys;
    },
    // 设置按钮
    SetButton() {
      const paraId = [
        {
          MenuId: 2,
          ButtonIds: [1,38]
        },
        {
          MenuId: 17,
          ButtonIds: [1,38]          
        },
      ];
      this.para.Code = "SetYsMenuButton";
      this.para.Data = JSON.stringify(paraId);
      console.log("menuuu", this.para);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          console.log("ButtonIds", res);
          this.dialogFormVisibleButton = false;
          this.$message({
            message: "分配按钮-成功！",
            type: "success"
          });
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

    //双击编辑
    Rowdblclick(val) {
      this.currentRow = val;
      this.getDataList();
      this.dialogFormVisibleEdit = true;
      this.editForm = Object.assign({}, this.currentRow);
    },

    // 显示编辑界面
    handleEdit(index, row) {
      this.dialogStatus = "update";
      this.dialogFormVisibleEdit = true;
      // this.$refs["editForm"].resetFields(); //重置editForm
      (this.editForm = {}), (this.editForm = Object.assign({}, row));
      let paert = {
        Pid: -1
      };
      this.para.Data = JSON.stringify(paert);
      this.para.Code = this.bllCode.getList;
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

    // 显示添加界面
    handleAdd() {
      this.dialogStatus = "create";
      this.dialogFormVisibleAdd = true;
      // this.$refs["editForm"].resetFields(); //重置editForm
      this.editForm = {
        Isvisiable: true,
        State: true,
        Sort: 1
      };

      let paert = {
        IsList: true
      };
      this.para.Data = JSON.stringify(paert);
      this.para.Code = this.bllCode.getList;
      console.log(this.para);
      handlePost(this.para).then(res => {
        if (res.IsSuccess == true) {
          this.ListsuperiorMenu = res.Data.List;
          let top = {
            Id: 0,
            Name: "一级菜单"
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
      console.log(this.sels);
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
    this.getMock();
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
  font-size: 24px;
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

/* ----------------条件绑定 */
.p1 {
        display: none;
    }
    .p {
        color: blue
    }
</style>
