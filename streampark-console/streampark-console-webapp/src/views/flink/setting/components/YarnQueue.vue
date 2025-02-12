<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<template>
  <div>
    <BasicTable @register="registerTable">
      <template #toolbar>
        <a-button type="primary" @click="handleCreate" v-auth="'yarnQueue:create'">
          <Icon icon="ant-design:plus-outlined" />
          {{ t('common.add') }}
        </a-button>
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'action'">
          <TableAction
            :actions="[
              {
                icon: 'clarity:note-edit-line',
                auth: 'yarnQueue:update',
                tooltip: t('common.edit'),
                onClick: handleYarnQueueEdit.bind(null, record),
              },
              {
                icon: 'ant-design:delete-outlined',
                color: 'error',
                tooltip: t('common.delText'),
                auth: 'yarnQueue:delete',
                popConfirm: {
                  title: t('flink.setting.yarnQueue.deleteConfirm'),
                  confirm: handleDelete.bind(null, record),
                },
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
    <YarnQueueDrawer :okText="t('common.okText')" @register="registerDrawer" @success="handleSuccess" />
  </div>
</template>
<script lang="ts">
  import { defineComponent } from 'vue';

  import { BasicTable, useTable, TableAction } from '/@/components/Table';
  import YarnQueueDrawer from './YarnQueueDrawer.vue';
  import { useDrawer } from '/@/components/Drawer';
  import { columns, searchFormSchema } from './YarnQueue.data';
  import { fetchYarnQueueList, fetchYarnQueueDelete } from '/@/api/flink/setting/yarnQueue';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useI18n } from '/@/hooks/web/useI18n';
  import Icon from '/@/components/Icon';
  export default defineComponent({
    name: 'YarnQueue',
    components: { BasicTable, YarnQueueDrawer, TableAction, Icon },
    setup() {
      const [registerDrawer, { openDrawer }] = useDrawer();
      const { createMessage } = useMessage();
      const { t } = useI18n();
      const [registerTable, { reload }] = useTable({
        title: t('flink.setting.yarnQueue.tableTitle'),
        api: fetchYarnQueueList,
        columns,
        formConfig: {
          baseColProps: { style: { paddingRight: '30px' } },
          schemas: searchFormSchema,
          fieldMapToTime: [['createTime', ['createTimeFrom', 'createTimeTo'], 'YYYY-MM-DD']],
        },
        rowKey: 'id',
        pagination: true,
        useSearchForm: true,
        showTableSetting: true,
        showIndexColumn: false,
        canResize: false,
        actionColumn: {
          width: 200,
          title: t('component.table.operation'),
          dataIndex: 'action',
        },
      });

      function handleCreate() {
        openDrawer(true, {
          isUpdate: false,
        });
      }

      function handleYarnQueueEdit(record: Recordable) {
        openDrawer(true, {
          record,
          isUpdate: true,
        });
      }

      /* Delete the organization */
      async function handleDelete(record: Recordable) {
        const { data } = await fetchYarnQueueDelete({ id: record.id });
        if (data.status === 'success') {
          createMessage.success(
            `${t('flink.setting.yarnQueue.deleteYarnQueue')} ${t(
              'flink.setting.yarnQueue.success',
            )}`,
          );
          reload();
        } else {
          createMessage.error(
            `${t('flink.setting.yarnQueue.deleteYarnQueue')} ${t('common.failed')}`,
          );
        }
      }

      function handleSuccess(isUpdate: boolean) {
        createMessage.success(
          `${isUpdate ? t('common.edit') : t('flink.setting.yarnQueue.createQueue')} ${t(
            'flink.setting.yarnQueue.success',
          )}`,
        );
        reload();
      }

      return {
        t,
        registerTable,
        registerDrawer,
        handleCreate,
        handleYarnQueueEdit,
        handleDelete,
        handleSuccess,
      };
    },
  });
</script>
