<template>
  <div class="user-content">
    <div class="wrap-content-head">
      <div class="title-content"
        >{{ isDepartment ? departmentData.name : memberData.title }}.
        {{ userData.rows && userData.rows.length }}
      </div>
      <div v-if="isDepartment" class="wrap-ctrl">
        <el-button
          v-for="(item, index) in departmentOperationBtns"
          :key="index"
          type="text"
          :disabled="item.disabled"
          @click="departmentOperationBtnClick(index)"
          ><i :class="`iconfont ${item.icon}`"></i> {{ item.label }}
        </el-button>
      </div>
    </div>
    <div v-loading="onLoading" class="wrap-list">
      <div v-for="(item, index) in userData.rows" :key="index" class="wrap-list-item">
        <img class="user-avatar" :src="item.avatar" alt="" />
        <div class="user-info">
          <div class="user-name" @click="usernameClick(item)">{{ item.username }}</div>
          <div class="foot color-light">
            <div class="user-emial">{{ item.email }}</div>
            <div class="user-deportment">{{ item.department && item.department.name }}</div>
          </div>
        </div>
        <div v-if="isDepartment" class="wrap-ctrl color-light">
          <el-popconfirm v-if="item.state === 1" title="确定禁用此用户吗？" @confirm="forbiddenUser(item)">
            <BtnTooltip
              :disabled="!$checkPermission(userPermissions.doEdit)"
              slot="reference"
              icon="iconfont icon-icon-test"
              tooltipContent="禁用"
            ></BtnTooltip>
          </el-popconfirm>
          <el-popconfirm v-else title="确定启用此用户吗？" @confirm="enableUser(item)">
            <BtnTooltip
              :disabled="!$checkPermission(userPermissions.doEdit)"
              slot="reference"
              icon="iconfont icon-qiyong"
              tooltipContent="启用"
            ></BtnTooltip>
          </el-popconfirm>

          <span class="line"></span>
          <el-popconfirm title="确定移除此用户吗？" @confirm="removeUserFromDepartment(item)">
            <BtnTooltip
              :disabled="!$checkPermission(userPermissions.updateUserDepartment)"
              slot="reference"
              icon="iconfont icon-ren-jianshao"
              tooltipContent="移除"
            ></BtnTooltip>
          </el-popconfirm>
        </div>
      </div>
    </div>
    <AddMemberToDepartmentDialog
      ref="AddMemberToDepartmentDialog"
      :departmentData="departmentData"
      @addUserToDepartmentSuccess="getUserList"
    ></AddMemberToDepartmentDialog>
    <DepartmentOperation
      ref="DepartmentOperation"
      :isCreateDepartment="isCreateDepartment"
      :departmentData="departmentData"
      @doCreateDepartmentSuccess="doCreateDepartmentSuccess"
    ></DepartmentOperation>
    <UserInfoDialog ref="UserInfoDialog"></UserInfoDialog>
  </div>
</template>

