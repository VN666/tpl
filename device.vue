<template>
  <hik-cloud-page-container class="devices-failure">
    <hik-cloud-layout>
      <resizeBox :defaultW="280" :minW="240" :maxW="624">
        <div class="left-part">
          <TreeAnsySearch ref="tree" :treeAnsyProps="treeAnsyProps" :storeListProps="storeListProps"
            :isInitSelected="isInitSelected" :checkedNode="checkedNode" @getClickData="getClickData">
          </TreeAnsySearch>
        </div>
      </resizeBox>
      <hik-cloud-page-content class="devices-failure-content" flex>
        <hik-cloud-page-search ref="search" :model="search">
          <hik-cloud-page-search-item v-if="isBigTenant" label="项目/客户" prop="projectCustom">
            <selfSelect
              remote
              clear
              filterDrop
              v-model="search.projectCustom"
              placeholder="请选择项目/客户"
              loading-text="搜索中..."
              :filterKey.sync="projectKey"
              :filter-method="filterMethods1"
              @on-scrolling-y="scrollY1">
              <el-option key="" label="全部" value="" />
              <el-option v-for="(item,index) in projectList" :key="index" :label="item.name" :value="item.val" />
            </selfSelect>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item v-if="isBigTenant" label="代理商" prop="agentId">
            <selfSelect
              remote
              clear
              filterDrop
              v-model="search.agentId"
              placeholder="请选择代理商"
              loading-text="搜索中..."
              :filterKey.sync="agentKey"
              :filter-method="filterMethods2"
              @on-scrolling-y="scrollY2"
            >
              <el-option key="" label="全部" value="" />
              <el-option v-for="(item,index) in agentList" :key="index" :label="item.agentName" :value="item.id" />
            </selfSelect>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="客户租户" prop="subTenantId" v-if="isBigTenant">
            <selfSelect
              remote
              clear
              filterDrop
              v-model="search.subTenantId"
              placeholder="全部客户租户"
              loading-text="搜索中..."
              :filterKey.sync="subTenantName"
              :filter-method="filterMethods3"
              @on-scrolling-y="scrollY3"
              @input="handleSubTenantChange"
            >
              <el-option label="全部" value=""></el-option>
              <el-option
                v-for="(item,index) in subTenantList"
                :key="item.subTenantId"
                :label="item.subTenantName"
                :value="item.subTenantId"
              />
            </selfSelect>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="客户租户项目" prop="subProjectId" v-if="isBigTenant">
            <selfSelect
              remote
              clear
              filterDrop
              v-model="search.subProjectId"
              placeholder="全部客户租户项目"
              loading-text="搜索中..."
              :filterKey.sync="subProjectName"
              :filter-method="filterMethods4"
              @on-scrolling-y="scrollY4"
            >
              <el-option label="全部" value=""></el-option>
              <el-option
                v-for="(item,index) in subProjectList"
                :key="item.projectId"
                :label="item.projectName"
                :value="item.projectId"
              />
            </selfSelect>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="设备状态" prop="devStatus">
            <el-select v-model="search.devStatus" clear placeholder="请选择设备状态">
              <el-option key="" label="全部" value="" />
              <el-option :key="0" label="离线" :value="0" />
              <el-option :key="1" label="在线" :value="1" />
            </el-select>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="设备类型" prop="devTypeList">
            <selfSelect v-model="search.devTypeList" clearable placeholder="设备类型" collapse-tags multiple multiple-nowrap showAllTag showSelectAll>
              <el-option v-for="(item,index) in devTypeList" :key="index" :label="item.name" :value="item.val" />
            </selfSelect>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item v-if='search.masterOrSub==0' label="所属主设备序列号" prop="parDevSerial">
            <inputSearch
              placeholder="请输入完整设备序列号"
              :value.sync="search.parDevSerial"
              suffixIcon=""
              :clearSeach="false"
              @search="handleSearch"
            >
            </inputSearch>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="设备序列号" prop="devSerial">
            <inputSearch
              placeholder="请输入完整设备序列号"
              :value.sync="search.devSerial"
              suffixIcon=""
              :clearSeach="false"
              @search="handleSearch"
            >
            </inputSearch>
          </hik-cloud-page-search-item>
          <!-- 待处理、处理中、待服务商处理、服务商处理中、已完成 -->
          <hik-cloud-page-search-item label="设备型号" prop="devMode">
            <el-select v-model="search.devMode" clear placeholder="请选择设备型号">
              <el-option key="" label="全部" value="" />
              <el-option v-for="(item,index) in devModelList" :key="index" :label="item" :value="item" />
            </el-Select>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="停车场" prop="parkingIot">
            <inputSearch
              placeholder="请输入..."
              :value.sync="search.parkingIot"
              suffixIcon=""
              :clearSeach="false"
              @search="handleSearch"
            >
            </inputSearch>
          </hik-cloud-page-search-item>
          <hik-cloud-page-search-item label="设备名称" prop="devName">
            <inputSearch
              placeholder="请输入..."
              :value.sync="search.devName"
              suffixIcon=""
              :clearSeach="false"
              @search="handleSearch"
            >
            </inputSearch>
          </hik-cloud-page-search-item>
          <template slot="pageSearchAction">
            <el-button type="primary" @click="handleSearch('search')">查询</el-button>
            <el-button @click="handleReset">重置</el-button>
          </template>
        </hik-cloud-page-search>
        <hik-cloud-page-action>
          <el-radio-group v-model="search.masterOrSub" @change='masterOrSubChange' type="simple">
            <el-radio-button :label="1">主设备</el-radio-button>
            <el-radio-button :label="0">子设备</el-radio-button>
          </el-radio-group>
          <div slot="rightAction">
            <el-button type="iconButton" v-if="!isBigTenant" icon="h-icon-add" @click="addDevice">添加</el-button>
            <el-button type="iconButton" v-if="!isBigTenant" icon="h-icon-details"
              @click="importDevice('')">导表添加</el-button>
            <el-button icon="h-icon-import" v-if="isBigTenant" type="iconButton"
              @click="importDevice('')">导入设备信息</el-button>
            <el-popover trigger="manual" placement="bottom-start" width="320" v-model="showExport">
              <div class="export-tips-div">
                <div class="export-tips-title">导出说明</div>
                <div class="export-tips-content"><span>导出任务已提交，请前往 </span><span class="export-tips-text"
                    @click="toExportPage">导出记录</span><span> 进行下载</span></div>
                <div class="export-tips-btns">
                  <el-button type="primary" @click="closeAlert" size="mini">确定</el-button>
                </div>
              </div>
              <el-button v-show="menuCodes.includes('IOMP_PARKING_MENU_0502')" slot="reference"
                style="margin-left: 8px;" @click="exportDevice" type="iconButton" icon="h-icon-export">导出设备</el-button>
            </el-popover>
            <el-button style="margin-left: 8px;" icon="h-icon-import" v-if="isBigTenant" type="iconButton"
              @click='importDevice("customerTenant")'>导表关联客户租户</el-button>
          </div>
        </hik-cloud-page-action>
        <hik-cloud-page-table
          class="devices-failure-table-wrapper"
          full
          :current-page.sync="search.pageNo"
          :page-size="search.pageSize"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        >
          <el-table stripe show-overflow-tooltip height="100%" :data="tableData" fixed
            :cell-class-name="setCellClassName">
            <el-table-column prop="devSerial" label="设备序列号" min-width="120" fixed :formatter="columnTextFormatter" />
            <el-table-column prop="devModel" label="设备型号" min-width="120" fixed :formatter="columnTextFormatter" />
            <el-table-column prop="devVersion" label="软件版本" min-width="120" fixed :formatter="columnTextFormatter" />
            <el-table-column prop="devStatus" label="设备状态" min-width="120" fixed :formatter="columnTextFormatter">
              <template v-slot="scope">
                <i class="icon-yuandian  yuandian__success mr4" v-if="scope.row.devStatus === 1"></i>
                <i class="icon-yuandian  yuandian__none mr4" v-else-if="scope.row.devStatus === 2"></i>
                <i class="icon-yuandian  yuandian__error mr4" v-else></i>
                <span v-if="scope.row.devStatus == 0">离线</span>
                <span v-else-if="scope.row.devStatus == 1">在线</span>
                <span v-else-if="scope.row.devStatus == 2">休眠</span>
                <span v-else>未知</span>
              </template>
            </el-table-column>
            <el-table-column prop="customName" v-if="isBigTenant" label="项目/客户" min-width="120"
              :formatter="columnTextFormatter" />
            <el-table-column prop="agentName" v-if="isBigTenant" label="代理商" min-width="120"
              :formatter="columnTextFormatter" />
            <el-table-column prop="devName" label="设备名称" min-width="120" :formatter="columnTextFormatter" />
            <el-table-column prop="devTypeName" label="设备类型" min-width="120" :formatter="columnTextFormatter" />
            <el-table-column v-if='search.masterOrSub==0' prop="parDevSerial" label="所属主设备序列号" min-width="170"
              :formatter="columnTextFormatter" />
            <el-table-column prop="subTenantName" label="客户租户" v-if="isBigTenant" min-width="170" :formatter="columnTextFormatter" />
            <el-table-column prop="subGroupName" label="客户租户项目" v-if="isBigTenant" min-width="170" :formatter="columnTextFormatter" />
            <el-table-column prop="parkingIot" label="停车场" min-width="170" :formatter="columnTextFormatter" />
            <el-table-column prop="health" label="网络健康度" min-width="120">
              <template #header>
                <span class="table-header-label">网络健康度</span>
                <el-tooltip
                  placement="top"
                  :content="search.masterOrSub === 1 ? heathTip : subHeathTip"
                  popper-class="heath-tip-popper"
                >
                  <i class="help-tip-icon h-icon-help"></i>
                </el-tooltip>
              </template>
              <template v-slot="scope">
                <span class="net-status bad" v-if="scope.row.health === '极差'">极差</span>
                <span class="net-status middle" v-else-if="scope.row.health === '中等'">中等</span>
                <span class="net-status normal" v-else-if="scope.row.health === '良好'">良好</span>
                <span class="net-status good" v-else-if="scope.row.health === '优秀'">优秀</span>
                <span class="net-status empty" v-else>--</span>
              </template>
            </el-table-column>
            <el-table-column prop="pathName" label="所属路径" min-width="120" :formatter="columnTextFormatter" />
            <el-table-column prop="groupName" :label="`所属${sceneName}`" min-width="120"
              :formatter="columnTextFormatter" />
            <el-table-column label="操作" width="260" fixed="right">
              <template v-slot="scope">
                <template>
                  <el-button type="link" @click="toDetail(scope.row)">详情</el-button>
                  <el-button type="link" v-if='scope.row.remoteConfig' @click="toHandle(scope.row,1)">远程配置</el-button>
                  <el-button type="link" v-if='search.masterOrSub == 1' @click="toHandle(scope.row,2)">变更区域</el-button>
                  <el-button type="link" v-if='search.masterOrSub == 1' @click="toHandle(scope.row,3)">刷新</el-button>
                  <el-button type="link" v-if='search.masterOrSub == 1 && !isBigTenant'
                    @click="toHandle(scope.row,4)">删除</el-button>
                </template>
              </template>
            </el-table-column>
          </el-table>
        </hik-cloud-page-table>
      </hik-cloud-page-content>
    </hik-cloud-layout>
    <transferDialog v-if="transferDialogVisible" :source="source" :visible.sync="transferDialogVisible"
      :node="currentManifastNode" :currentRow='currentRow' @success="getTable"></transferDialog>
    <uploadFile @success='refresh' :source="sourceName" :dialogVisible.sync="fileVisible"></uploadFile>
    <!-- 添加设备流程 -->
    <addDevice :visible.sync="addDeviceVisible" :node="currentNode" @success="getTable"></addDevice>
  </hik-cloud-page-container>
