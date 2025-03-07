<template>
  <div>
    <div class="gva-search-box">
      <el-form :inline="true" :model="searchInfo" class="demo-form-inline">
        <el-form-item label="任务名称">
          <el-input v-model="searchInfo.name" placeholder="搜索条件" />
        </el-form-item>
        <el-form-item>
          <el-button icon="search" size="mini" type="primary" @click="onSubmit"
            >查询</el-button
          >
          <el-button icon="refresh" size="mini" @click="onReset"
            >重置</el-button
          >
        </el-form-item>
      </el-form>
    </div>
    <div class="gva-table-box">
      <div class="gva-btn-list">
        <el-button icon="plus" size="mini" type="primary" @click="openDialog"
          >新增</el-button
        >
        <el-popover v-model:visible="deleteVisible" placement="top" width="160">
          <p>确定要删除吗？</p>
          <div style="text-align: right; margin-top: 8px">
            <el-button size="mini" type="text" @click="deleteVisible = false"
              >取消</el-button
            >
            <el-button size="mini" type="primary" @click="onDelete"
              >确定</el-button
            >
          </div>
          <template #reference>
            <el-button
              :disabled="!multipleSelection.length"
              icon="delete"
              size="mini"
              style="margin-left: 10px"
              >删除
            </el-button>
          </template>
        </el-popover>
        <el-button icon="plus" size="mini" type="primary" @click="openDialogTag"
          >标签管理</el-button
        >
        <el-button
          icon="plus"
          size="mini"
          type="primary"
          @click="openDialogTagRun"
          >运行标签</el-button
        >
      </div>
      <el-table
        ref="multipleTable"
        :data="tableData"
        row-key="ID"
        style="width: 100%"
        tooltip-effect="dark"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column
          align="left"
          label="任务名称"
          prop="name"
          width="240"
        />
        <el-table-column
          align="left"
          label="运行环境"
          prop="api_env_name"
          width="240"
        />
        <el-table-column
          align="left"
          label="时间配置"
          prop="runTime"
          width="120"
        />
        <el-table-column align="left" label="下次执行时间" width="180">
          <template #default="scope">{{
            formatDate(scope.row.nextRunTime)
          }}</template>
        </el-table-column>
        <el-table-column
          align="left"
          label="运行次数"
          prop="runNumber"
          width="120"
        />
        <el-table-column
          align="left"
          label="备注"
          prop="describe"
          width="200"
        />
        <el-table-column
          align="left"
          label="定时执行"
          prop="status"
          width="120"
        >
          <template #default="scope">
            <el-tag :type="scope.row.status ? 'success' : 'info'">{{
              scope.row.status ? "启用" : "禁用"
            }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column align="left" label="按钮组" min-width="360">
          <template #default="scope">
            <el-button
              class="table-button"
              icon="detail"
              size="small"
              type="text"
              @click="detailTaskCaseFunc(scope.row)"
              >任务详情</el-button
            >
            <el-button
              class="table-button"
              icon="detail"
              size="small"
              type="text"
              @click="runCase(scope.row)"
              >后台运行</el-button
            >
            <el-button
              class="table-button"
              icon="edit"
              size="small"
              type="text"
              @click="updateTimerTaskFunc(scope.row)"
              >变更
            </el-button>
            <el-button
              icon="delete"
              size="mini"
              type="text"
              @click="deleteRow(scope.row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <div class="gva-pagination">
        <el-pagination
          :current-page="page"
          :page-size="pageSize"
          :page-sizes="[10, 30, 50, 100]"
          :total="total"
          layout="total, sizes, prev, pager, next, jumper"
          @current-change="handleCurrentChange"
          @size-change="handleSizeChange"
        />
      </div>
    </div>
    <el-dialog
      v-model="dialogFormVisible"
      :before-close="closeDialog"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      :title="type === 'create' ? '新增定时任务' : '编辑定时任务'"
    >
      <el-form :model="formData" label-position="right" label-width="80px">
        <el-form-item label="任务名称:">
          <el-input v-model="formData.name" clearable placeholder="请输入" />
        </el-form-item>
        <el-form-item label="时间配置:">
          <el-popover
            v-model:visible="cronVisible"
            placement="bottom-start"
            width="30"
          >
            <template #reference>
              <el-input
                v-model="formData.runTime"
                clearable
                placeholder="请选择时间"
                @click="cronFun"
              />
            </template>
            <timerTaskCron
              v-if="creatCron"
              :runTimeStr="formData.runTime"
              @closeTime="closeRunTimeCron"
              @runTime="runTimeCron"
            />
          </el-popover>
        </el-form-item>
        <el-form-item label="定时执行:">
          <el-switch
            v-model="formData.status"
            active-color="#13ce66"
            active-text="启用"
            clearable
            inactive-color="#ff4949"
            inactive-text="禁用"
          ></el-switch>
        </el-form-item>
        <el-form-item label="环境变量:">
          <el-select
            v-model="apiEnvID"
            placeholder="请选择"
            @change="envChange"
          >
            <el-option
              v-for="item in apiEnvData"
              :key="item.ID"
              :label="item.name"
              :value="item.ID"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="运行标签" prop="tagId">
          <el-cascader
            v-if="dialogFormVisible"
            v-model="tagIds"
            :clearable="false"
            :options="tagTableOption"
            :props="{
              multiple: true,
              checkStrictly: true,
              label: 'name',
              value: 'ID',
              disabled: 'disabled',
              emitPath: false,
            }"
            :show-all-levels="false"
            style="width: 100%"
          />
        </el-form-item>
        <el-form-item label="备注:">
          <el-input
            v-model="formData.describe"
            clearable
            placeholder="请输入"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button size="small" @click="closeDialog">取 消</el-button>
          <el-button size="small" type="primary" @click="enterDialog"
            >确 定</el-button
          >
        </div>
      </template>
    </el-dialog>
    <el-dialog
      v-model="tagDialog"
      :before-close="closeDialogTag"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      title="标签管理"
    >
      <timer-task-tag v-if="tagDialog"></timer-task-tag>
    </el-dialog>
    <el-dialog
      width="350px"
      v-model="tagDialogRun"
      :before-close="closeDialogTagRung"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      title="运行标签"
      center
    >
      <template #footer>
        <el-form label-position="right" label-width="80px">
          <el-form-item label="运行标签:">
            <el-select
              v-model="runTagId"
              class="m-2"
              placeholder="请选择运行标签"
              size="large"
            >
              <el-option
                v-for="item in tagTableOption"
                :key="item.ID"
                :label="item.name"
                :value="item.ID"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="环境变量:">
            <el-select
              v-model="runEnvId"
              placeholder="请选择"
              @change="envChange"
            >
              <el-option
                v-for="item in apiEnvData"
                :key="item.ID"
                :label="item.name"
                :value="item.ID"
              />
            </el-select>
          </el-form-item>
        </el-form>
        <div class="dialog-footer">
          <el-button size="small" @click="closeDialogTagRung">取 消</el-button>
          <el-popconfirm
            confirm-button-text="确认运行"
            cancel-button-text="取消运行"
            title="是否进入后台运行？"
            @confirm="runDialog"
          >
            <template #reference>
              <el-button type="primary">后台运行</el-button>
            </template>
          </el-popconfirm>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "TimerTask",
};
</script>

