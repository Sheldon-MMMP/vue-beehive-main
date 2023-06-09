<template>
  <div class="task-log">
    <div class="wrap-log">
      <div class="wrap-filter">
        <el-dropdown trigger="click" @command="dropdownCommand">
          <span class="el-dropdown-link"> {{ typeName }}<i class="el-icon-arrow-down el-icon--right"></i> </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item v-for="type in types" :key="type.command" :command="type.command"
              >{{ type.label }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
      <div class="log-list color-light">
        <div v-if="btnShowAllText" class="btn-more" @click="showSwitchClick">
          <i class="el-icon-more"></i> {{ btnShowAllText }}
        </div>
        <div v-for="log in dataListFilter" :key="log.id" class="item">
          <div class="info">
            <div class="info-header">
              <i v-if="log.is_comment === 0" :class="['icon', log.icon]"></i>
              <BImage
                v-else
                class="user-avatar"
                :src="(log.operator && log.operator.avatar) || ''"
                :width="24"
                :height="24"
                :borderRadius="24"
              ></BImage>
              <span class="username-and-remark">
                <span :class="[{ 'username-comment': log.is_comment === 1 }]">{{
                  log.operator && log.operator.username
                }}</span>
                {{ log.remark }}
              </span>
            </div>
            <span class="create-date">
              <el-tooltip class="item" effect="dark" :content="log.created_at" :enterable="false" placement="top">
                <div>{{ log.created_at_humanize }}</div>
              </el-tooltip>
            </span>
          </div>
          <LogComment :log="log"></LogComment>
        </div>
      </div>
    </div>
    <div class="wrap-comment">
      <div class="input">
        <el-popover v-model="showUserListPopover" placement="top-start" popper-class="mention-user-list" width="200">
          <div class="user-list">
            <div
              v-for="user in taskInfo.participators"
              :key="user.id"
              class="user-item"
              @click="mentionUserSelect(user)"
            >
              <BImage class="user-avatar" :src="user.avatar || ''" :width="24" :height="24" :borderRadius="24"></BImage>
              <div class="username">{{ user.username }}</div>
            </div>
          </div>
          <div slot="reference"></div>
        </el-popover>
        <el-input
          v-model="content"
          ref="contentInputRef"
          type="textarea"
          :autosize="{ minRows: 1, maxRows: 1 }"
          :disabled="!$checkPermission(taskLogPermissions.doCreate)"
          placeholder="@ 提及他人，按Ctrl + Enter快速发布"
          @keyup.ctrl.enter.native="doCreate"
          @input="contentChange"
        ></el-input>
      </div>
      <div class="ctrl">
        <i class="iconfont icon-emoji btn-emoji"></i>
        <el-button
          :disabled="isOnCreate || !$checkPermission(taskLogPermissions.doCreate)"
          type="text"
          size="medium"
          @click="doCreate"
          >发布
        </el-button>
      </div>
    </div>
  </div>
</template>

<script>
  import BImage from '@/components/B-image';
  import LogComment from './components/LogComment';
  import { getList, doCreate, permissions as taskLogPermissions } from '@/api/taskLogManagement';
  import { doCreate as doCreateMessage } from '@/api/messageManagement';
  import { dateHumanizeFormat } from '@/utils';
  import { mapState } from 'vuex';

  export default {
    name: 'TaskLog',
    components: {
      BImage,
      LogComment,
    },
    props: {
      taskInfo: {
        type: Object,
        required: true,
      },
      taskId: {
        type: Number,
        required: true,
      },
      participators: {
        type: Array,
        required: true,
      },
      dialogTableVisible: {
        type: Boolean,
        required: true,
      },
    },
    data() {
      return {
        taskLogPermissions,
        dataList: [],
        showCount: 5,
        showAll: false,
        isOnCreate: false,
        showUserListPopover: false,
        is_comment: '',
        content: '',
        types: [
          {
            command: '',
            label: '所有动态',
          },
          {
            command: 1,
            label: '仅评论',
          },
          {
            command: 0,
            label: '仅动态',
          },
        ],
      };
    },
    computed: {
      ...mapState('user', ['userInfo']),
      ...mapState('project', ['currentProjectId']),
      dataListFilter() {
        return this.showAll
          ? this.dataList
          : this.dataList.filter((item, index) => {
              return index > this.dataList.length - this.showCount - 1;
            });
      },
      btnShowAllText() {
        if (this.dataList.length <= this.showCount) {
          return false;
        }
        return this.showAll ? '隐藏较早的动态' : `显示较早的 ${this.dataList.length - this.showCount} 条动态`;
      },
      typeName() {
        return this.types.find(type => {
          return type.command === this.is_comment;
        }).label;
      },
    },
    watch: {
      dialogTableVisible(newValue, oldValue) {
        if (!newValue) {
          this.showUserListPopover = newValue;
          this.content = '';
        }
      },
    },
    sockets: {
      sync: function (data) {
        const { params, action } = data;
        switch (action) {
          case 'create:task_log': {
            const taskExisting = this.dataList?.find(item => item.id === params.id);
            // 如果不存在，则添加
            if (!taskExisting) {
              params.operator = this.participators.find(item => item.id === params.operator_id) || {};
              params.created_at = this.$baseDayjs(params.created_at).format('YYYY-MM-DD HH:mm:ss');
              params.created_at_humanize = dateHumanizeFormat(params.created_at).value;
              this.dataList?.push(params);
              this.dataList = this.$baseLodash.sortBy(this.dataList, 'id');
            }
            break;
          }
          case 'update:task_log':
            this.dataList.forEach(item => {
              if (item.id === params.id) {
                Object.assign(item, params);
              }
            });
            break;
          case 'delete:task_log':
            this.dataList = this.dataList?.filter(item => item.id !== params.id);
            break;
          default:
            break;
        }
      },
    },
    created() {},
    methods: {
      async getList() {
        const query = {
          task_id: this.taskId,
        };
        if (this.is_comment !== '') {
          query.is_comment = this.is_comment;
        }
        const {
          data: { rows },
        } = await getList(query);
        this.dataList = rows.map(log => {
          return {
            ...log,
            created_at_humanize: dateHumanizeFormat(log.created_at).value,
          };
        });
      },
      doCreateMessage(content) {
        let mentions = content.match(/(@(\S+)\s)|(@(\S+)$)/g);
        mentions = mentions && mentions.map(item => item.replace(/@|\s|\r\n|\n|\r/g, ''));
        mentions = this.$baseLodash.uniq(mentions);
        if (mentions) {
          // 根据@xxx用户名集合，在参与者中找出对应用户集合，且排除操作者自己
          const receivers = this.participators.filter(user => {
            return mentions.find(username => user.username === username && username !== this.userInfo.username);
          });
          receivers.forEach(receiver => {
            doCreateMessage({
              actor_id: this.userInfo.id,
              receiver_id: receiver.id,
              content: `在任务 <span class="task-name">${this.taskInfo.name}</span> 中@了你`,
              type: 'mention',
              url: `${this.$configSettings.project_path}/${this.currentProjectId}?taskId=${this.taskInfo.id}`,
            });
          });
        }
      },
      async doCreate() {
        if (this.content.trim() === '' || this.isOnCreate) {
          return;
        }
        this.isOnCreate = true;
        const { msg } = await doCreate({
          content: this.content,
          task_id: this.taskInfo.id,
          project_id: this.currentProjectId,
          operator_id: this.userInfo.id,
          type: 'comment',
          is_comment: 1,
        });
        this.isOnCreate = false;
        this.doCreateMessage(this.content);
        this.content = '';
        this.$baseMessage(msg, 'success');
      },
      showSwitchClick() {
        this.showAll = !this.showAll;
      },
      dropdownCommand(command) {
        this.is_comment = command;
        this.getList();
      },
      contentChange(value) {
        console.log(value);
        console.log(/@$/.test(value));
        this.showUserListPopover = /@$/.test(value);
      },
      mentionUserSelect(user) {
        this.showUserListPopover = false;
        console.log(user);
        this.content += user.username + ' ';
        this.$refs.contentInputRef.focus();
      },
    },
  };
</script>

<style lang="scss" scoped>
  .task-log {
    height: 100%;

    .wrap-log {
      height: calc(100vh - 390px);
      padding: 20px 0 20px 20px;

      .wrap-filter {
        padding-bottom: 10px;

        .el-dropdown-link {
          cursor: pointer;
        }

        .el-dropdown-link:hover {
          color: #1b9aee;
        }
      }

      .log-list {
        height: calc(100% - 30px);
        overflow-x: hidden;
        overflow-y: auto;

        .btn-more {
          display: flex;
          align-items: center;
          margin: 10px 0 15px 0;
          cursor: pointer;

          .el-icon-more {
            margin-right: 10px;
          }
        }

        .btn-more:hover {
          color: #1b9aee;
        }

        .item {
          margin-bottom: 15px;
          font-size: 12px;

          .info {
            display: flex;
            align-items: center;
            justify-content: space-between;

            .info-header {
              display: flex;
              align-items: center;

              .icon {
                width: 24px;
                margin-right: 10px;
                font-size: 16px;
                text-align: center;
              }

              .user-avatar {
                margin-right: 10px;
              }

              .username-and-remark {
                flex: 1;
                display: inline-block;
                line-height: 24px;

                .username-comment {
                  color: #262626;
                }
              }
            }

            .create-date {
              width: 130px;
              padding-right: 20px;
              text-align: right;
            }
          }
        }
      }
    }

    .wrap-comment {
      /*height: 96px;*/
      padding: 15px 0;
      border-top: 1px solid #e5e5e5;

      .input {
        margin-bottom: 5px;

        ::v-deep .el-textarea__inner {
          border: none;
        }
      }

      .ctrl {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding-left: 15px;

        .btn-emoji {
          font-size: 20px;
        }
      }
    }
  }
</style>
<style lang="scss">
  .mention-user-list {
    padding: 8px 0px !important;
    .user-list {
      .user-item {
        display: flex;
        align-items: center;
        padding: 5px;
        .user-avatar {
          margin-right: 5px;
        }
        &:hover {
          background-color: #f5f5f5;
          cursor: pointer;
        }
      }
    }
  }
</style>