</template>

<script>
  import {
    orgAndUserNum, deleteNode, orgChangeOrdinal, getNodeOrUserList,
    getDeviceList, getdevTypeList, getdevModelList,
    goToRemoteConfig, deviceFlush, exportDevice, delDevice
  } from '@/api/devicesManage'
  import TreeAnsySearch from '@/components/treeAnsySearchChain'
  import treeBtnGroup from './components/treeBtnGroup'
  import addOrgDialog from './components/addOrgDialog'
  import editNameDialog from './components/editNameDialog'
  import transferDialog from './components/transferDialog'
  import tip from './components/tip'
  import resizeBox from '@/components/resizeBox'
  import uploadFile from "./modal/fileUpload.vue"
  import { mapState } from 'vuex'
  import addDevice from "./components/addDevice/addDevice.vue"
  import mixin from '../mixin'
  import { HEATH_TIP } from '../const'
  export default {
    mixins: [mixin],
    name: 'devicesManage',
    components: {
      treeBtnGroup, addOrgDialog, editNameDialog, transferDialog,
      resizeBox, uploadFile, TreeAnsySearch, addDevice
    },
    data() {
      return {
        checkedNode: null,
        devTypeList: [],
        devModelList: [],
        search: {
          agentId: '',
          groupId: '',
          subTenantId: '',
          subProjectId: '',
          devStatus: '',
          devTypeList: [],
          parDevSerial: '',
          devSerial: '',
          devMode: '',
          parkingIot: '',
          devName: '',
          projectCustom: '',
          masterOrSub: 1,
          pageNo: 1,
          pageSize: 20
        },
        subTenantName: '',
        subProjectName: '',
        heathTip: HEATH_TIP.heathTip,
        subHeathTip: HEATH_TIP.subHeathTip,
        tableData: [],
        total: 0,
        showTip: false,
        isOnlyRoot: false,
        currentManifastNode: {},
        addOrgDialogVisible: false,
        editNameDialogVisible: false,
        transferDialogVisible: false,
        //变更区域需要
        source: '',
        currentRow: {},
        //导入逻辑
        fileVisible: false,
        treeAnsyProps: {
          method: 'get',
          url: '/apiChain/v1/chain/basic/areas/actions/findAreaStoreTreeForBusiness',
          params: {}
        },
        storeListProps: {
          method: '',
          url: '/apiChain/v1/chain/basic/stores/actions/findStoresForBusinessNoPage',
          params: {}
        },
        isInitSelected: true,
        //导出逻辑
        showExport: false,
        exportTimerIndex: null,
        noBounce: true,
        //添加设备流程
        addDeviceVisible: false,
        currentNode: {},
        sourceName: ""
      }
    },
    beforeRouteEnter(to, from, next) {
      next(vm => {
        if (from.name == 'deviceDetail') {
          vm.search.masterOrSub = Number(window.sessionStorage.getItem('deviceManagemasterOrSub')) || 0
        } else {
          sessionStorage.removeItem('deviceManagemasterOrSub')
        }
      });
    },
    created() {
      const { devStatus, nodeId, nodeName, subTenantId, subTenantName, subProjectId, subProjectName } = this.$route.query
      this.search.devStatus = Number(devStatus) || ''
      this.search.subTenantId = subTenantId || ''
      this.search.subProjectId = subProjectId || ''
      this.filterMethods1('')
      this.filterMethods2('')
      this.subTenantName = subTenantName || ''
      this.filterMethods3(this.subTenantName)
      this.subProjectName = subProjectName || ''
      this.filterMethods4(this.subProjectName, this.search.subTenantId)
      if (nodeId) {
        const node = {
          nodeId,
          nodeName
        }
        this.checkedNode = node
        this.getClickData(node)
      }
      // this.getprojectList()
      // this.getagentList()
      this.getdevTypeList()
      this.getdevModelList()
      // 埋点
      this.sendCollectMessage({
        big: { lc: '20_4_0011', ety: 'enter' },
        custom: { lc: '20_5_0012', ety: 'enter' }
      })
    },
    computed: {
      ...mapState(['menuCodes']),
    },
    deactivated() {
      this.closeAlert()
    },
    beforeRouteLeave (to, from, next) {
      // 埋点
      this.getEndStamp((duration) => {
        this.sendCollectMessage({
          big: { lc: '20_4_0012', ety: 'aac', biz:{ c: duration } },
          custom: { lc: '20_5_0013', ety: 'aac', biz:{ c: duration } }
        })
      })
      next()
    },
    methods: {
      handleSubTenantChange () {
        this.search.subProjectId = ''
        this.subProjectName = ''
        this.filterMethods4('', this.search.subTenantId)
        // this.getTable()
      },
      addDevice() {
        this.addDeviceVisible = true
        this.sendClickMessage({ lc: '20_5_0017' })
      },
      importDevice(sourceName) {
        this.sourceName = sourceName
        this.fileVisible = true
        // 埋点
        if (sourceName !== 'customerTenant') {
          this.sendCollectMessage({
            big: { lc: '20_4_0016' },
            custom: { lc: '20_5_0018' }
          })
        } else {
          this.sendClickMessage({ lc: '20_4_0018' })
        }
      },
      //导出逻辑
      async exportDevice() {
        // 埋点
        this.sendCollectMessage({
          big: { lc: '20_4_0017' },
          custom: { lc: '20_5_0019' }
        })
        if (this.total === 0) {
          this.$message.error('暂无可导出数据')
          return
        }
        if (this.noBounce) {
          this.noBounce = false
          let params = {
            ...this.search,
            areaName: this.sceneName
          }
          await exportDevice(params)
          this.showExport = true
          if (this.exportTimerIndex) {
            window.clearTimeout(this.exportTimerIndex)
            this.exportTimerIndex = null
          }
          this.exportTimerIndex = setTimeout(() => {
            this.showExport = false
          }, 10000)
          setTimeout(() => {
            this.noBounce = true
          }, 1000);
        }
      },
      closeAlert() {
        this.showExport = false
      },
      toExportPage() {
        this.showExport = false
        this.$router.push({
          path: '/export-records/index'
        })
      },
      async getdevTypeList() {
        let { data } = await getdevTypeList()
        if (data && data.length > 0) {
          this.devTypeList = data
        } else {
          this.devTypeList = []
        }
      },
      async getdevModelList() {
        let { data } = await getdevModelList({ masterOrSub: this.search.masterOrSub, parDevSerial: this.parDevSerial })
        if (data && data.length > 0) {
          this.devModelList = data
        } else {
          this.devModelList = []
        }
      },
      masterOrSubChange(val) {
        this.search.devMode = ''
        this.getdevModelList()
        this.handleSearch()
        // 埋点
        const bigLcObj = {
          0: '20_4_0014',
          1: '20_4_0013'
        }
        const customLcObj = {
          0: '20_5_0015',
          1: '20_5_0014'
        }
        this.sendCollectMessage({
          big: { lc: bigLcObj[val] },
          custom: { lc: customLcObj[val] }
        })
      },
      async handleNodeDelete(data) {
        let params = { orgId: data.nodeId }
        let res = await orgAndUserNum(params)
        if (res.data.storeNum || res.data.userNum) {
          this.$msgbox({
            title: '无法删除',
            type: 'warning',
            message: `该组织下还存在${this.sceneName}或用户数据`,
            cancelButtonText: '关闭',
            showCancelButton: true,
            showConfirmButton: false
          })
        } else {
          this.$msgbox({
            title: `确定删除“${data.nodeName}"？`,
            type: 'question',
            showConfirmButton: true,
            showCancelButton: true,
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            callback: (action) => {
              if (action === 'confirm') {
                deleteNode(data.nodeId).then((response) => {
                  if (response.code === 0) {
                    this.$message({
                      message: '删除成功',
                      type: 'success'
                    })
                  }
                })
              }
            }
          })
        }
      },
      async handleNodeMove(data, status) {
        let params = { orgId: data.nodeId, status }
        let { code } = await orgChangeOrdinal(params)
        if (code === 0) {
          this.$message({
            message: '移动成功',
            type: 'success'
          })
        }
      },
      navToStoreManage() {
        if (!hasMenuAuth('RETAIL_MENU_1105')) {
          this.$message.error(`抱歉，您无【${this.sceneName}管理】权限`)
          return
        }
        this.$router.push('/system/scene')
      },
      navToUserManage() {
        if (!hasMenuAuth('RETAIL_MENU_1101')) {
          this.$message.error(`抱歉，您无【用户管理】权限`)
          return
        }
        this.$router.push('/system/user')
      },
      manifastHandler(eventName, data) {
        this.currentManifastNode = data
        switch (eventName) {
          case 'addOrg':
            this.addOrgDialogVisible = true
            break
          case 'addStore':
            if (!hasMenuAuth('RETAIL_MENU_1105')) {
              this.$message.error(`抱歉，您无【${this.sceneName}管理】权限`)
              return
            }
            this.$router.push({
              name: 'addScene',
              params: {
                isAdd: 'add',
                id: data.nodeId,
                name: data.pathName
              }
            })
            break
          case 'editName':
            this.editNameDialogVisible = true
            break
          case 'transfer':
            this.transferDialogVisible = true
            break
          case 'upMove':
            this.handleNodeMove(data, 1)
            break
          case 'downMove':
            this.handleNodeMove(data, 2)
            break
          case 'delete':
            this.handleNodeDelete(data)
            break
          default:
            break
        }
      },
      getClickData(data) {
        this.currentNode = data
        this.search.groupId = data.nodeId
        this.handleSearch()
      },
      refresh() {
        this.getdevTypeList()
        // this.getprojectList()
        // this.getagentList()
        this.$nextTick(() => {
          this.filterMethods1('')
          this.filterMethods2('')
          this.getdevModelList()
          this.getTable()
        })
      },
      toHandle(row, type) {
        this.currentRow = row
        if (type == 1) {
          goToRemoteConfig({ devSerial: row.devSerial, masterOrSub: this.search.masterOrSub }).then((res) => {
            if (res.code == 0 && res.data && res.data.url) {
              window.open(res.data.url)
            }
          })
          // 埋点
          this.sendCollectMessage({
            big: { lc: '20_4_0020' },
            custom: { lc: '20_5_0021' }
          })
        } else if (type == 2) {
          this.source = 'deviceManage'
          this.transferDialogVisible = true
          // 埋点
          this.sendCollectMessage({
            big: { lc: '20_4_0021' },
            custom: { lc: '20_5_0022' }
          })
        } else if (type == 3) {
          deviceFlush({ devSerial: row.devSerial }).then((res) => {
            if (res.code == 0) {
              this.$message.success('刷新成功！')
              this.getTable()
            }
          })
          // 埋点
          this.sendCollectMessage({
            big: { lc: '20_4_0022' },
            custom: { lc: '20_5_0023' }
          })
        } else if (type == 4) {
          this.$confirm(`确定删除该设备？`, {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'question'
          }).then(async () => {
            delDevice({ devId: row.devId }).then(res => {
              if (res.code == 0) {
                this.$message.success('删除成功！')
                this.getTable()
              }
            })
          }).catch(() => { })
          this.sendClickMessage({ lc: '20_5_0024' })
        }
      },
      toDetail(row) {
        window.sessionStorage.setItem('deviceManagemasterOrSub', this.search.masterOrSub)
        this.$router.push({
          path: `/deviceManage/deviceDetail`,
          query: {
            id: row.id,
            masterOrSub: this.search.masterOrSub,
            devSerial: row.devSerial,
            source: 'deviceManage'
          }
        })
        // 埋点
        this.sendCollectMessage({
          big: { lc: '20_4_0019' },
          custom: { lc: '20_5_0020' }
        })
      },
      handleSearch (type) {
        if (this.search.pageNo !== 1) {
          this.search.pageNo = 1
        } else {
          this.getTable()
        }
        if (type === 'search') {
          this.sendCollectMessage({
            big: { lc: '20_4_0015' },
            custom: { lc: '20_5_0016' }
          })
        }
      },
      async getTable() {
        try {
          const res = await getDeviceList(this.search)
          if (res.code === 0) {
            const { rows = [], total = 0 } = (res && res.data) || {}
            this.tableData = rows
            this.total = total
          }
        } catch (e) {
          console.log('getDeviceList Error: ', e)
        }
      },
      handleSizeChange(size) {
        this.search.pageSize = size
        this.handleSearch()
      },
      handleCurrentChange(number) {
        this.search.pageNo = number
        this.getTable()
      },
      handleReset() {
        this.search = {
          agentId: '',
          subTenantId: '',
          subProjectId: '',
          devStatus: '',
          devTypeList: [],
          parDevSerial: '',
          devSerial: '',
          devMode: '',
          parkingIot: '',
          groupId: this.search.groupId,
          devName: '',
          projectCustom: '',
          masterOrSub: this.search.masterOrSub,
          pageNo: 1,
          pageSize: 20
        }
        this.subProjectName = ''
        this.subTenantName = ''
        this.filterMethods1('')
        this.filterMethods2('')
        this.filterMethods3(this.subTenantName)
        this.filterMethods4(this.subProjectName, this.search.subTenantId)
        this.projectPageNo = 1
        this.agentPageNo = 1
        this.getTable()
      },
      // 故障对象 是否满足跳转到详情条件
      isEventNameToDetail(prop, row) {
        return prop === 'eventName' && !(+row.eventSort === 3 && +row.eventType === 3)
      },
      isTotalNumClickable(prop, row) {
        return prop === 'totalNum' && row.totalNum > 0
      },
      // 设置特殊cell的样式
      setCellClassName({ row, column }) {
        if (this.isEventNameToDetail(column.property, row) || this.isTotalNumClickable(column.property, row)) {
          return 'clickable'
        }
        return ''
      },
      // 为了列伸缩后字符省略了依然可以用组件自带的tooltip
      columnTextFormatter(row, column) {
        const prop = column.property
        const value = row[prop]
        return value || '--'
      },
    }
  }
