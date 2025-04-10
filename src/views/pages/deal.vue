<template>
  <div class="container">
    <el-row class="head" v-if="!isMobile">
      <el-col :span="10" class="headbar">
        <el-row style="width: 100%; align-items: center">
          <el-col :span="6" class="headfont">
            <span>当前阶段 </span>
          </el-col>
          <el-col :span="15">
            <el-select
              v-model="stageValue"
              class="m-2"
              placeholder="Select"
              @change="filterMember(true)"
            >
              <el-option
                class="option"
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-col>
        </el-row>
      </el-col>
      <el-col :span="10" class="headbar">
        <el-row style="width: 100%; align-items: center">
          <el-col :span="7">
            <span>欲通过的阶段 </span>
          </el-col>
          <el-col :span="10">
            <el-select
              v-model="subStageValue"
              class="m-2"
              placeholder="Select"
              @change="filterMember(false)"
            >
              <el-option
                class="option"
                v-for="item in options_little[stageValue]"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-col>
          <el-col :span="7">
            <el-button
              style="width: 100%; font-size: 14px; margin-left: 10px"
              color="#c7242f"
              @click="multiProcessAccess()"
              :disabled="stageValue === 0"
            >
              通过选中成员
            </el-button>
          </el-col>
        </el-row>
      </el-col>
    </el-row>
    <div class="mobile-head" v-else>
      <el-col :span="24" class="headbar" style="margin-bottom: 10px">
        <el-row style="width: 100%; align-items: center">
          <el-col :span="6" class="headfont">
            <span>当前阶段:</span>
          </el-col>
          <el-col :span="18">
            <el-select
              v-model="stageValue"
              class="m-2"
              placeholder="Select"
              @change="filterMember(true)"
            >
              <el-option
                class="option"
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-col>
        </el-row>
      </el-col>
      <el-col :span="24" class="headbar">
        <el-row style="width: 100%; align-items: center">
          <el-col :span="6">
            <span>期望阶段:</span>
          </el-col>
          <el-col :span="18">
            <el-select
              v-model="subStageValue"
              class="m-2"
              placeholder="Select"
              @change="filterMember(false)"
            >
              <el-option
                class="option"
                v-for="item in options_little[stageValue]"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-col>
        </el-row>
      </el-col>
      <el-col :span="24">
        <el-button
          style="width: 100%; font-size: 16px; margin-top: 10px"
          color="#c7242f"
          @click="multiProcessAccess()"
          :disabled="stageValue === 0"
        >
          通过选中成员
        </el-button>
      </el-col>
    </div>
    <div class="Main">
      <el-table
        ref="multipleTableRef"
        :data="tableData"
        style="width: 100%; height: 100%"
        :header-cell-style="{ background: '#FFF8F9', color: '#2F2F2F' }"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" />
        <el-table-column label="姓名">
          <template #default="scope">{{ scope.row.userName }}</template>
        </el-table-column>
        <el-table-column
          property="sno"
          label="学号"
          class-name="son6"
          v-if="!isMobile"
        />
        <el-table-column property="identity" label="当前阶段">
          {{ options[stageValue].label }}
        </el-table-column>
        <el-table-column label="操作">
          <template #default="scope">
            <el-button
              style="color: #21b339; font-size: 10px"
              link
              @click="processAcsess(scope.row)"
              >通过</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </div>
    <el-config-provider :locale="zhCn" class="tail">
      <el-pagination
        small
        class="el-pagination"
        v-model:current-page="PageNum"
        v-model:page-size="pageSize"
        :style="{ margin: '20px' }"
        background
        layout="total, ->,sizes, prev, pager, next, jumper"
        :total="memberNum"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-config-provider>
    <!----------------------------------------弹窗------------------------------------------->
    <el-dialog v-model="visible" title="请填写以下成员变更时间：" class="popup">
      <div class="modal">
        <div style="display: flex" v-if="showMuti">
          <div class="font-4">批量填写：</div>
          <el-date-picker
            v-model="date"
            type="datetime"
            size="small"
            format="YYYY-MM-DD-HH:mm"
            value-format="YYYY-MM-DDTHH:mm:ss.sss"
          />
          <el-button
            color="#c7242f"
            style="width: 50px; height: 24px"
            @click="batchFilling"
            >填写</el-button
          >
        </div>
        <el-table
          style="width: 90%; margin: 20px"
          max-height="50vh"
          size="small"
          :cell-style="{ padding: '2px', fontSize: '13px' }"
          :data="selectData"
          :header-cell-style="{ background: '#FFF8F9', color: '#2F2F2F' }"
        >
          <el-table-column prop="userName" label="姓名" min-width="30%">
          </el-table-column>
          <el-table-column prop="sno" label="学号" min-width="45%">
          </el-table-column>
          <el-table-column label="状态变更时间" prop="passAt">
            <template #default="scope">
              <el-date-picker
                type="datetime"
                size="small"
                v-model="scope.row.passAt"
                format="YYYY-MM-DD-HH:mm"
                value-format="YYYY-MM-DDTHH:mm:ss.sss"
              ></el-date-picker>
            </template>
          </el-table-column>
        </el-table>

        <div class="modal-tail">
          <el-button
            color="#c7242f"
            @click="confirmUpdateBatch(true)"
            style="font-size: 14px"
            >确认更改</el-button
          >
          <div style="color: #c7242f; margin-top: 5px">
            *本操作无法回退，请确认无误后再操作
          </div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { ElMessage, ElTable } from "element-plus";
