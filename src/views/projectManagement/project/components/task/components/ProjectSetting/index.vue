<template>
  <div class="project-setting">
    <el-popover placement="bottom" width="230" trigger="click">
      <div class="project-setting-wrap-content">
        <div class="wrap-setting-list">
          <div v-for="(item, index) in settings" :key="index" class="item" @click="select(index)">
            <i :class="item.icon"></i>
            {{ item.title }}
          </div>
        </div>
      </div>
      <slot slot="reference"></slot>
    </el-popover>
    <ProjectEdit ref="ProjectEdit"></ProjectEdit>
    <RecycleDialog ref="RecycleDialog"></RecycleDialog>
  </div>
</template>

<script>
  import ProjectEdit from '@/views/projectManagement/projectList/components/ProjectEdit';
  import RecycleDialog from './components/RecycleDialog';
  import mixin from '@/mixins';
  import { mapGetters } from 'vuex';

  export default {
    name: 'ProjectSetting',
    components: {
      ProjectEdit,
      RecycleDialog,
    },
    mixins: [mixin],
    data() {
      return {
        settings: [
          {
            icon: 'el-icon-setting',
            title: '项目设置',
          },
          {
            icon: 'el-icon-collection-tag',
            title: '标签 *',
          },
          {
            icon: 'el-icon-delete',
            title: '查看回收站',
          },
          {
            icon: 'el-icon-document-checked',
            title: '复制项目链接',
          },
          {
            icon: 'el-icon-document-copy',
            title: '复制项目 *',
          },
          {
            icon: 'el-icon-folder-add',
            title: '保存为项目模板 *',
          },
        ],
      };
    },
    computed: {
      ...mapGetters('project', ['currentProject']),
    },
    methods: {
      select(index) {
        switch (index) {
          case 0:
            this.$refs['ProjectEdit'].showEdit(this.currentProject);
            break;
          case 1:
            break;
          case 2:
            this.$refs.RecycleDialog.show();
            break;
          case 3:
            this.doCopy(`${window.location.origin}${this.$route.path}`);
            this.$baseNotify('可粘贴到地址栏中，快速打开此项目', '复制项目链接成功');
            break;
          case 4:
            break;
          case 5:
            break;
          default:
            break;
        }
      },
    },
  };
</script>

<style lang="scss">
  .project-setting-wrap-content {
    .wrap-setting-list {
      .item {
        display: flex;
        align-items: center;
        height: 40px;
        padding: 0 10px;
        font-weight: 600;
        cursor: pointer;
        user-select: none;
        & i {
          margin-right: 10px;
          font-size: 16px;
        }
        &:hover {
          background-color: #eeeeee;
        }
      }
    }
  }
</style>
<style lang="scss" scoped>
  .project-setting {
  }
</style>
