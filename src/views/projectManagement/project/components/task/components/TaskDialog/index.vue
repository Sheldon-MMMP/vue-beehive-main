<template>
  <el-dialog
    class="task-dialog"
    :visible.sync="dialogTableVisible"
    :destroy-on-close="false"
    top="43px"
    width="1300px"
    @close="close"
  >
    <template slot="title">
      <div v-if="taskInfo.is_recycle === 0" class="on-normal">
        <Dropdown :selector="taskTypeSelect" :selectList="taskTypesSelectList" @command="commandTaskType">
          <span class="el-dropdown-link-type">
            <i :class="taskTypeSelect.icon" :style="`color: ${taskTypeSelect.color};margin-right: 5px;`"></i
            >{{ taskTypeSelect.name }}
          </span>
        </Dropdown>
        <TaskController :task="taskInfo"></TaskController>
      </div>
      <div v-else class="on-recycle">
        <div class="tip"><i class="el-icon-delete"></i>任务已在回收站中，不可修改</div>
        <div class="ctrl">
          <el-button type="text" size="medium" icon="el-icon-refresh-left" @click="recoverTaskHandler"
            >恢复内容
          </el-button>
          <el-button type="text" size="medium" icon="el-icon-delete" @click="deleteTaskHandler">彻底删除</el-button>
        </div>
      </div>
    </template>
    <el-row v-loading="loading">
      <el-col :span="14">
        <div class="wrap-task">
          <div class="wrap-name" :class="[{ 'wrap-name-done': taskInfo.is_done === 1 }]">
            <el-input
              v-model="taskInfo.name"
              :disabled="!isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit)"
              type="textarea"
              autosize
              @blur="doEditExec"
            ></el-input>
          </div>
          <div class="wrap-item wrap-state">
            <div class="label"><i class="iconfont icon-xuanzhong2"></i> 完成状态</div>
            <div class="content">
              <Dropdown
                :selector="taskDoneStateSelect"
                :selectList="taskDoneStatesSelectList"
                @command="commandTaskDoneState"
              ></Dropdown>
            </div>
          </div>
          <div class="wrap-item wrap-state">
            <div class="label"><i class="el-icon-pie-chart"></i> 执行状态</div>
            <div class="content">
              <Dropdown
                :selector="taskStateSelect"
                :selectList="taskStatesSelectList"
                @command="commandTaskState"
              ></Dropdown>
            </div>
          </div>
          <div class="wrap-item">
            <div class="label"><i class="el-icon-user"></i> 执行者</div>
            <div class="content">
              <ExecutorSelect
                ref="ExecutorSelect"
                :executorId="taskInfo.executor_id"
                :showNoOne="true"
                @select="executorSelect"
              ></ExecutorSelect>
            </div>
          </div>
          <div class="wrap-item wrap-date">
            <div class="label"><i class="el-icon-date"></i> 时间</div>
            <div class="content">
              <el-date-picker
                v-model="taskInfo.start_date"
                type="datetime"
                placeholder="设置开始时间"
                style="width: 183px"
                size="mini"
                :disabled="!isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit)"
                :picker-options="startDatePickerOptions"
                value-format="yyyy-MM-dd HH:mm:ss"
                default-time="09:00:00"
                @change="startDatePickerChange"
              >
              </el-date-picker>
              <i class="el-icon-minus"></i>
              <el-date-picker
                v-model="taskInfo.end_date"
                type="datetime"
                placeholder="设置截止时间"
                style="width: 183px"
                size="mini"
                :disabled="!isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit)"
                :picker-options="endDatePickerOptions"
                value-format="yyyy-MM-dd HH:mm:ss"
                default-time="18:00:00"
                @change="endDatePickerChange"
              >
              </el-date-picker>
            </div>
          </div>
          <div class="wrap-item">
            <div class="label"><i class="iconfont icon-i"></i> 优先级</div>
            <div class="content">
              <el-dropdown trigger="click" placement="bottom-start" @command="commandTaskPriority">
                <el-tag
                  effect="plain"
                  size="medium"
                  :style="`color: ${taskPrioritySelect.color};border-color: ${taskPrioritySelect.color};cursor: pointer;`"
                  >{{ taskPrioritySelect.name }}
                </el-tag>
                <el-dropdown-menu slot="dropdown" style="width: 200px">
                  <el-dropdown-item
                    v-for="item in taskPrioritysSelectList"
                    :key="item.id"
                    :disabled="item.disabled"
                    :command="item"
                  >
                    <span>
                      <div style="padding: 5px 0px; display: flex; justify-content: space-between; align-items: center">
                        <el-tag
                          effect="plain"
                          size="medium"
                          :style="`color: ${item.color};border-color: ${item.color};`"
                        >
                          {{ item.name }}
                        </el-tag>
                        <i v-if="taskPrioritySelect.id === item.id" class="el-icon-check" style="font-size: 16px"></i>
                      </div>
                    </span>
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </div>
          </div>
          <div class="wrap-item wrap-remark">
            <div class="label"><i class="el-icon-document"></i> 备注</div>
            <div class="content">
              <div
                v-show="!showRichText"
                class="btn-remark"
                :class="[{ 'disabled-custom': !isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit) }]"
                @click="showRichTextClick"
              >
                <div v-if="taskInfo.remark" class="wrap-remark-html" v-html="taskInfo.remark"></div>
                <div v-else class="btn">添加备注</div>
              </div>
              <!--              <div v-show="!showRichText" class="btn-remark" @click="showRichTextClick">-->
              <!--                <div v-if="taskInfo.remark" class="wrap-remark-html" v-html="taskInfo.remark"></div>-->
              <!--                <div v-else class="btn">添加备注</div>-->
              <!--&lt;!&ndash;                <el-button v-else class="btn" type="text">添加备注</el-button>&ndash;&gt;-->
              <!--              </div>-->
              <div v-if="showRichText" class="wrap-rich-text">
                <RichText ref="RichText" :value="taskInfo.remark" @changeValue="richTextChangeValue"></RichText>
                <div class="wrap-btn-remark">
                  <el-button @click="showRichText = false">取消</el-button>
                  <el-button type="primary" @click="saveRichTextValue">保存</el-button>
                </div>
              </div>
            </div>
          </div>
          <div class="wrap-item wrap-tag">
            <div class="label"><i class="el-icon-collection-tag"></i> 标签</div>
            <div class="content">
              <el-tag
                v-for="tag in task_tags_selected"
                :key="tag.id"
                effect="plain"
                :style="`color: ${tag.color};border-color: ${tag.color};`"
                :closable="isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit)"
                @close="closeTaskTag(tag)"
              >
                {{ tag.name }}
              </el-tag>
              <el-popover placement="right" width="230" trigger="click" @hide="hideTaskTag">
                <div class="wrap-task-tags">
                  <TaskTag ref="TaskTag" :taskInfo="taskInfo" @change="taskTagChange"></TaskTag>
                </div>
                <el-button
                  type="text"
                  slot="reference"
                  :disabled="!isCurrentProjectMember || !$checkPermission(taskPermissions.doEdit)"
                  size="medium"
                  class="btn-add-tag"
                >
                  <span v-if="!task_tags_selected.length">添加标签</span>
                  <i v-else class="el-icon-circle-plus"></i>
                </el-button>
              </el-popover>
            </div>
          </div>
          <WorkingHour :task="taskInfo"></WorkingHour>
          <TaskFile :task="taskInfo"></TaskFile>
        </div>
      </el-col>
      <el-col :span="10">
        <div class="wrap-dynamic">
          <Participator
            v-if="taskInfo.participators"
            :taskId="taskId"
            :taskInfo="taskInfo"
            @change="participatorChange"
          ></Participator>
          <TaskLog
            ref="TaskLog"
            :participators="taskInfo.participators || []"
            :taskInfo="taskInfo"
            :dialogTableVisible="dialogTableVisible"
            :taskId="taskId"
          ></TaskLog>
        </div>
      </el-col>
    </el-row>
  </el-dialog>
