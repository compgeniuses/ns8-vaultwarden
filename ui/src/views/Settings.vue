<template>
  <div class="bx--grid bx--grid--full-width">
    <div class="bx--row">
      <div class="bx--col-lg-16 page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </div>
    </div>
    <div v-if="error.getConfiguration" class="bx--row">
      <div class="bx--col">
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </div>
    </div>
    <div class="bx--row">
      <div class="bx--col-lg-16">
        <cv-tile :light="true">
          <cv-form @submit.prevent="configureModule">
            <cv-text-input
              :label="$t('settings.paperless_name')"
              v-model.trim="paperless"
              class="mg-bottom"
              :invalid-message="$t(error.paperless_name)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="paperless"
            >
            </cv-text-input>
            <!-- <cv-text-input
              :label="$t('settings.admin_username')"
              v-model.trim="username"
              class="mg-bottom"
              :invalid-message="$t(error.username)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="username"
            >
            </cv-text-input> -->
            <cv-text-input
              :label="$t('settings.PAPERLESS_SECRET_KEY')"
              v-model.trim="password"
              type="password"
              :password-show-label="$t('settings.show_password')"
              :password-hide-label="$t('settings.hide_password')"
              class="mg-bottom"
              :invalid-message="$t(error.password)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="password"
            >
            </cv-text-input>

            <!-- <cv-text-input
              :label="$t('settings.admin_email')"
              placeholder="admin@example.com"
              v-model.trim="email"
              class="mg-bottom"
              :invalid-message="$t(error.email)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="email"
            >
            </cv-text-input> -->

            <!-- <cv-text-input
              :label="$t('settings.admin_full_name')"
              v-model.trim="userFullName"
              class="mg-bottom"
              :invalid-message="$t(error.user_full_name)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="userFullName"
            >
            </cv-text-input> -->
            <cv-text-input
              :label="$t('settings.PAPERLESS_UR')"
              placeholder="paperless.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            >
            </cv-text-input>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <div v-if="error.configureModule" class="bx--row">
              <div class="bx--col">
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </div>
            </div>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </div>
    </div>
  </div>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [TaskService, IconService, UtilService, QueryParamService],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      paperless: "",
      username: "",
      password: "",
      email: "",
      userFullName: "",
      host: "",
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: false,
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        paperless: "",
        username: "",
        password: "",
        user_full_name: "",
        email: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.core.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.wikiName = config.wiki_name;
      this.username = config.username;
      this.password = config.password;
      this.userFullName = config.user_full_name;
      this.email = config.email;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.loading.getConfiguration = false;
      this.focusElement("wikiName");
    },
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;

      if (!this.wikiName) {
        this.error.wiki_name = "common.required";

        if (isValidationOk) {
          this.focusElement("wikiName");
        }
        isValidationOk = false;
      }

      if (!this.username) {
        this.error.username = "common.required";

        if (isValidationOk) {
          this.focusElement("username");
        }
        isValidationOk = false;
      }

      if (!this.password) {
        this.error.password = "common.required";

        if (isValidationOk) {
          this.focusElement("password");
        }
        isValidationOk = false;
      }

      if (!this.email) {
        this.error.email = "common.required";

        if (isValidationOk) {
          this.focusElement("email");
        }
        isValidationOk = false;
      }

      if (this.email && !/^\S+@\S+$/.test(this.email)) {
        this.error.email = "settings.email_format";

        if (isValidationOk) {
          this.focusElement("email");
        }
        isValidationOk = false;
      }

      if (!this.userFullName) {
        this.error.user_full_name = "common.required";

        if (isValidationOk) {
          this.focusElement("userFullName");
        }
        isValidationOk = false;
      }

      if (!this.host) {
        this.error.host = "common.required";

        if (isValidationOk) {
          this.focusElement("host");
        }
        isValidationOk = false;
      }

      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = "settings." + validationError.error;

        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";

      // register to task error
      this.core.$root.$off(taskAction + "-aborted");
      this.core.$root.$once(
        taskAction + "-aborted",
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$off(taskAction + "-validation-failed");
      this.core.$root.$once(
        taskAction + "-validation-failed",
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$off(taskAction + "-completed");
      this.core.$root.$once(
        taskAction + "-completed",
        this.configureModuleCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            wiki_name: this.wikiName,
            username: this.username,
            password: this.password,
            user_full_name: this.userFullName,
            email: this.email,
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.core.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}
</style>