</script>

<style lang="scss">
  .heath-tip-popper {
    white-space: pre-line;
    max-width: 240px;
  }
</style>

<style lang="scss" scoped>
  .devices-failure {
    .devices-failure-table-wrapper {
      ::v-deep .el-table__row {
        .clickable .label {
          cursor: pointer;
          color: #1E7FFF;
        }
      }
    }
    .net-status {
      display: inline-block;
      vertical-align: middle;
      height: 20px;
      line-height: 20px;
      padding: 0 8px;
      border-radius: 4px;
      font-size: 12px;
      color: #FFFFFF;
      &.empty {
        color: rgba(0, 0, 0, 0.7);
      }
      &.good {
        background: #3FCA8E;
      }
      &.normal {
        background: #1457FB;
      }
      &.middle {
        background: #FFAD19;
      }
      &.bad {
        background: #FF5E34;
      }
    }
  }




  .left-part {
    /* width: 400px; */
    height: 100%;
    padding-top: 12px;
    flex-grow: 0;
    flex-shrink: 0;
    border-right: 1px solid rgba(0, 0, 0, 0.12);
    display: flex;
    flex-direction: column;

    .org-store-tree {
      flex-grow: 1;
      flex-shrink: 1;
    }
  }
  .table-header-label {
    color: rgba(0,0,0,.7);
    vertical-align: middle;
  }
  .help-tip-icon {
    font-size: 20px;
    vertical-align: middle;
  }
</style>
