<template>
  <div class="register-container">
    <ShapeShifter></ShapeShifter>
    <div class="content">
      <el-row>
        <el-col :xs="24" :sm="24" :md="12" :lg="16" :xl="16">
          <div style="color: transparent">占位符</div>
        </el-col>
        <el-col :xs="24" :sm="24" :md="12" :lg="8" :xl="8" @click.stop="() => {}">
          <el-form ref="registerForm" :model="form" class="register-form" :rules="registerRules" size="mini">
            <el-form-item prop="email">
              <el-input
                v-model.trim="form.email"
                type="text"
                placeholder="请输入邮箱"
                show-word-limit
                autocomplete="off"
              >
                <i slot="prefix" class="iconfont icon-youxiang"></i>
              </el-input>
            </el-form-item>
            <el-form-item prop="code" style="position: relative">
              <el-input v-model.trim="form.code" type="text" placeholder="验证码">
                <i slot="prefix" class="iconfont icon-yanzhengma"></i>
              </el-input>
              <el-button type="primary" class="show-pwd phone-code" :disabled="isGetphone" @click="getPhoneCode">
                {{ codeButtonText }}
              </el-button>
            </el-form-item>
            <el-form-item prop="password">
              <el-input v-model.trim="form.password" type="password" placeholder="新密码" autocomplete="new-password">
                <i slot="prefix" class="iconfont icon-suo"></i>
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button class="register-btn" type="primary" @click.native.prevent="handleReister">提交</el-button>
              <div class="router-link-box" style="margin-top: 20px">
                <router-link to="/login">登录</router-link>
              </div>
            </el-form-item>
          </el-form>
        </el-col>
      </el-row>
    </div>
  </div>
</template>
<script>
  import { isPassword, isEmail } from '@/utils/validate';
  import { usersPassword } from '@/api/user';
  import { create as createVerificationCode } from '@/api/verification_code';
  import ShapeShifter from '@/components/ShapeShifter';

  export default {
    username: 'Register',
    components: {
      ShapeShifter,
    },
    directives: {
      focus: {
        inserted(el) {
          el.querySelector('input').focus();
        },
      },
    },
    data() {
      const validateusername = (rule, value, callback) => {
        if ('' == value) {
          callback(new Error('用户名不能为空'));
        } else {
          callback();
        }
      };
      const validatePassword = (rule, value, callback) => {
        if (!isPassword(value)) {
          callback(new Error('密码不能少于6位'));
        } else {
          callback();
        }
      };
      const validatePhone = (rule, value, callback) => {
        if (!isEmail(value)) {
          callback(new Error('请输入正确的邮箱'));
        } else {
          callback();
        }
      };
      return {
        isGetphone: false,
        getPhoneIntval: null,
        codeButtonText: '获取验证码',
        showRegister: false,
        nodeEnv: process.env.NODE_ENV,
        title: this.$baseTitle,
        form: {},
        registerRules: {
          email: [
            { required: true, trigger: 'blur', message: '请输入邮箱' },
            { validator: validatePhone, trigger: 'blur' },
          ],
          password: [
            { required: true, trigger: 'blur', message: '请输入新密码' },
            { validator: validatePassword, trigger: 'blur' },
          ],
          code: [{ required: true, trigger: 'blur', message: '请输入验证码' }],
        },
        loading: false,
        passwordType: 'password',
      };
    },
    created() {},
    mounted() {},
    beforeDestroy() {
      this.getPhoneIntval = null;
      clearInterval(this.getPhoneIntval);
    },
    methods: {
      async getPhoneCode() {
        this.$refs['registerForm'].validateField('email', async errorMessage => {
          if (!errorMessage) {
            this.isGetphone = true;
            let n = 3;
            this.getPhoneIntval = setInterval(() => {
              if (n > 0) {
                n--;
                this.codeButtonText = '重新获取(' + n + 's)';
              } else {
                this.getPhoneIntval = null;
                clearInterval(this.getPhoneIntval);
                this.codeButtonText = '获取验证码';
                this.isGetphone = false;
              }
            }, 1000);
            const { code } = await createVerificationCode({
              target: this.form.email,
              type: 1,
            });
            if (code === 0) {
              this.$baseMessage(`验证码发送到${this.form.email}, 请查收！`, 'success');
            }
          }
        });
      },
      handleReister() {
        this.$refs['registerForm'].validate(async valid => {
          if (valid) {
            const { code, msg } = await usersPassword({ ...this.form });
            switch (code) {
              case 0:
                this.$baseMessage('新密码设置成功', 'success');
                break;
              default:
                this.$baseMessage(msg, 'error');
                break;
            }
          }
        });
      },
    },
  };
