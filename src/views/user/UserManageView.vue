<template>
  <div id="userManageView">
    <a-form
      :model="searchParams"
      layout="inline"
      style="justify-content: center; align-content: center; margin: 25px"
    >
      <a-form-item field="title" label="用户编号：" tooltip="请输入用户的编号">
        <a-input
          v-model="searchParams.id"
          placeholder="请输入要搜索的用户编号"
        />
      </a-form-item>
      <a-form-item field="title" label="用户名称：" tooltip="请输入用户名称">
        <a-input
          v-model="searchParams.userName"
          placeholder="请输入要搜索的用户名称"
        />
      </a-form-item>
      <a-form-item field="title" label="电话：" tooltip="请输入电话">
        <a-input v-model="searchParams.phone" placeholder="请输入搜索电话" />
      </a-form-item>
      <a-form-item field="title" label="邮箱：" tooltip="请输入邮箱">
        <a-input v-model="searchParams.email" placeholder="请输入搜索邮箱" />
      </a-form-item>
      <a-form-item>
        <a-button shape="round" status="normal" type="outline" @click="doSubmit"
          >搜 索
        </a-button>
      </a-form-item>
      <a-form-item>
        <a-button shape="round" status="normal" type="outline" @click="loadData"
          >刷 新
        </a-button>
      </a-form-item>
    </a-form>
    <a-table
      :ref="tableRef"
      :column-resizable="true"
      :columns="columns"
      :data="dataList"
      :pagination="{
        showTotal: true,
        pageSize: searchParams.pageSize,
        current: searchParams.current,
        total,
        showJumper: true,
        showPageSize: true,
      }"
      @pageSizeChange="onPageSizeChange"
      @page-change="onPageChange"
    >
      <template #userAvatar="{ record }">
        <a-avatar :size="70" shape="circle">
          <img :src="record.userAvatar" alt="userAvatar" />
        </a-avatar>
      </template>
      <template #userRole="{ record }">
        <!-- user普通用户 admin管理员 -->
        <a-tag v-if="record.userRole === 'user'" color="arcoblue"
          >普通用户
        </a-tag>
        <a-tag v-if="record.userRole === 'admin'" color="green">管理员</a-tag>
      </template>
      <template #createTime="{ record }">
        {{ moment(record.createTime).format("YYYY-MM-DD HH:mm:ss") }}
      </template>
      <template #updateTime="{ record }">
        {{ moment(record.updateTime).format("YYYY-MM-DD HH:mm:ss") }}
      </template>
      <template #userState="{ record }">
        <a-tag v-if="record.userState === '正常'" color="blue">正常</a-tag>
        <a-tag v-if="record.userState === '注销'" color="grey">注销</a-tag>
        <a-tag v-if="record.userState === '封号'" color="red">封号</a-tag>
      </template>
      <template #optional="{ record }">
        <a-space>
          <a-button
            shape="round"
            type="outline"
            @click="openModalForm(record.id)"
            >修改
          </a-button>
          <a-popconfirm
            cancelText="否"
            content="确定要删除此题目吗?"
            okText="是"
            type="error"
            @cancel="
              () => {
                console.log(`已取消`);
              }
            "
            @ok="doDelete(record)"
          >
            <a-button shape="round" status="danger" type="outline"
              >删除
            </a-button>
          </a-popconfirm>
        </a-space>
      </template>
    </a-table>
    <a-modal
      :visible="visible"
      placement="right"
      unmountOnClose
      width="30%"
      @cancel="closeModel"
      @ok="handleOk"
    >
      <div style="text-align: center">
        <a-upload
          :custom-request="uploadAvatar"
          :fileList="file ? [file] : []"
          :show-file-list="false"
          action="/"
          @change="onChange"
        >
          <template #upload-button>
            <a-avatar :size="70" shape="circle">
              <img :src="userInfo?.userAvatar" alt="头像" />
            </a-avatar>
          </template>
        </a-upload>
      </div>
      <a-form
        label-align="right"
        style="max-width: 480px; margin: 0 auto"
        title="个人信息"
      >
        <a-form-item field="名称" label="名称 :">
          <a-input v-model="userInfo.userName" placeholder="请输入用户名称" />
        </a-form-item>
        <a-form-item field="账号" label="账号 :">
          <a-input v-model="userInfo.userAccount" placeholder="请输入账号" />
        </a-form-item>
        <a-form-item field="邮箱" label="邮箱 :">
          <a-input v-model="userInfo.email" placeholder="请输入邮箱" />
        </a-form-item>
        <a-form-item field="电话" label="电话 :">
          <a-input v-model="userInfo.phone" placeholder="请输入电话号码" />
        </a-form-item>
        <a-form-item field="用户状态" label="状态 :">
          <a-select v-model="userInfo.userState" placeholder="请输入用户状态">
            <a-option value="正常">正常</a-option>
            <a-option value="注销">注销</a-option>
            <a-option value="封号">封号</a-option>
          </a-select>
        </a-form-item>
        <a-form-item field="用户角色" label="角色 :">
          <a-select v-model="userInfo.userRole" placeholder="请输入用户角色">
            <a-option value="admin">管理员</a-option>
            <a-option value="user">普通用户</a-option>
          </a-select>
        </a-form-item>
        <a-form-item field="性别" label="性别 :">
          <a-input v-model="userInfo.gender" placeholder="请输入性别" />
        </a-form-item>
        <a-form-item field="userProfile" label="简介 :">
          <a-textarea v-model="userInfo.userProfile" placeholder="请输入简介" />
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, ref, watchEffect } from "vue";
import {
  FileControllerService,
  User,
  UserControllerService,
} from "../../../backapi/index";
import message from "@arco-design/web-vue/es/message";
import moment from "moment";
import { useRouter } from "vue-router";
import { FileItem, Message } from "@arco-design/web-vue";