</template>

<script>
  import { getInfo, doEdit, permissions as taskPermissions } from '@/api/taskManagement';
  import Participator from '../Participator';
  import TaskLog from '../TaskLog/index';
  import WorkingHour from '../WorkingHour/index';
  import { mapGetters, mapState } from 'vuex';
  import ExecutorSelect from '@/components/ExecutorSelect';
  import RichText from '../RichText';
  import TaskTag from '../TaskTag';
  import TaskFile from '../TaskFile/index';
  import Dropdown from '@/components/Dropdown';
  import TaskController from './components/TaskController';
  import mixin from '@/mixins';

  export default {
    name: 'TaskDialog',
    components: {
      ExecutorSelect,
      RichText,
      TaskTag,
      Participator,
      TaskLog,
      WorkingHour,
      TaskFile,
      Dropdown,
      TaskController,
    },
    mixins: [mixin],
    data() {
      return {
        taskPermissions,
        dialogTableVisible: false,
        isEdited: false,
        loading: false,
        taskId: null,
        taskInfo: {
          name: '',
          start_date: '',
          end_date: '',
          remark: '',
        },
        taskInfoOrigin: {}, // 用于记录改动后的task数据
        taskTypeSelect: {},
        taskStateSelect: {},
        taskDoneStateSelect: {},
        taskPrioritySelect: {},
        showRichText: false,
        task_tags_selected: [],
        taskDoneStates: [
          {
            id: 0,
            color: '#7c7c7c',
            icon: 'iconfont icon-fangxing1',
            name: '未完成',
          },
          {
            id: 1,
            color: '#1ae54d',
            icon: 'iconfont icon-xuanzhong2',
            name: '已完成',
          },
        ],
      };
    },
    computed: {
      ...mapState('project', ['taskStates', 'taskPrioritys', 'taskTypes']),
      ...mapGetters('project', ['isCurrentProjectMember']),
      taskTypesSelectList() {
        return this.taskTypes.map(item => {
          item.disabled = !this.isCurrentProjectMember || !this.$checkPermission(taskPermissions.doEdit);
          return item;
        });
      },
      taskDoneStatesSelectList() {
        return this.taskDoneStates.map(item => {
          item.disabled = !this.isCurrentProjectMember || !this.$checkPermission(taskPermissions.doEdit);
          return item;
        });
      },
      taskStatesSelectList() {
        return this.taskStates.map(item => {
          item.disabled = !this.isCurrentProjectMember || !this.$checkPermission(taskPermissions.doEdit);
          return item;
        });
      },
      taskPrioritysSelectList() {
        return this.taskPrioritys.map(item => {
          item.disabled = !this.isCurrentProjectMember || !this.$checkPermission(taskPermissions.doEdit);
          return item;
        });
      },
      startDatePickerOptions() {
        let selectableRange = '00:00:00 - 23:59:59';
        if (this.taskInfo.end_date) {
          const endTime = this.$baseDayjs(this.taskInfo.end_date).subtract(1, 'second').format('HH:mm:ss');
          selectableRange = `00:00:00 - ${endTime}`;
        }
        return {
          disabledDate: time => {
            if (this.taskInfo.end_date) {
              return time.getTime() > new Date(this.taskInfo.end_date).getTime();
            } else {
              return false;
            }
          },
          selectableRange,
        };
      },
      endDatePickerOptions() {
        let selectableRange = '00:00:00 - 23:59:59';
        if (this.taskInfo.start_date) {
          const startTime = this.$baseDayjs(this.taskInfo.start_date).add(1, 'second').format('HH:mm:ss');
          selectableRange = `${startTime} - 23:59:59`;
        }
        return {
          disabledDate: time => {
            if (this.taskInfo.start_date) {
              return time.getTime() < new Date(this.taskInfo.start_date).getTime() - 24 * 60 * 60 * 1000;
            } else {
              return false;
            }
          },
          selectableRange,
        };
      },
    },
    watch: {
      'taskInfo.is_done'(newValue, oldValue) {
        this.taskDoneStateSelect = this.taskDoneStates.find(item => {
          return item.id === newValue;
        });
      },
      'taskInfo.task_type_id'(newValue, oldValue) {
        this.taskTypeSelect = this.taskTypes.find(item => {
          return item.id === newValue;
        });
      },
      'taskInfo.task_state_id'(newValue, oldValue) {
        this.taskStateSelect = this.taskStates.find(item => {
          return item.id === newValue;
        });
      },
      'taskInfo.task_priority_id'(newValue, oldValue) {
        this.taskPrioritySelect = this.taskPrioritys.find(item => {
          return item.id === newValue;
        });
      },
    },
    sockets: {
      sync: function (data) {
        const { params, action } = data;
        switch (action) {
          case 'update:task':
            Object.assign(this.taskInfo, params);
            this.setTaskInfoOrigin();
            break;
          case 'create:task_task_tag': {
            const taskExisting = this.taskInfo.task_tags?.find(item => item.id === params.id);
            // 如果不存在，则添加
            if (!taskExisting) {
              this.taskInfo.task_tags?.push(params);
              this.setTaskInfoOrigin();
            }
            break;
          }
          case 'delete:task_task_tag':
            this.taskInfo.task_tags = this.taskInfo.task_tags?.filter(item => item.id !== params.task_tag_id);
            this.setTaskInfoOrigin();
            break;
          case 'create:user_task': {
            const userExisting = this.taskInfo.participators?.find(item => item.id === params.id);
            // 如果不存在，则添加
            if (!userExisting) {
              this.taskInfo.participators?.push(params);
              this.setTaskInfoOrigin();
            }
            break;
          }
          case 'delete:user_task':
            this.taskInfo.participators = this.taskInfo.participators?.filter(item => item.id !== params.user_id);
            this.setTaskInfoOrigin();
            break;
          default:
            break;
        }
      },
    },
    methods: {
      show(taskId) {
        this.dialogTableVisible = true;
        this.isEdited = false;
        this.showRichText = false;
        this.taskId = taskId;
        this.getInfoExec();
        this.$nextTick(() => {
          this.$refs.TaskLog && this.$refs.TaskLog.getList();
        });
      },
      async getInfoExec() {
        this.loading = true;
        const { data } = await getInfo({ id: this.taskId });
        this.taskInfo = data;
        if (this.taskInfo.remark === null) this.taskInfo.remark = '';
        this.setTaskInfoOrigin();
        this.loading = false;
      },
      setTaskInfoOrigin() {
        this.taskInfoOrigin = this.$baseLodash.cloneDeep(this.taskInfo);
      },
      close() {
        this.dialogTableVisible = false;
        this.$emit('close', this.isEdited);
        this.$router.replace(this.$route.path);
      },
      commandTaskType(type) {
        this.taskTypeSelect = type;
        this.doEditExec();
      },
      commandTaskState(state) {
        this.taskStateSelect = state;
        this.doEditExec();
      },
      commandTaskDoneState(state) {
        this.taskDoneStateSelect = state;
        this.doEditExec();
      },
      commandTaskPriority(priority) {
        this.taskPrioritySelect = priority;
        this.doEditExec();
      },
      startDatePickerChange(date) {
        if (date === null) {
          this.taskInfo.start_date = '';
        }
        this.doEditExec();
      },
      endDatePickerChange(date) {
        if (date === null) {
          this.taskInfo.end_date = '';
        }
        this.doEditExec();
      },
      async doEditExec() {
        this.taskInfo.task_type_id = this.taskTypeSelect.id;
        this.taskInfo.task_state_id = this.taskStateSelect.id;
        this.taskInfo.is_done = this.taskDoneStateSelect.id;
        this.taskInfo.task_priority_id = this.taskPrioritySelect.id;
        const diff = {};
        // 获取改动数据到diff
        for (let taskInfoKey in this.taskInfo) {
          if (!this.$baseLodash.isEqual(this.taskInfo[taskInfoKey], this.taskInfoOrigin[taskInfoKey])) {
            diff[taskInfoKey] = this.taskInfo[taskInfoKey];
          }
        }
        if (Object.keys(diff).length) {
          await doEdit({
            id: this.taskInfo.id,
            ...diff,
          });
          this.isEdited = true;
          this.setTaskInfoOrigin();
        }
      },
      showRichTextClick() {
        if (!this.isCurrentProjectMember || !this.$checkPermission(taskPermissions.doEdit)) return;
        this.showRichText = true;
      },
      richTextChangeValue(value) {
        this.taskInfo.remark = value;
        this.doEditExec();
      },
      saveRichTextValue() {
        this.$refs.RichText.save();
        this.showRichText = false;
      },
      taskTagChange(task_tags_selected) {
        this.task_tags_selected = task_tags_selected;
      },
      closeTaskTag(tag) {
        this.$refs.TaskTag.tagClick(tag);
      },
      hideTaskTag() {
        this.$refs.TaskTag.closeEdit();
      },
      executorSelect(user) {
        this.taskInfo.executor_id = user.id;
        this.doEditExec();
      },
      participatorChange(users) {
        this.taskInfo.participators = users;
      },
      recoverTaskHandler() {
        this.recoverTask(this.taskInfo);
      },
      async deleteTaskHandler() {
        await this.deleteTask(this.taskInfo);
        this.close();
      },
    },
  };