import zhCn from "element-plus/es/locale/lang/zh-cn";
import { reactive } from "vue";
import { processFilter, updateBatch } from "../../api/stateControl";
import { useIsMobileStore } from "@/stores/isMobileStore";

const isMobileStore = useIsMobileStore();
const isMobile = computed(() => isMobileStore.isMobile);

interface User {
  userName: string;
  sno: string;
  id: number;
  passAt: String;
}

interface User_R {
  userId: number;
  progressId: number;
  status: number;
  passAt: String;
}

const $route = useRoute();
const $router = useRouter();
const stageValue = ref(0);
const subStageValue = ref(0);
const input = ref("");
const visible = ref(false);
const showMuti = ref(true);
const date = ref("");
const options = [
  {
    label: "入党申请人",
    value: 0,
  },
  {
    label: "入党积极分子",
    value: 1,
  },
  {
    label: "发展对象",
    value: 2,
  },
  {
    label: "中共预备党员",
    value: 3,
  },
];
const options_little = [

  [
    {
      label: "群团组织推优",
      value: 0,
      stage: 3,
    },
    {
      label: "支委会审议并上报支部党委组织",
      value: 1,
      stage: 5,
    },
    {
      label: "入党积极分子",
      value: 2,
      stage: 6,
    },
  ],
  [
    {
      label: "确定培养联系人",
      value: 0,
      stage: 9,
    },
    {
      label: "支部考察",
      value: 1,
      stage: 10,
    },
    {
      label: "听取党员意见",
      value: 2,
      stage: 11,
    },
    {
      label: "征求群众意见",
      value: 3,
      stage: 12,
    },
    {
      label: "征求党小组、联系人意见",
      value: 4,
      stage: 13,
    },
    {
      label: "支部会讨论并上报上级党委备案",
      value: 5,
      stage: 14,
    },
    {
      label: "发展对象",
      value: 6,
      stage: 15,
    },
  ],
  [
    {
      label: "确定2名正式党员为介绍人",
      value: 0,
      stage: 16,
    },

    {
      label: "政审",
      value: 1,
      stage: 17,
    },
    {
      label: "通过发展对象党校培训",
      value: 2,
      stage: 19,
    },
    {
      label: "材料齐全",
      value: 3,
      stage: 20,
    },
    {
      label: "支部综合审查",
      value: 4,
      stage: 21,
    },
    {
      label: "上级党委预审并公示",
      value: 5,
      stage: 22,
    },
    {
      label: "党员发展大会",
      value: 6,
      stage: 24,
    },
    {
      label: "党委谈话、审批",
      value: 7,
      stage: 25,
    },
    {
      label: "报再上一级党委组织部门备案",
      value: 8,
      stage: 26,
    },
    {
      label: "中共预备党员",
      value: 9,
      stage: 27,
    },
  ],
  [
    {
      label: "入党宣誓",
      value: 0,
      stage: 28,
    },
    {
      label: "参加组织生活",
      value: 1,
      stage: 31,
    },
    {
      label: "支部按季度进行考察",
      value: 2,
      stage: 32,
    },
    {
      label: "征求党员、群众意见",
      value: 3,
      stage: 34,
    },
    {
      label: "听取党小组、介绍人意见",
      value: 4,
      stage: 35,
    },
    {
      label: "转正公示",
      value: 5,
      stage: 36,
    },
    {
      label: "转正大会",
      value: 6,
      stage: 37,
    },
    {
      label: "党委审批",
      value: 7,
      stage: 38,
    },
    {
      label: "党员书记谈话",
      value: 8,
      stage: 39,
    },
    {
      label: "中共党员",
      value: 9,
      stage: 40,
    },
  ],
];
const keyNode = [
  {
    label: "递交入党申请书",
    value: 0,
    stage: 0,
  },
  {
    label: "群团组织推优",
    value: 1,
    stage: 3,
  },
  {
    label: "入党积极分子",
    value: 4,
    stage: 6,
  },
  {
    label: "发展对象",
    value: 8,
    stage: 15,
  },
  {
    label: "政审",
    value: 2,
    stage: 17,
  },
  {
    label: "党员发展大会",
    value: 8,
    stage: 24,
  },
];
const tableData = ref(<User[]>[]);
const selectData = ref<User[]>([]);
const multipleTableRef = ref<InstanceType<typeof ElTable>>();
const multipleSelection = ref<User[]>([]);
const pageSize = ref(<number>15); //每页显示数量
let memberRawData = ref(<User[]>[]);
let memberNum = ref(0); //总长度
let PageNum = ref(1); //当前页码

