<template>
  <div class="login">
    <div class="login-mask" />
    <div class="login-form-wrap">
      <div class="login-form mx-6">
        <div class="login-form__content px-2 py-10">
          <header>
            <img src="/@/assets/images/logo.png" class="mr-4" />
            <h1>{{ title }}</h1>
          </header>

          <a-form class="mx-auto mt-10" :model="formData" :rules="formRules" ref="formRef">
            <a-form-item name="account">
              <a-input size="large" v-model:value="formData.account" placeholder="vben" />
            </a-form-item>
            <a-form-item name="password">
              <a-input-password
                autofocus="autofocus"
                size="large"
                visibilityToggle
                v-model:value="formData.password"
                placeholder="123456"
              />
            </a-form-item>
            <a-form-item name="verify" v-if="openLoginVerify">
              <BasicDragVerify v-model:value="formData.verify" ref="verifyRef" />
            </a-form-item>
            <a-form-item>
              <a-button
                type="primary"
                size="large"
                class="rounded-sm"
                block
                @click="login"
                :loading="formState.loading"
                >登录</a-button
              >
            </a-form-item>
          </a-form>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
  import { defineComponent, reactive, ref, unref, toRaw, computed } from 'vue';
  import { BasicDragVerify, DragVerifyActionType } from '/@/components/Verify/index';
  import { userStore } from '/@/store/modules/user';
  import { appStore } from '/@/store/modules/app';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useSetting } from '/@/hooks/core/useSetting';
  export default defineComponent({
    components: { BasicDragVerify },
    setup() {
      const { globSetting } = useSetting();
      const { notification } = useMessage();
      const formRef = ref<any>(null);
      const verifyRef = ref<RefInstanceType<DragVerifyActionType>>(null);

      const openLoginVerifyRef = computed(() => appStore.getProjectConfig.openLoginVerify);

      const formData = reactive({
        account: 'vben',
        password: '123456',
        verify: undefined,
      });
      const formState = reactive({
        loading: false,
      });

      const formRules = reactive({
        account: [{ required: true, message: '请输入账号', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        verify: unref(openLoginVerifyRef) ? [{ required: true, message: '请通过验证码校验' }] : [],
      });

      function resetVerify() {
        const verify = unref(verifyRef);
        if (!verify) return;
        formData.verify && verify.$.resume();
        formData.verify = undefined;
      }

      async function handleLogin() {
        const form = unref(formRef);
        if (!form) return;
        formState.loading = true;
        try {
          const data = await form.validate();
          const userInfo = await userStore.login(
            toRaw({
              password: data.password,
              username: data.account,
            })
          );
          if (userInfo) {
            notification.success({
              message: '登录成功',
              description: `欢迎回来: ${userInfo.realName}`,
              duration: 3,
            });
          }
        } catch (error) {
        } finally {
          resetVerify();
          formState.loading = false;
        }
      }

      return {
        formRef,
        verifyRef,
        formData,
        formState,
        formRules,
        login: handleLogin,
        openLoginVerify: openLoginVerifyRef,
        title: globSetting && globSetting.title,
      };
    },
  });
</script>
<style lang="less" scoped>
  @import (reference) '../../../design/index.less';

  .login {
    position: relative;
    height: 100vh;
    background: url(../../../assets/images/login/login-bg.png) no-repeat;
    background-size: 100% 100%;

    &-mask {
      display: none;
      height: 100%;
      background: url(../../../assets/images/login/login-in.png) no-repeat;
      background-size: 100% 100%;

      .respond-to(large, { display: block;});
    }

    &-form {
      width: 100%;
      background: @white;
      border: 10px solid rgba(255, 255, 255, 0.5);
      border-width: 10px;
      border-radius: 4px;
      background-clip: padding-box;
      .respond-to(xlarge, { margin: 0 56px});

      &-wrap {
        position: absolute;
        top: 0;
        right: 0;
        display: flex;
        width: 100%;
        height: 100%;
        justify-content: center;
        align-items: center;
        .respond-to(large, { width: 40%;});
        .respond-to(xlarge, { width: 33.3%;});
      }

      &__content {
        width: 100%;
        height: 100%;
        border: 1px solid #999;
        border-radius: 2px;

        header {
          display: flex;
          justify-content: center;
          align-items: center;

          img {
            display: inline-block;
            width: 80px;
          }

          h1 {
            margin-bottom: 0;
            font-size: 24px;
            color: @primary-color;
            text-align: center;
          }
        }

        form {
          width: 80%;
        }
      }
    }
  }
</style>