</script>
<style lang="scss" scoped>
  .register-container {
    position: relative;
    height: 100vh;
    background: -webkit-linear-gradient(top, #fff6f2 0, #fbdd9f 120%);

    .content {
      position: absolute;
      top: 0;
      left: 0;
      width: 100vw;

      .title {
        font-size: 54px;
        font-weight: 500;
        color: rgba(14, 18, 26, 1);
      }

      .title-tips {
        margin-top: 29px;
        font-size: 26px;
        font-weight: 400;
        color: rgba(14, 18, 26, 1);
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      .register-btn {
        display: inherit;
        width: 220px;
        height: 60px;
        margin-top: 5px;
        border: 0;

        &:hover {
          opacity: 0.9;
        }
      }

      ::v-deep {
        .el-button {
          background-color: $color-lucency-btn-bgc;
          color: #000 !important;
        }
      }

      .router-link-box {
        & a {
          color: #000;
        }
      }

      .register-form {
        position: relative;
        max-width: 100%;
        margin: calc((100vh - 499px) / 2) 10% 10%;
        overflow: hidden;

        .forget-password {
          width: 100%;
          margin-top: 40px;
          text-align: left;

          .forget-password {
            width: 129px;
            height: 19px;
            font-size: 20px;
            font-weight: 400;
            color: rgba(92, 102, 240, 1);
          }
        }

        .per-code {
          width: 100px;
          height: 36px;
          font-size: 20px;
          line-height: 36px;
          color: #fff;
          text-align: center;
          cursor: pointer;
          background: #bbc1ce;
        }

        .phone-code {
          width: 120px;
          height: 36px;
          font-size: 14px;
          color: #fff;
          border-radius: 3px;
        }
      }

      .tips {
        margin-bottom: 10px;
        font-size: $base-font-size-default;
        color: $base-color-white;

        span {
          &:first-of-type {
            margin-right: 16px;
          }
        }
      }

      .title-container {
        position: relative;

        .title {
          margin: 0 auto 40px auto;
          font-size: 34px;
          font-weight: bold;
          color: $base-color-blue;
          text-align: center;
        }
      }

      .svg-container {
        position: absolute;
        top: 14px;
        left: 15px;
        z-index: $base-z-index;
        font-size: 16px;
        color: #d7dee3;
        cursor: pointer;
        user-select: none;
      }

      .show-pwd {
        position: absolute;
        top: 14px;
        right: 25px;
        font-size: 16px;
        color: $base-font-color;
        cursor: pointer;
        user-select: none;
      }

      ::v-deep {
        .el-form-item {
          padding-right: 0;
          margin: 20px 0;
          color: #454545;
          background: transparent;
          border: 1px solid transparent;
          border-radius: 2px;

          &__content {
            min-height: $base-input-height;
            line-height: $base-input-height;
          }

          &__error {
            position: absolute;
            top: 100%;
            left: 18px;
            font-size: $base-font-size-small;
            line-height: 18px;
            color: $base-color-red;
          }
        }

        .el-input {
          box-sizing: border-box;

          .el-input__count {
            .el-input__count-inner {
              background: transparent;
            }
          }

          .el-input__prefix {
            left: 15px;
            line-height: 56px;

            .svg-inline--fa {
              width: 20px;
            }
          }

          input {
            height: 58px;
            padding-left: 45px;
            font-size: $base-font-size-default;
            line-height: 58px;
            color: $base-font-color;
            background: $color-lucency-input-bgc;
            border: 0;
            caret-color: $base-font-color;
          }
        }
      }
    }
  }
</style>
