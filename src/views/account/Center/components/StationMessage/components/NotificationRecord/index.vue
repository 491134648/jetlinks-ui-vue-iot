<template>
    <div class="notification-record-container">
        <pro-search
            :columns="columns"
            :target="type"
            style="padding: 0"
            @search="(e) => (queryParams = e)"
        />
        <j-pro-table
            ref="tableRef"
            :columns="columns"
            :request="getList_api"
            model="TABLE"
            :params="queryParams"
            :bodyStyle="{ padding: 0 }"
            :defaultParams="defaultParams"
        >
            <!-- <template #rightExtraRender>
                <j-popconfirm title="确认全部已读？" @confirm="onAllRead">
                    <j-button type="primary">全部已读</j-button>
                </j-popconfirm>
            </template> -->
            <template #topicProvider="slotProps">
                {{ slotProps.topicName }}
            </template>
            <template #notifyTime="slotProps">
                {{ dayjs(slotProps.notifyTime).format('YYYY-MM-DD HH:mm:ss') }}
            </template>
            <template #state="slotProps">
                <BadgeStatus
                    :status="slotProps.state.value"
                    :text="slotProps.state.text"
                    :statusNames="{
                        read: 'success',
                        unread: 'error',
                    }"
                ></BadgeStatus>
            </template>
            <template #action="slotProps">
                <j-space :size="16">
                    <PermissionButton
                        type="link"
                        :popConfirm="{
                            title: `确认标为${
                                slotProps.state.value === 'read'
                                    ? '未读'
                                    : '已读'
                            }`,
                            onConfirm: () => changeStatus(slotProps),
                        }"
                        :tooltip="{
                            title:
                                slotProps.state.value === 'read'
                                    ? '标为未读'
                                    : '标为已读',
                        }"
                    >
                        <AIcon type="icon-a-PIZHU1" />
                    </PermissionButton>
                    <PermissionButton
                        type="link"
                        :tooltip="{
                            title: '查看',
                        }"
                        @click="view(slotProps)"
                    >
                        <AIcon type="SearchOutlined" />
                    </PermissionButton>
                </j-space>
            </template>
        </j-pro-table>
        <ViewDialog
            v-if="viewVisible"
            v-model:visible="viewVisible"
            :data="viewItem"
            :type="type"
        />
    </div>
</template>

<script setup lang="ts" name="NotificationRecord">
import ViewDialog from './components/ViewDialog.vue';
import PermissionButton from '@/components/PermissionButton/index.vue';
import {
    getList_api,
    changeStatus_api,
    changeAllStatus,
} from '@/api/account/notificationRecord';
import dayjs from 'dayjs';
import { useUserInfo } from '@/store/userInfo';
import { useRouterParams } from '@/utils/hooks/useParams';
import { getTypeList_api } from '@/api/account/notificationSubscription';
import { onlyMessage } from '@/utils/comm';

const user = useUserInfo();

const props = defineProps({
    type: {
        type: String,
        default: '',
    },
});

const getType = computed(() => {
    if (props.type === 'system-business') {
        return ['device-transparent-codec'];
    } else if (props.type === 'system-monitor') {
        return ['system-event'];
    } else {
        return [
            'alarm',
            'alarm-product',
            'alarm-device',
            'alarm-other',
            'alarm-org',
        ];
    }
});

const columns = [
    {
        title: '类型',
        dataIndex: 'topicProvider',
        key: 'topicProvider',
        search: {
            type: 'select',
            options: () =>
                getTypeList_api().then((resp: any) => {
                    return resp.result
                        .map((item: any) => ({
                            label: item.name,
                            value: item.id,
                        }))
                        .filter((item: any) => {
                            return [...getType.value].includes(item?.value)
                        }).sort((a: any, b: any) => {
                            return b?.value?.length - a?.value?.length
                        });
                }),
        },
        scopedSlots: true,
        ellipsis: true,
    },
    {
        title: '消息',
        dataIndex: 'message',
        key: 'message',
        search: {
            type: 'string',
        },
        scopedSlots: true,
        ellipsis: true,
    },
    {
        title: '通知时间',
        dataIndex: 'notifyTime',
        key: 'notifyTime',
        search: {
            type: 'date',
        },
        scopedSlots: true,
        ellipsis: true,
    },
    {
        title: '状态',
        dataIndex: 'state',
        key: 'state',
        search: {
            type: 'select',
            options: [
                {
                    label: '未读',
                    value: 'unread',
                },
                {
                    label: '已读',
                    value: 'read',
                },
            ],
        },
        scopedSlots: true,
        ellipsis: true,
    },
    {
        title: '操作',
        dataIndex: 'action',
        key: 'action',
        ellipsis: true,
        scopedSlots: true,
        width: '200px',
    },
];

const viewVisible = ref<boolean>(false);
const viewItem = ref<any>({});

const routerParams = useRouterParams();

const defaultParams = {
    sorts: [{ name: 'notifyTime', order: 'desc' }],
    terms: [
        {
            terms: [
                {
                    column: 'topicProvider',
                    value: getType.value,
                    termType: 'in',
                },
            ],
            type: 'and',
        },
    ],
};
const queryParams = ref({});

const tableRef = ref();

const view = (row: any) => {
    viewItem.value = row;
    viewVisible.value = true;
};
const refresh = () => {
    tableRef.value && tableRef.value.reload();
};

const changeStatus = (row: any) => {
    const type = row.state.value === 'read' ? '_unread' : '_read';
    changeStatus_api(type, [row.id]).then((resp: any) => {
        if (resp.status === 200) {
            onlyMessage('操作成功！');
            refresh();
            user.updateAlarm();
        }
    });
};

watchEffect(() => {
    if (user.messageInfo?.id) {
        view(user.messageInfo);
    }
});

// const onAllRead = async () => {
//     const resp = await changeAllStatus('_read', getType.value);
//     if (resp.status === 200) {
//         onlyMessage('操作成功！');
//         refresh();
//         user.updateAlarm();
//     }
// };

onMounted(() => {
    if (routerParams.params?.value.row) {
        view(routerParams.params?.value.row);
    }
});
onUnmounted(() => {
    user.messageInfo = {}
    viewVisible.value = false
})
</script>

<style lang="less" scoped>
.notification-record-container {
    :deep(.ant-table-tbody) {
        .ant-table-cell {
            .ant-space-item {
                .ant-btn-link {
                    padding: 0;
                }
            }
        }
    }
}
</style>