<script setup>
import {
  createTimerTask,
  deleteTimerTask,
  deleteTimerTaskByIds,
  updateTimerTask,
  findTimerTask,
  getTimerTaskList,
  setTaskCase,
  getTimerTaskTagList,
} from "@/api/timerTask";

// 全量引入格式化工具 请按需保留
import {
  getDictFunc,
  formatDate,
  formatBoolean,
  filterDict,
} from "@/utils/format";
import { ElMessage, ElMessageBox } from "element-plus";
import { ref, watch, nextTick } from "vue";

import timerTaskCron from "@/view/interface/timerTask/timerTaskCron.vue";
import { getApiConfigList } from "@/api/apiConfig";
import { runTimerTask } from "@/api/runTestCase";
import { useRouter } from "vue-router";
import TimerTaskTag from "@/view/interface/timerTask/timerTaskTag.vue";
import { getEnvList } from "@/api/env";

const router = useRouter();

// 自动化生成的字典（可能为空）以及字段
const formData = ref({
  name: "",
  runTime: "",
  nextRunTime: new Date(),
  status: false,
  describe: "",
  runNumber: 0,
  TestCase: [],
  config: {
    ID: 0,
  },
  api_env_id: 0,
});
const cronVisible = ref(false);
const cronFun = () => {
  cronVisible.value = true;
};
const creatCron = ref(false);
const runTimeCron = (cron) => {
  formData.value.runTime = cron;
};
const closeRunTimeCron = (isClose) => {
  cronVisible.value = isClose;
};
const configID = ref();
const configChange = (key) => {
  formData.value.config.ID = key;
};