</script>

<style lang="scss" scoped>
  $min-height: calc(100vh - 170px);
  .task-dialog {
    white-space: normal;
    .on-normal {
      display: flex;
      align-items: center;
      justify-content: space-between;
      height: 50px;
      padding: 5px 60px 5px 20px;
    }
    .on-recycle {
      display: flex;
      align-items: center;
      justify-content: space-between;
      height: 50px;
      padding: 5px 60px 5px 20px;
      background-color: $colorF5;
      .tip {
        color: $colorLight;
        & i {
          padding-right: 10px;
        }
      }
      .ctrl {
      }
    }

    ::v-deep .el-dialog__body {
      padding: 0px 20px;
    }
    ::v-deep .el-dialog__header {
      padding: 0px;
    }

    .el-dropdown-link-type {
      cursor: pointer;
      user-select: none;
      padding: 5px;
      background-color: #e1f3fc;
      border-radius: 4px;
      color: #1b9aee;
    }

    .wrap-task {
      height: $min-height;
      padding: 10px 10px 0 0;
      overflow-x: hidden;
      overflow-y: auto;

      .wrap-name {
        ::v-deep .el-textarea__inner {
          border: none;
          font-size: 18px;
        }

        ::v-deep .el-textarea__inner:hover {
          background-color: #f7f7f7;
        }
      }

      .wrap-name-done {
        ::v-deep .el-textarea__inner {
          color: #bfbfbf;
        }
      }

      .wrap-item {
        display: flex;
        align-items: center;
        min-height: 36px;
        padding: 5px 0;

        .iconfont {
          font-size: 14px;
        }

        .label {
          width: 132px;
          color: $colorLight;

          & i {
            font-size: 16px;
            margin-right: 3px;
          }
        }

        .content {
          width: 580px;
        }
      }

      .wrap-state {
      }

      .wrap-date {
        .el-icon-minus {
          padding: 0px 5px;
        }
      }

      .wrap-remark {
        align-items: normal;
        .label {
          margin-top: 8px;
        }
        .btn-remark {
          cursor: pointer;

          .wrap-remark-html {
            min-height: 36px;
          }

          .btn {
            line-height: 36px;
            padding-left: 10px;
          }
        }

        .btn-remark:hover {
          background-color: #f5f5f5;
        }

        .wrap-rich-text {
          min-height: 200px;

          .wrap-btn-remark {
            margin-top: 5px;
            text-align: right;
          }
        }
      }

      .wrap-tag {
        .content {
          display: flex;
          align-items: center;

          .btn-add-tag {
            display: inline-block;

            .el-icon-circle-plus {
              font-size: 18px;
              margin-left: 10px;
            }

            .el-icon-circle-plus:hover {
              color: #3da8f5;
            }
          }

          .btn-add-tag:hover {
            color: #3da8f5;
          }
        }
      }
    }

    .wrap-dynamic {
      min-height: $min-height;
      padding: 10px 0 0 0;
      border-left: 1px solid #ccc;
    }
  }
</style>