const filterMember = async (init: boolean) => {
  if (init) subStageValue.value = 0; //不加这个会有bug
  let RawData: { code: number; data: []; msg: string } = await processFilter(
    options_little[stageValue.value][subStageValue.value].stage
  );
  if (RawData.code === 0) {
    memberRawData.value = RawData.data;
    memberNum.value = RawData.data.length;
    tableData.value = memberRawData.value.slice(
      (PageNum.value - 1) * pageSize.value,
      PageNum.value * pageSize.value
    );
  } else {
    ElMessage.error(RawData.msg + ":" + RawData.code);
  }
};

//多选
const multiProcessAccess = async () => {
  selectData.value = multipleSelection.value;
  showMuti.value = true;
  for (let item of keyNode) {
    if (
      options_little[stageValue.value][subStageValue.value].stage == item.stage
    ) {
      showModal(); //关键节点
      return;
    }
  }
  //普通节点
  selectData.value.forEach((member) => {
    member.passAt = "";
  });
  confirmUpdateBatch(false);
};

//个人
const processAcsess = (row: User) => {
  selectData.value = [row];
  showMuti.value = false;
  for (let item of keyNode) {
    if (
      options_little[stageValue.value][subStageValue.value].stage == item.stage
    ) {
      showModal();
      return;
    }
  }

  selectData.value.forEach((member) => {
    member.passAt = "";
  });
  confirmUpdateBatch(false);
};

const confirmUpdateBatch = async (isKeyNode: boolean) => {
  if (isKeyNode) {
    //判断时间不能为空
    for (let item of selectData.value)
      if (!item.passAt) {
        ElMessage.error("不能为空");
        return;
      }
  }
  let data: User_R[] = [];
  selectData.value.forEach((member) => {
    data.push({
      userId: member.id,
      progressId: options_little[stageValue.value][subStageValue.value].stage,
      status: 1,
      passAt: member.passAt,
    });
  });
  let res: { code: number; msg: string } = await updateBatch(data);
  if (res.code == 0) {
    ElMessage.success("修改成功");
    filterMember(false);
    closeModal();
  } else {
    ElMessage.error(res.msg + ":" + res.code);
  }
};
const handleSizeChange = () => {
  tableData.value = memberRawData.value.slice(
    (PageNum.value - 1) * pageSize.value,
    PageNum.value * pageSize.value
  );
};

const handleCurrentChange = () => {
  tableData.value = memberRawData.value.slice(
    (PageNum.value - 1) * pageSize.value,
    PageNum.value * pageSize.value
  );
};

onMounted(() => {
  filterMember(true);
});

const toggleSelection = (rows?: User[]) => {
  if (rows) {
    rows.forEach((row) => {
      // TODO: improvement typing when refactor table
      // eslint-disable-next-line @typescript-eslint/ban-ts-comment
      // @ts-expect-error
      multipleTableRef.value!.toggleRowSelection(row, undefined);
    });
  } else {
    multipleTableRef.value!.clearSelection();
  }
};

const handleSelectionChange = (val: User[]) => {
  multipleSelection.value = val;
};

const batchFilling = () => {
  multipleSelection.value.forEach((member) => {
    member.passAt = date.value;
  });
};
const showModal = () => {
  visible.value = true;
};
const closeModal = () => {
  visible.value = false;
};
</script>

<style scoped>
.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.head {
  width: 80%;
  height: 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.headbar {
  display: flex;
  align-items: center;
}
:deep(.el-select__wrapper) {
  font-size: 10px;
}
.headbar span {
  width: 160px;
  font-size: 14px;
}
.Main {
  margin-top: 20px;
  flex: 1;
  overflow: auto;
}

.el-pagination.is-background .el-pager li {
  border-radius: 50%;
}

.modal {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.modal-tail {
  width: 100%;
  height: 70px;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 10;
}
.font-1 {
  margin-left: 20px;
  font-size: 1.5em;
  font-weight: 400;
}
.font-2 {
  margin-left: 20px;
  font-size: 1em;
  font-weight: 400;
}
.font-3 {
  margin-top: 3px;
  margin-left: 20px;
  color: #c7242f;
}
.font-4 {
  margin-left: 20px;
}

.mobile-head {
  padding: 15px;
}

.el-pagination.is-background .el-pager li {
  border-radius: 50%;
}

.modal {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.modal-tail {
  width: 100%;
  height: 70px;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 10;
}

.popup {
  width: 35%;
}
@media screen and (max-width: 768px) {
  .container {
    width: 100%;
    padding: 0px 0px;
    display: flex;
    flex-direction: column;
  }
  .font {
    font-size: 3rem;
  }
  .option {
    font-size: 3rem;
  }
  .popup {
    width: 90%;
  }
}
</style>

<style>
@media screen and (max-width: 768px) {
  .el-pagination__jump {
    display: none;
  }
  .el-pagination__sizes.is-first {
    display: none;
    width: 0;
    height: 0;
    overflow: hidden;
  }
  .el-pager {
    display: none;
  }
}
</style>