<script setup lang="ts">
import type { RuleObject } from 'ant-design-vue/es/form'
import { definePageMeta, iconMap, reactive, ref, useApi, useI18n, validateEmail } from '#imports'

definePageMeta({
  requiresAuth: false,
})

const route = useRoute()

const { api, isLoading, error } = useApi({ useGlobalInstance: true })

const { t } = useI18n()

const success = ref(false)

const formValidator = ref()

const form = reactive({
  email: '',
})

const formRules = {
  email: [
    // E-mail is required
    { required: true, message: t('msg.error.signUpRules.emailRequired') },
    // E-mail must be valid format
    {
      validator: (_: unknown, v: string) => {
        return new Promise((resolve, reject) => {
          if (!v?.length || validateEmail(v)) return resolve()

          reject(new Error(t('msg.error.signUpRules.emailInvalid')))
        })
      },
      message: t('msg.error.signUpRules.emailInvalid'),
    },
  ] as RuleObject[],
}

async function resetPassword() {
  if (!formValidator.value.validate()) return

  resetError()

  await api.auth.passwordForgot(form).then(() => {
    success.value = true
  })
}

function resetError() {
  if (error.value) error.value = null
}

function navigateSignIn() {
  navigateTo({
    path: '/signin',
    query: route.query,
  })
}
</script>

<template>
  <NuxtLayout>
    <div class="md:bg-primary bg-opacity-5 forgot-password h-full min-h-[600px] flex flex-col justify-center items-center">
      <div
        class="bg-white mt-[60px] relative flex flex-col justify-center gap-2 w-full max-w-[500px] mx-auto p-8 md:(rounded-lg border-1 border-gray-200 shadow-xl)"
      >
        <LazyGeneralNocoIcon class="color-transition hover:(ring ring-accent ring-opacity-100)" :animate="isLoading" />

        <div class="self-center flex flex-col justify-center items-center text-center gap-2">
          <h1 class="prose-2xl font-bold my-4 w-full">{{ $t('title.resetPassword') }}</h1>

          <template v-if="!success">
            <div class="prose-sm">{{ $t('msg.info.passwordRecovery.message_1') }}</div>
            <div class="prose-sm mb-4">{{ $t('msg.info.passwordRecovery.message_2') }}</div>
          </template>

          <template v-else>
            <div class="prose-sm text-success flex items-center leading-8 gap-2">
              {{ $t('msg.info.passwordRecovery.success') }} <ClaritySuccessLine />
            </div>

            <nuxt-link @click="navigateSignIn">{{ $t('general.signIn') }}</nuxt-link>
          </template>
        </div>

        <a-form ref="formValidator" layout="vertical" :model="form" no-style @finish="resetPassword">
          <Transition name="layout">
            <div v-if="error" class="self-center mb-4 bg-red-500 text-white rounded-lg w-3/4 mx-auto p-1">
              <div class="flex items-center gap-2 justify-center">
                <MaterialSymbolsWarning />
                <div class="break-words">{{ error }}</div>
              </div>
            </div>
          </Transition>

          <a-form-item :label="$t('labels.email')" name="email" :rules="formRules.email">
            <a-input v-model:value="form.email" size="large" :placeholder="$t('msg.info.signUp.workEmail')" @focus="resetError" />
          </a-form-item>

          <div class="self-center flex flex-col gap-4 items-center justify-center w-full">
            <button class="scaling-btn bg-opacity-100" type="submit">
              <span class="flex items-center gap-2">
                <component :is="iconMap.signin" />
                {{ $t('activity.sendEmail') }}
              </span>
            </button>

            <div class="text-end prose-sm">
              {{ $t('msg.info.signUp.alreadyHaveAccount') }}
              <nuxt-link @click="navigateSignIn">{{ $t('general.signIn') }}</nuxt-link>
            </div>
          </div>
        </a-form>
      </div>
    </div>
  </NuxtLayout>
</template>

<style lang="scss">
.forgot-password {
  .ant-input-affix-wrapper,
  .ant-input {
    @apply !appearance-none my-1 border-1 border-solid border-primary border-opacity-50 rounded;
  }
}
</style>
