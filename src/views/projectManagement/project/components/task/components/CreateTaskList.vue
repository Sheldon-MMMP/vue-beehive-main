<template>
  <div class="create-task-list color-light">
    <div>
      <el-popover
        v-model="visible"
        :disabled="!isCurrentProjectMember || !this.$checkPermission(taskListPermissions.doCreate)"
        placement="bottom"
        width="200"
        trigger="click"
      >
        <div class="create-task-list-wrap-form">
          <div class="title">新建任务列表</div>
          <el-input v-model="name" placeholder="列表名称" style="width: 100%; margin: 15px 0"></el-input>
          <el-button
            type="primary"
            style="width: 100%; margin-bottom: 10px"
            :disabled="!name.length"
            @click="createClick"
            >创建
          </el-button>
        </div>
        <span slot="reference"
          ><span
            :class="[{ disabled: !isCurrentProjectMember || !this.$checkPermission(taskListPermissions.doCreate) }]"
            ><i class="el-icon-plus"></i>新建任务列表</span
          ></span
        >
      </el-popover>
    </div>
  </div>
</template>

<script>
  import { doCreate, permissions as taskListPermissions } from '@/api/taskListManagement';
  import { mapGetters, mapState } from 'vuex';

  export default {
    name: 'CreateTaskList',
    data() {
      return {
        taskListPermissions,
        visible: false,
        name: '',
      };
    },
    computed: {
      ...mapState('project', ['currentProjectId']),
      ...mapGetters('project', ['isCurrentProjectMember']),
    },
    methods: {
      async createClick() {
        await doCreate({
          name: this.name,
          project_id: this.currentProjectId,
        });
        this.visible = false;
        this.name = '';
      },
    },
  };
</script>

<style lang="scss">
  .create-task-list-wrap-form {
    .title {
      font-size: 16px;
      font-weight: 700;
      text-align: center;
      padding-bottom: 10px;
      border-bottom: 1px solid #ccc;
    }
  }
</style>
<style lang="scss" scoped>
  .create-task-list {
    div {
      width: 269px;
      padding: 0 15px 20px 0;
      font-weight: 600;
      span {
        cursor: pointer;
        user-select: none;
        i {
          margin-right: 5px;
          font-weight: 600;
        }
        &:hover {
          color: $colorBlue;
        }
      }
    }
    .disabled {
      cursor: not-allowed !important;
      &:hover {
        color: #ccc;
      }
    }
  }
</style>
