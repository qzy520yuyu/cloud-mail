<template>
  <div id="login-box">
    <div class="login-container">
      <div class="login-card">
        <div class="logo-section">
          <svg class="server-icon" viewBox="0 0 24 24" width="60" height="60">
            <rect x="2" y="3" width="20" height="5" rx="1" fill="#5dade2"/>
            <rect x="2" y="10" width="20" height="5" rx="1" fill="#5dade2"/>
            <circle cx="5" cy="5.5" r="0.8" fill="white"/>
            <circle cx="7" cy="5.5" r="0.8" fill="white"/>
            <circle cx="5" cy="12.5" r="0.8" fill="white"/>
            <circle cx="7" cy="12.5" r="0.8" fill="white"/>
          </svg>
        </div>
        
        <h1 class="form-title">{{ settingStore.settings.title }}</h1>
        <p class="form-subtitle">{{ show === 'login' ? $t('loginTitle') : $t('regTitle') }}</p>
        
        <!-- 登录表单 -->
        <div v-show="show === 'login'" class="form-content">
          <div class="form-group">
            <div class="input-wrapper">
              <el-input 
                :class="settingStore.settings.loginDomain === 0 ? 'email-input' : ''" 
                v-model="form.email"
                type="text" 
                :placeholder="$t('emailAccount')" 
                autocomplete="off">
                <template #append v-if="settingStore.settings.loginDomain === 0">
                  <div @click.stop="openSelect">
                    <el-select
                        v-if="show === 'login'"
                        ref="mySelect"
                        v-model="suffix"
                        :placeholder="$t('select')"
                        class="select"
                    >
                      <el-option
                          v-for="item in domainList"
                          :key="item"
                          :label="item"
                          :value="item"
                      />
                    </el-select>
                    <div style="color: var(--el-text-color-primary)">
                      <span>{{ suffix }}</span>
                      <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                    </div>
                  </div>
                </template>
              </el-input>
            </div>
          </div>
          
          <div class="form-group">
            <el-input 
              v-model="form.password" 
              :placeholder="$t('password')" 
              type="password" 
              autocomplete="off">
            </el-input>
          </div>
          
          <el-button class="btn btn-primary" type="primary" @click="submit" :loading="loginLoading">
            <Icon icon="mingcute:arrow-right-fill" width="18" height="18" style="margin-right: 5px"/>
            {{ $t('loginBtn') }}
          </el-button>
          
          <div class="switch-link" v-if="settingStore.settings.register === 0">
            {{ $t('noAccount') }} 
            <span class="link-button" @click="show = 'register'">{{ $t('regSwitch') }}</span>
          </div>
        </div>
        
        <!-- 注册表单 -->
        <div v-show="show !== 'login'" class="form-content">
          <div class="form-group">
            <div class="input-wrapper">
              <el-input 
                class="email-input" 
                v-model="registerForm.email" 
                type="text" 
                :placeholder="$t('emailAccount')"
                autocomplete="off">
                <template #append>
                  <div @click.stop="openSelect">
                    <el-select
                        v-if="show !== 'login'"
                        ref="mySelect"
                        v-model="suffix"
                        :placeholder="$t('select')"
                        class="select"
                    >
                      <el-option
                          v-for="item in domainList"
                          :key="item"
                          :label="item"
                          :value="item"
                      />
                    </el-select>
                    <div>
                      <span>{{ suffix }}</span>
                      <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                    </div>
                  </div>
                </template>
              </el-input>
            </div>
          </div>
          
          <div class="form-group">
            <el-input 
              v-model="registerForm.password" 
              :placeholder="$t('password')" 
              type="password" 
              autocomplete="off"/>
          </div>
          
          <div class="form-group">
            <el-input 
              v-model="registerForm.confirmPassword" 
              :placeholder="$t('confirmPwd')" 
              type="password"
              autocomplete="off"/>
          </div>
          
          <div class="form-group" v-if="settingStore.settings.regKey === 0 || settingStore.settings.regKey === 2">
            <el-input 
              v-model="registerForm.code" 
              :placeholder="settingStore.settings.regKey === 0 ? $t('regKey') : $t('regKeyOptional')" 
              type="text" 
              autocomplete="off"/>
          </div>
          
          <div v-show="verifyShow"
               class="register-turnstile"
               :data-sitekey="settingStore.settings.siteKey"
               data-callback="onTurnstileSuccess"
               data-error-callback="onTurnstileError"
               data-after-interactive-callback="loadAfter"
               data-before-interactive-callback="loadBefore"
          >
            <span style="font-size: 12px;color: #F56C6C" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
          </div>
          
          <el-button class="btn btn-primary" type="primary" @click="submitRegister" :loading="registerLoading">
            {{ $t('regBtn') }}
          </el-button>
          
          <div class="switch-link" v-if="settingStore.settings.register === 0">
            {{ $t('hasAccount') }} 
            <span class="link-button" @click="show = 'login'">{{ $t('loginSwitch') }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import router from "@/router";
import {computed, nextTick, reactive, ref} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";

const {t} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();
const loginLoading = ref(false)
const show = ref('login')
const form = reactive({
  email: '',
  password: '',

});
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
})
const domainList = settingStore.domainList;
const registerLoading = ref(false)
suffix.value = domainList[0]
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};