// =========== 表格控制部分 ===========
const page = ref(1);
const total = ref(0);
const pageSize = ref(10);
const tableData = ref([]);
const searchInfo = ref({});

// 重置
const onReset = () => {
  searchInfo.value = {};
};

// 搜索
const onSubmit = () => {
  page.value = 1;
  pageSize.value = 10;
  if (searchInfo.value.status === "") {
    searchInfo.value.status = null;
  }
  getTableData();
};

// 分页
const handleSizeChange = (val) => {
  pageSize.value = val;
  getTableData();
};

// 修改页面容量
const handleCurrentChange = (val) => {
  page.value = val;
  getTableData();
};
const configData = ref([]);
const getConfigData = async () => {
  const config = await getApiConfigList({ page: 1, pageSize: 99999 });
  if (config.code === 0) {
    configData.value = config.data.list;
  }
};

// 查询
const getTableData = async () => {
  const table = await getTimerTaskList({
    page: page.value,
    pageSize: pageSize.value,
    ...searchInfo.value,
  });
  if (table.code === 0) {
    tableData.value = table.data.list;
    total.value = table.data.total;
    page.value = table.data.page;
    pageSize.value = table.data.pageSize;
  }
};

const initPage = async () => {
  await getTableData();
};
initPage();

watch(tableData, () => {
  setCaseIds();
});

const setCaseIds = () => {
  tableData.value &&
    tableData.value.forEach((testCase) => {
      const caseIds =
        testCase.TestCase &&
        testCase.TestCase.map((i) => {
          return i.ID;
        });
      testCase.caseIds = caseIds;
    });
};

const changeCase = async (row, flag) => {
  if (flag) {
    return;
  }
  await nextTick();
  const res = await setTaskCase({
    ID: row.ID,
    caseIds: row.caseIds,
  });
  if (res.code === 0) {
    ElMessage({ type: "success", message: "用例设置成功" });
  } else {
    ElMessage({ type: "success", message: "用例设置失败" });
  }
};

// ============== 表格控制部分结束 ===============

// 获取需要的字典 可能为空 按需保留
const setOptions = async () => {};

// 获取需要的字典 可能为空 按需保留
setOptions();

// 多选数据
const multipleSelection = ref([]);
// 多选
const handleSelectionChange = (val) => {
  multipleSelection.value = val;
};

// 删除行
const deleteRow = (row) => {
  ElMessageBox.confirm("确定要删除吗?", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning",
  }).then(() => {
    deleteTimerTaskFunc(row);
  });
};

// 批量删除控制标记
const deleteVisible = ref(false);
// 多选删除
const onDelete = async () => {
  const ids = [];
  if (multipleSelection.value.length === 0) {
    ElMessage({
      type: "warning",
      message: "请选择要删除的数据",
    });
    return;
  }
  multipleSelection.value &&
    multipleSelection.value.map((item) => {
      ids.push(item.ID);
    });
  const res = await deleteTimerTaskByIds({ ids });
  if (res.code === 0) {
    ElMessage({
      type: "success",
      message: "删除成功",
    });
    if (tableData.value.length === ids.length && page.value > 1) {
      page.value--;
    }
    deleteVisible.value = false;
    getTableData();
  }
};

const detailTaskCaseFunc = (row) => {
  if (row) {
    router.push({
      name: "taskCaseDetail",
      params: {
        id: row.ID,
      },
    });
  } else {
    router.push({ name: "taskCaseDetail" });
  }
};

const runCase = async (row) => {
  let data = { taskID: Number(row.ID) };
  const res = await runTimerTask(data);
  if (res.code === 0) {
    ElMessage({
      type: "success",
      message: "运行成功",
    });
  }
};

// 行为控制标记（弹窗内部需要增还是改）
const type = ref("");
const tagDialog = ref(false);
const tagDialogRun = ref(false);
const tagIds = ref([]);
const runTagId = ref("");
const runEnvId = ref("");
// 更新行
const updateTimerTaskFunc = async (row) => {
  tagIds.value = [];
  const res = await findTimerTask({ ID: row.ID });
  await getTagData();
  creatCron.value = true;
  // configID.value = configData.value.config.ID
  type.value = "update";
  if (res.code === 0) {
    formData.value = res.data.retask;
    configID.value = formData.value.config.ID;
    dialogFormVisible.value = true;
    if (
      res.data.retask.apiTimerTaskTag &&
      res.data.retask.apiTimerTaskTag.length > 0
    ) {
      res.data.retask.apiTimerTaskTag.forEach((item) => {
        tagIds.value.push(item.ID);
      });
    }
    formData.value.apiTimerTaskTag = [];
    formData.value.tagIds = tagIds.value;
    if (formData.value.api_env_id > 0) {
      apiEnvID.value = formData.value.api_env_id;
    }
  }
};

