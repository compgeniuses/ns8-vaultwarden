<!--
  Copyright (C) 2023 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("status.title") }}</h2>
      </cv-column>
    </cv-row>
    <cv-row v-if="error.getStatus">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-status')"
          :description="error.getStatus"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row v-if="error.listBackupRepositories">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.list-backup-repositories')"
          :description="error.listBackupRepositories"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row v-if="error.listBackups">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.list-backups')"
          :description="error.listBackups"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row>
      <template v-if="host">
        <cv-column :md="4" :max="4">
          <NsInfoCard
            light
            :title="$t('status.paperless_url')"
            :icon="Settings32"
            :loading="loading.getStatus"
            class="min-height-card"
          >
          <template slot="content">
            <div class="card-rows">
              <div class="card-row">
                <NsButton kind="ghost" :icon="Launch20" @click="goToPaperless">
                  {{ $t("status.open_paperless_page") }}
                </NsButton>
              </div>
            </div>
          </template>
          </NsInfoCard>
        </cv-column>
      </template>
      <cv-column :md="4" :max="4">
        <NsInfoCard
          light
          :title="status.instance || '-'"
          :description="$t('status.app_instance')"
          :icon="Application32"
          :loading="loading.getStatus"
          class="min-height-card"
        />
      </cv-column>
      <cv-column :md="4" :max="4">
        <NsInfoCard
          light
          :title="installationNodeTitle"
          :titleTooltip="installationNodeTitleTooltip"
          :description="$t('status.installation_node')"
          :icon="Chip32"
          :loading="loading.getStatus"
          class="min-height-card"
        />
      </cv-column>
      <cv-column :md="4" :max="4">
        <NsBackupCard
          :title="core.$t('backup.title')"
          :noBackupMessage="core.$t('backup.no_backup_configured')"
          :goToBackupLabel="core.$t('backup.go_to_backup')"
          :repositoryLabel="core.$t('backup.repository')"
          :statusLabel="core.$t('common.status')"
          :statusSuccessLabel="core.$t('common.success')"
          :statusNotRunLabel="core.$t('backup.backup_has_not_run_yet')"
          :statusErrorLabel="core.$t('error.error')"
          :completedLabel="core.$t('backup.completed')"
          :durationLabel="core.$t('backup.duration')"
          :totalSizeLabel="core.$t('backup.total_size')"
          :totalFileCountLabel="core.$t('backup.total_file_count')"
          :backupDisabledLabel="core.$t('common.disabled')"
          :showMoreLabel="core.$t('common.show_more')"
          :moduleId="instanceName"
          :moduleUiName="instanceLabel"
          :repositories="backupRepositories"
          :backups="backups"
          :loading="loading.listBackupRepositories || loading.listBackups"
          :coreContext="core"
          light
        />
      </cv-column>
      <cv-column :md="4" :max="4">
        <NsSystemLogsCard
          :title="core.$t('system_logs.card_title')"
          :description="
            core.$t('system_logs.card_description', {
              name: instanceLabel || instanceName,
            })
          "
          :buttonLabel="core.$t('system_logs.card_button_label')"
          :router="core.$router"
          context="module"
          :moduleId="instanceName"
          light
        />
      </cv-column>
    </cv-row>
    <!-- services -->
    <cv-row>
      <cv-column class="page-subtitle">
        <h4>{{ $tc("status.services", 2) }}</h4>
      </cv-column>
    </cv-row>
    <cv-row v-if="!loading.getStatus">
      <cv-column v-if="!status.services.length">
        <cv-tile light>
          <NsEmptyState :title="$t('status.no_services')"> </NsEmptyState>
        </cv-tile>
      </cv-column>
      <cv-column
        v-else
        v-for="(service, index) in status.services"
        :key="index"
        :md="4"
        :max="4"
      >
        <NsSystemdServiceCard
          light
          class="min-height-card"
          :serviceName="service.name"
          :active="service.active"
          :failed="service.failed"
          :enabled="service.enabled"
          :icon="Cube32"
        />
      </cv-column>
    </cv-row>
    <cv-row v-else>
      // ...

      methods: {
        // ...

        listBackupsCompleted(_, taskResult) {
          let backups = taskResult.output.backups;
          backups.sort(this.sortByProperty("name"));

          // get repository name
          for (const backup of backups) {
            const repo = this.backupRepositories.find(
              (r) => r.id == backup.repository
            );

            if (repo) {
              backup.repoName = repo.name;
            }
          }
          this.backups = backups;
          this.loading.listBackups = false;
        },

        listBackupRepositoriesCompleted(_, taskResult) {
          let backupRepositories = taskResult.output.repositories.sort(
            this.sortByProperty("name")
          );
          this.backupRepositories = backupRepositories;
          this.loading.listBackupRepositories = false;
          this.listBackups();
        },

        getStatusCompleted(_, taskResult) {
          this.status = taskResult.output;
          this.loading.getStatus = false;
        },

        beforeRouteLeave(_, next) {
          clearInterval(this.urlCheckInterval);
          next();
        },

        beforeRouteEnter(_, __, next) {
          next((vm) => {
            vm.watchQueryData(vm);
            vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
          });
        },

        beforeUnmount() {
          clearTimeout(this.redirectTimeout);
        },

        // ...
      },

      // ...
