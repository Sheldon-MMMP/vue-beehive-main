<template>
  <div class="userManagement-container wrap-content-main">
    <vab-query-form>
      <vab-query-form-left-panel :span="12">
        <el-button
          :disabled="!this.$checkPermission(userPermissions.doDelete)"
          icon="el-icon-delete"
          type="danger"
          @click="handleDelete"
        >
          批量删除
        </el-button>
      </vab-query-form-left-panel>
      <vab-query-form-right-panel :span="12">
        <el-form :inline="true" :model="queryForm" @submit.native.prevent>
          <el-form-item>
            <el-input
              v-model.trim="queryForm.keyword"
              placeholder="用户名/邮箱/手机"
              clearable
              @keyup.enter.native="queryData"
            />
          </el-form-item>
          <el-form-item>
            <el-button icon="el-icon-search" type="primary" @click="queryData">查询</el-button>
          </el-form-item>
        </el-form>
      </vab-query-form-right-panel>
    </vab-query-form>

    <el-table
      v-loading="listLoading"
      :data="list"
      :element-loading-text="elementLoadingText"
      @selection-change="setSelectRows"
      @sort-change="sortChang"
    >
      <el-table-column show-overflow-tooltip type="selection"></el-table-column>
      <el-table-column show-overflow-tooltip prop="id" label="id" :sortable="'custom'"></el-table-column>
      <el-table-column show-overflow-tooltip prop="username" label="用户名" :sortable="'custom'"></el-table-column>
      <el-table-column show-overflow-tooltip prop="nickname" label="昵称" :sortable="'custom'"></el-table-column>
      <el-table-column label="角色">
        <template slot-scope="scope">
          <div v-for="item in scope.row.roles" :key="item.id">
            <el-tag>{{ item.name }}</el-tag>
          </div>
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip prop="avatar" label="头像" :sortable="'custom'">
        <template slot-scope="scope">
          <BImage
            v-if="scope.row.avatar"
            class="user-avatar"
            :src="scope.row.avatar || ''"
            :width="40"
            :height="40"
            :borderRadius="6"
          ></BImage>
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip prop="email" label="邮箱"></el-table-column>
      <el-table-column show-overflow-tooltip prop="phone" label="手机"></el-table-column>
      <el-table-column show-overflow-tooltip prop="state" label="状态" width="100px">
        <template slot-scope="scope">
          <span :style="`color: ${scope.row.state === 1 ? '#67c23a' : '#F56C6C'}`">{{
            scope.row.state === 1 ? '正常' : '停用'
          }}</span>
        </template>
      </el-table-column>
      <el-table-column show-overflow-tooltip prop="last_login" label="最近登录时间"></el-table-column>
      <el-table-column show-overflow-tooltip fixed="right" label="操作" width="200">
        <template v-slot="scope">
          <el-button
            :disabled="
              !$checkPermission(userRolePermissions.doBulkRoleCreate) || !$checkPermission(userRolePermissions.doDelete)
            "
            type="text"
            @click="handleRoleEdit(scope.row)"
            >角色管理</el-button
          >
          <el-button :disabled="!$checkPermission(userPermissions.doEdit)" type="text" @click="handleEdit(scope.row)"
            >编辑</el-button
          >
          <el-button
            v-if="scope.row.id !== userInfo.id"
            :disabled="!$checkPermission(userPermissions.doDelete)"
            type="text"
            @click="handleDelete(scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      :current-page="queryForm.pageNo"
      :page-size="queryForm.pageSize"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    ></el-pagination>
    <edit ref="edit" @fetchData="fetchData"></edit>
    <UserRoleManagementEdit ref="UserRoleManagementEdit" @edited="fetchData"></UserRoleManagementEdit>
  </div>
</template>

<script>
  import BImage from '@/components/B-image';
  import { getList, doDelete, permissions as userPermissions } from '@/api/user';
  import { permissions as userRolePermissions } from '@/api/userRoleManagement';
  import Edit from './components/UserManagementEdit';
  import UserRoleManagementEdit from './components/UserRoleManagementEdit';
  import { mapState } from 'vuex';

  export default {
    name: 'UserManagement',
    components: { Edit, UserRoleManagementEdit, BImage },
    data() {
      return {
        userPermissions,
        userRolePermissions,
        list: null,
        listLoading: true,
        layout: 'total, sizes, prev, pager, next, jumper',
        total: 0,
        selectRows: '',
        elementLoadingText: '正在加载...',
        queryForm: {
          prop_order: '',
          order: '',
          pageNo: 1,
          pageSize: 10,
          keyword: '',
        },
      };
    },
    computed: {
      ...mapState('user', ['userInfo']),
    },
    created() {
      this.fetchData();
    },
    methods: {
      setSelectRows(val) {
        this.selectRows = val;
      },
      sortChang(val) {
        this.queryForm.prop_order = val.prop;
        this.queryForm.order = (val.order && val.order.replace('ending', '')) || '';
        this.queryForm.pageNo = 1;
        this.fetchData();
      },
      handleRoleEdit(row) {
        if (row.id) {
          this.$refs['UserRoleManagementEdit'].showEdit(row);
        } else {
          this.$refs['UserRoleManagementEdit'].showEdit();
        }
      },
      handleEdit(row) {
        this.$refs['edit'].showEdit(row);
      },
      handleDelete(row) {
        if (row.id) {
          this.$baseConfirm('你确定要删除当前项吗', null, async () => {
            await doDelete({ ids: [row.id] });
            this.$baseMessage('删除成功', 'success');
            this.fetchData();
          });
        } else {
          if (this.selectRows.length > 0) {
            const ids = this.selectRows.map(item => item.id);
            this.$baseConfirm('你确定要删除选中项吗', null, async () => {
              await doDelete({ ids });
              this.$baseMessage('删除成功', 'success');
              this.fetchData();
            });
          } else {
            this.$baseMessage('未选中任何行', 'error');
            return false;
          }
        }
      },
      handleSizeChange(val) {
        this.queryForm.pageSize = val;
        this.fetchData();
      },
      handleCurrentChange(val) {
        this.queryForm.pageNo = val;
        this.fetchData();
      },
      queryData() {
        this.queryForm.pageNo = 1;
        this.fetchData();
      },
      async fetchData() {
        this.listLoading = true;
        const { pageSize, pageNo } = this.queryForm;
        const {
          data: { rows, count },
        } = await getList({ ...this.queryForm, limit: pageSize, offset: (pageNo - 1) * pageSize });
        this.list = rows;
        this.total = count;
        this.listLoading = false;
      },
    },
  };
</script>