const router = useRouter();
const tableRef = ref();
const file = ref();

const visible = ref(false);
const userInfo = ref<User>();

const dataList = ref([]);
const total = ref(0);
const searchParams = ref({
  id: undefined,
  userName: "",
  email: "",
  phone: "",
  pageSize: 10,
  current: 1,
});

const loadData = async () => {
  const res = await UserControllerService.listUserByPageUsingPost({
    ...searchParams.value,
    sortField: "createTime",
    sortOrder: "descend",
  });
  if (res.code === 0) {
    dataList.value = res.data.records;
    total.value = res.data.total;
  } else {
    message.error("加载失败，" + res.message);
  }
};

/**
 * 监听 searchParams 变量，改变时触发页面的重新加载
 */
watchEffect(() => {
  loadData();
});

/**
 * 页面加载时，请求数据
 */
onMounted(() => {
  loadData();
});

const columns = [
  {
    title: "编号",
    dataIndex: "id",
    align: "center",
  },
  {
    title: "账号",
    dataIndex: "userAccount",
    align: "center",
  },
  {
    title: "名称",
    dataIndex: "userName",
    align: "center",
  },
  {
    title: "头像",
    slotName: "userAvatar",
    align: "center",
    width: 64,
  },
  {
    title: "简介",
    dataIndex: "userProfile",
    align: "center",
  },
  {
    title: "性别",
    dataIndex: "gender",
    align: "center",
  },
  {
    title: "电话",
    dataIndex: "phone",
    align: "center",
  },
  {
    title: "邮箱",
    dataIndex: "email",
    align: "center",
  },
  {
    title: "状态",
    slotName: "userState",
    align: "center",
  },
  {
    title: "角色",
    slotName: "userRole",
    align: "center",
  },
  {
    title: "创建时间",
    slotName: "createTime",
    align: "center",
  },
  {
    title: "更新时间",
    slotName: "updateTime",
    align: "center",
  },
  {
    title: "操作",
    slotName: "optional",
    align: "center",
  },
];
/**
 * 分页
 * @param page
 */
const onPageChange = (page: number) => {
  searchParams.value = {
    ...searchParams.value,
    current: page,
  };
};

/**
 * 分页大小
 * @param size
 */
const onPageSizeChange = (size: number) => {
  searchParams.value = {
    ...searchParams.value,
    pageSize: size,
  };
};
/**
 * 打开弹窗,获取到用户信息
 */
const openModalForm = async (userId: any) => {
  const res = await UserControllerService.getUserByIdUsingGet(userId);
  console.log("res:", res.data);
  userInfo.value = res.data;
  console.log(userInfo.value);
  visible.value = true;
};
/**
 * 删除
 * @param user
 */
const doDelete = async (user: User) => {
  const res = await UserControllerService.deleteUserUsingPost({
    id: user.id,
  });
  if (res.code === 0) {
    message.success("删除成功");
    loadData();
  } else {
    message.error("删除失败");
  }
};

/**
 * 确认搜索，重新加载数据
 */
const doSubmit = () => {
  // 这里需要重置搜索页号
  searchParams.value = {
    ...searchParams.value,
    current: 1,
  };
};

// 从表单中获取的用户头像
let userAvatarImg = userInfo.value?.userAvatar;

/**
 * 上传头像
 */
const uploadAvatar = async () => {
  const res = await FileControllerService.uploadFileUsingPost(file?.value.file);
  if (res.code === 0) {
    userAvatarImg = res.data;
    Message.success("上传成功，点击确认即可修改头像");
  } else {
    Message.error("上传失败！" + res.data);
  }
};

/**
 * 确定修改按钮
 */
const handleOk = async () => {
  const res = await UserControllerService.updateUserUsingPost({
    ...userInfo.value,
    userAvatar: userAvatarImg,
  });
  if (res.code === 0) {
    Message.success("更新成功！");
    visible.value = false;
    location.reload();
  } else {
    Message.error("更新失败！", res.msg);
  }
};
const closeModel = () => {
  visible.value = false;
};

const onChange = async (_: never, currentFile: FileItem) => {
  file.value = {
    ...currentFile,
  };
};
</script>

<style scoped>
#userManageView {
  padding: 5px;
  box-shadow: 0px 0px 10px rgba(35, 7, 7, 0.21);
  border-radius: 10px;
}
</style>