<script>
  import BtnTooltip from '@/components/Btn-tooltip';
  import AddMemberToDepartmentDialog from './AddMemberToDepartmentDialog';
  import DepartmentOperation from './DepartmentOperation';
  import UserInfoDialog from '@/components/UserInfoDialog';
  import { updateUserDepartment, getList, doEdit, permissions as userPermissions } from '@/api/user';
  import { doDelete, permissions as departmentPermissions } from '@/api/departmentManagement';

  export default {
    name: 'UserContent',
    components: {
      BtnTooltip,
      AddMemberToDepartmentDialog,
      DepartmentOperation,
      UserInfoDialog,
    },
    props: {
      isDepartment: {
        type: Boolean,
        default: false,
      },
      departmentData: {
        type: Object,
        required: false,
        default: () => {
          return {};
        },
      },
      memberData: {
        type: Object,
        required: false,
        default: () => {
          return {};
        },
      },
      memberKeyword: {
        type: String,
        required: false,
        default: '',
      },
    },
    data() {
      return {
        userPermissions,
        departmentPermissions,
        userData: {},
        isCreateDepartment: false,
        onLoading: false,
        departmentOperationBtns: [
          {
            icon: 'icon-jiaren',
            label: '添加成员',
            disabled: !this.$checkPermission(userPermissions.updateUserDepartment),
          },
          {
            icon: 'icon-bianji',
            label: '编辑部门',
            disabled: !this.$checkPermission(departmentPermissions.doEdit),
          },
          {
            icon: 'icon-dustbin_icon',
            label: '删除部门',
            disabled: !this.$checkPermission(departmentPermissions.doDelete),
          },
        ],
      };
    },
    computed: {
      department_id() {
        if (this.memberData.id === 2) {
          return 0;
        }
        return this.departmentData.id;
      },
    },
    created() {
      this.getUserList();
    },
    methods: {
      async getUserList() {
        this.onLoading = true;
        const params = {
          keyword: this.memberKeyword,
          department_id: this.department_id,
          state: this.memberData.id === 3 ? 0 : null,
        };
        this.memberData.id === 1
          ? (params.date_after_created = this.$baseDayjs().subtract(30, 'day').format('YYYY-MM-DD 00:00:00'))
          : null;
        const { data } = await getList(params);
        this.onLoading = false;
        this.userData = data;
      },
      showCreateDepartmentDialog() {
        this.isCreateDepartment = true;
        this.$refs.DepartmentOperation.dialogVisible = true;
      },
      departmentOperationBtnClick(index) {
        switch (index) {
          case 0:
            this.$refs.AddMemberToDepartmentDialog.show();
            break;
          case 1:
            this.isCreateDepartment = false;
            this.$refs.DepartmentOperation.dialogVisible = true;
            break;
          case 2:
            this.$confirm('此操作将永久删除该部门，并释放所有成员, 是否继续?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning',
            }).then(() => {
              this.doDelete();
            });
            break;
          default:
            break;
        }
      },
      async removeUserFromDepartment(user) {
        const { data } = await updateUserDepartment({
          id: user.id,
          department_id: 0,
        });
        this.getUserList();
        this.$message.success('移除成功');
      },
      async forbiddenUser(user) {
        const { data } = await doEdit({
          id: user.id,
          state: 0,
        });
        this.getUserList();
        this.$message.success('禁用成功');
      },
      async enableUser(user) {
        const { data } = await doEdit({
          id: user.id,
          state: 1,
        });
        this.getUserList();
        this.$message.success('启用成功');
      },
      doCreateDepartmentSuccess() {
        this.$emit('doCreateDepartmentSuccess');
      },
      async doDelete() {
        await doDelete({ ids: [this.departmentData.id] });
        this.$emit('doDeleteDepartmentSuccess');
        this.$message.success('删除成功');
      },
      usernameClick(user) {
        this.$refs.UserInfoDialog.show(user.id);
      },
    },
  };
</script>

<style lang="scss" scoped>
  .user-content {
    .wrap-content-head {
      display: flex;
      justify-content: space-between;
      margin-bottom: 18px;
      line-height: 30px;
      .title-content {
        font-size: 18px;
        color: rgb(51, 51, 51);
      }
      .wrap-ctrl {
        ::v-deep .el-button--small {
          font-size: 14px;
        }
        .iconfont {
          font-size: 14px;
        }
      }
    }
    .wrap-list {
      .wrap-list-item {
        display: flex;
        padding: 12px 0px;
        border-top: 1px solid #e8e8e8;
        .user-avatar {
          width: 32px;
          height: 32px;
          margin-right: 15px;
          border-radius: 50%;
        }
        .user-info {
          flex: 1;
          line-height: 22px;
          .user-name {
            cursor: pointer;
            &:hover {
              color: $colorBlue;
            }
          }
          .foot {
            display: flex;
            .user-emial {
              margin-right: 10px;
            }
          }
        }
        .wrap-ctrl {
          display: flex;
          align-items: center;
          justify-content: space-around;
          width: 74px;
          ::v-deep .iconfont {
            font-size: 12px;
          }
          .line {
            width: 1px;
            height: 14px;
            background-color: #ccc;
          }
          & i {
            cursor: pointer;
          }
        }
      }
    }
  }
</style>