window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) {
    return
  }
  verifyErrorCount++
  console.warn('人机验加载失败', e)
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile')
      } else {
        window.turnstile.reset(turnstileId);
      }
    })
  }, 1500)
};

window.loadAfter = (e) => {
  console.log('loadAfter')
}

window.loadBefore = (e) => {
  console.log('loadBefore')
}

const openSelect = () => {
  mySelect.value.toggleMenu()
}

const submit = () => {

  if (!form.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = form.email + (settingStore.settings.loginDomain === 0 ? suffix.value : '');

  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!form.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  loginLoading.value = true
  login(email, form.password).then(async data => {
    localStorage.setItem('token', data.token)
    const user = await loginUserInfo();
    accountStore.currentAccountId = user.accountId;
    userStore.user = user;
    const routers = permsToRouter(user.permKeys);
    routers.forEach(routerData => {
      router.addRoute('layout', routerData);
    });
    await router.replace({name: 'layout'})
    uiStore.showNotice()
  }).finally(() => {
    loginLoading.value = false
  })
}


function submitRegister() {

  if (!registerForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!isEmail(registerForm.email + suffix.value)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!registerForm.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password.length < 6) {
    ElMessage({
      message: t('pwdLengthMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password !== registerForm.confirmPassword) {

    ElMessage({
      message: t('confirmPwdFailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!registerForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  if (!verifyToken && (settingStore.settings.registerVerify === 0 || (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen))) {
    if (!verifyShow.value) {
      verifyShow.value = true
      nextTick(() => {
        if (!turnstileId) {
          try {
            turnstileId = window.turnstile.render('.register-turnstile')
          } catch (e) {
            botJsError.value = true
            console.log('人机验证js加载失败')
          }
        } else {
          window.turnstile.reset('.register-turnstile')
        }
      })
    } else if (!botJsError.value) {
      ElMessage({
        message: t('botVerifyMsg'),
        type: "error",
        plain: true
      })
    }
    return;
  }

  registerLoading.value = true

  const form = {
    email: registerForm.email + suffix.value,
    password: registerForm.password,
    token: verifyToken,
    code: registerForm.code
  }

  register(form).then(({regVerifyOpen}) => {
    show.value = 'login'
    registerForm.email = ''
    registerForm.password = ''
    registerForm.confirmPassword = ''
    registerForm.code = ''
    registerLoading.value = false
    verifyToken = ''
    settingStore.settings.regVerifyOpen = regVerifyOpen
    verifyShow.value = false
    ElMessage({
      message: t('regSuccessMsg'),
      type: 'success',
      plain: true,
    })
  }).catch(res => {

    registerLoading.value = false

    if (res.code === 400) {
      verifyToken = ''
      settingStore.settings.regVerifyOpen = true
      if (turnstileId) {
        window.turnstile.reset(turnstileId)
      } else {
        nextTick(() => {
          turnstileId = window.turnstile.render('.register-turnstile')
        })
      }
      verifyShow.value = true

    }
  });
}

</script>


<style>
.el-select-dropdown__item {
  padding: 0 15px;
}

.no-autofill-pwd {
  .el-input__inner {
    -webkit-text-security: disc !important;
  }
}
</style>

<style lang="scss" scoped>

#login-box {
  background: linear-gradient(135deg, #e8eaf6 0%, #f5f5f5 100%);
  font-family: Arial, sans-serif;
  height: 100vh;
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.login-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 20px;
}

.login-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  padding: 48px 40px;
  width: 100%;
  max-width: 440px;
  
  @media (max-width: 767px) {
    padding: 32px 24px;
    max-width: 100%;
  }
}

.logo-section {
  display: flex;
  justify-content: center;
  margin-bottom: 24px;
}

.server-icon {
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
}

.form-title {
  font-size: 28px;
  font-weight: bold;
  text-align: center;
  color: #333;
  margin: 0 0 8px 0;
}

.form-subtitle {
  font-size: 14px;
  color: #888;
  text-align: center;
  margin: 0 0 32px 0;
}

.form-content {
  width: 100%;
}

.form-group {
  margin-bottom: 20px;
}

.form-label {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 4px;
  font-size: 13px;
  font-weight: 500;
  margin-bottom: 8px;
  color: white;
  
  &.label-green {
    background: linear-gradient(135deg, #9acd32 0%, #7cb342 100%);
  }
  
  &.label-orange {
    background: linear-gradient(135deg, #ffa726 0%, #fb8c00 100%);
  }
  
  &.label-purple {
    background: linear-gradient(135deg, #ab47bc 0%, #8e24aa 100%);
  }
  
  &.label-blue {
    background: linear-gradient(135deg, #42a5f5 0%, #1e88e5 100%);
  }
}

.input-wrapper {
  width: 100%;
}

:deep(.el-input__wrapper) {
  border-radius: 8px;
  background: var(--el-bg-color);
  box-shadow: 0 0 0 1px #e0e0e0 inset;
  transition: all 0.3s;
  
  &:hover {
    box-shadow: 0 0 0 1px #b0b0b0 inset;
  }
  
  &.is-focus {
    box-shadow: 0 0 0 2px #5dade2 inset;
  }
}

.email-input :deep(.el-input__wrapper) {
  border-radius: 8px 0 0 8px;
  background: var(--el-bg-color);
}

.el-input {
  width: 100%;
  
  :deep(.el-input__inner) {
    height: 40px;
    padding-left: 0 !important;
    padding-right: 15px;
    text-align: left;
  }
  
  :deep(.el-input__inner::placeholder) {
    padding-left: 0;
    text-indent: 0;
  }
}

:deep(.el-input-group__append) {
  padding: 0 12px !important;
  background: var(--el-bg-color);
  border-radius: 0 8px 8px 0;
  box-shadow: 0 0 0 1px #e0e0e0 inset;
  cursor: pointer;
}

.btn {
  width: 100%;
  height: 44px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  margin-top: 8px;
  transition: all 0.3s;
  
  &.btn-primary {
    background: linear-gradient(135deg, #5dade2 0%, #3498db 100%);
    border: none;
    
    &:hover {
      background: linear-gradient(135deg, #4a9fd5 0%, #2980b9 100%);
      transform: translateY(-1px);
      box-shadow: 0 4px 12px rgba(93, 173, 226, 0.4);
    }
    
    &:active {
      transform: translateY(0);
    }
  }
}

.switch-link {
  text-align: center;
  margin-top: 24px;
  font-size: 14px;
  color: #666;
  
  .link-button {
    color: #e91e63;
    background: linear-gradient(135deg, #ff4081 0%, #e91e63 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    font-weight: 500;
    cursor: pointer;
    padding: 4px 12px;
    border-radius: 4px;
    transition: all 0.3s;
    
    &:hover {
      background: #e91e63;
      -webkit-text-fill-color: white;
      color: white;
    }
  }
}

.register-turnstile {
  margin-bottom: 18px;
}

.select {
  position: absolute;
  right: 30px;
  width: 100px;
  opacity: 0;
  pointer-events: none;
}

.setting-icon {
  position: relative;
  top: 6px;
}

:deep(.el-select-dropdown__item) {
  padding: 0 10px;
}

</style>