// 删除行
const deleteTimerTaskFunc = async (row) => {
  const res = await deleteTimerTask({ ID: row.ID });
  if (res.code === 0) {
    ElMessage({
      type: "success",
      message: "删除成功",
    });
    if (tableData.value.length === 1 && page.value > 1) {
      page.value--;
    }
    await getTableData();
  }
};

// 弹窗控制标记
const dialogFormVisible = ref(false);

// 打开弹窗
const openDialog = () => {
  getConfigData();
  getTagData();
  type.value = "create";
  creatCron.value = true;
  dialogFormVisible.value = true;
};

const tagTableOption = ref([]);
const tagTableData = ref([]);
const getTagData = async () => {
  tagTableOption.value = [];
  const tables = await getTimerTaskTagList();
  tagTableData.value = tables.data.list;
  tagTableData.value &&
    tagTableData.value.forEach((item) => {
      const option = {
        ID: item.ID,
        name: item.name,
      };
      tagTableOption.value.push(option);
    });
  await getApiEnv();
};

const openDialogTag = () => {
  tagDialog.value = true;
};

const openDialogTagRun = async () => {
  tagDialogRun.value = true;
  await getTagData();
};

const closeDialogTag = () => {
  tagDialog.value = false;
};

const closeDialogTagRung = () => {
  tagDialogRun.value = false;
  runTagId.value = "";
  runEnvId.value = "";
};

// 关闭弹窗
const closeDialog = () => {
  configID.value = "";
  apiEnvID.value = "";
  dialogFormVisible.value = false;
  formData.value = {
    name: "",
    runTime: "",
    nextRunTime: new Date(),
    status: false,
    describe: "",
    runNumber: 0,
    config: { ID: 0 },
    api_env_id: 0,
  };
  apiEnvID.value = "";
  creatCron.value = false;
  tagIds.value = [];
};
// 弹窗确定
const runDialog = async () => {
  if (runTagId.value === "" || runEnvId.value === "") {
    ElMessage({
      type: "error",
      message: "运行标签和环境变量不能为空，请设置后再运行",
    });
    return;
  }
  let data = { tagID: runTagId.value, env: runEnvId.value };
  const res = await runTimerTask(data);
  if (res.code === 0) {
    ElMessage({
      type: "success",
      message: "运行成功",
    });
    closeDialogTagRung();
  }
};

const enterDialog = async () => {
  if (formData.value.name === "") {
    ElMessage({
      type: "error",
      message: "任务名称不能为空",
    });
    return;
  }
  if (formData.value.status && formData.value.runTime === "") {
    ElMessage({
      type: "error",
      message: "开启定时任务时时间配置不能为空",
    });
    return;
  }
  if (formData.value.api_env_id < 1) {
    ElMessage({
      type: "error",
      message: "请选择环境变量",
    });
    return;
  }
  apiEnvData.value.forEach((item, index, arr) => {
    if (item.ID === formData.value.api_env_id) {
      formData.value.api_env_name = item.name;
    }
  });
  let res;
  formData.value.TestCase = [];
  formData.value.tagIds = tagIds;
  switch (type.value) {
    case "create":
      res = await createTimerTask(formData.value);
      break;
    case "update":
      res = await updateTimerTask(formData.value);
      break;
    default:
      res = await createTimerTask(formData.value);
      break;
  }
  if (res.code === 0) {
    ElMessage({
      type: "success",
      message: "创建/更改成功",
    });
    closeDialog();
    await getTableData();
    creatCron.value = false;
  }
};

const apiEnvData = ref([]);
const apiEnvID = ref();
const getApiEnv = async () => {
  const res = await getEnvList();
  if (res.code === 0) {
    apiEnvData.value = res.data.list;
  }
};

const envChange = (key) => {
  formData.value.api_env_id = null;
  if (key && key > 0) {
    formData.value.api_env_id = key;
  }
};
</script>

<style></style>
