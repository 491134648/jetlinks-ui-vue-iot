<template>
    <div class="role">
        <template v-if="type !== 'add'">
            <j-input-search
                allowClear
                @search="onSearch"
                placeholder="请输入名称"
            />
            <div class="role-alert">
                <j-alert type="info">
                    <template #message>
                        <div class="header">
                            <j-checkbox
                                :indeterminate="indeterminate"
                                :checked="checked"
                                @change="onSelectAll"
                                >全选</j-checkbox
                            >
                            <j-space v-if="_selectedRowKeys.length">
                                <span
                                    >已选择{{ _selectedRowKeys.length }}项</span
                                >
                                <j-button
                                    style="padding: 0; height: 22px"
                                    type="link"
                                    @click="cancelSelect"
                                    >取消选择</j-button
                                >
                            </j-space>
                        </div>
                    </template>
                </j-alert>
            </div>
        </template>
        <template v-else>
            <div class="role-alert" style="margin-bottom: 10px;">
                <j-alert type="info">
                    <template #message>
                        <div style="justify-content: space-between; display: flex; align-items: center;">
                            <j-input-search
                                allowClear
                                @search="onSearch"
                                placeholder="请输入名称"
                                style="width: 300px"
                            />
                            <j-space>
                                <template v-if="_selectedRowKeys.length">
                                    <span
                                        >已选择{{
                                            _selectedRowKeys.length
                                        }}项</span
                                    >
                                    <j-button
                                        style="padding: 0; height: 22px"
                                        type="link"
                                        @click="cancelSelect"
                                        >取消选择</j-button
                                    >
                                </template>

                                <j-checkbox
                                    :indeterminate="indeterminate"
                                    :checked="checked"
                                    @change="onSelectAll"
                                    >全选</j-checkbox
                                >
                            </j-space>
                        </div>
                    </template>
                </j-alert>
            </div>
        </template>
        <j-scrollbar height="400px">
            <j-pro-table
                ref="tableRef"
                :columns="columns"
                :request="queryRoleList"
                model="CARD"
                :params="params"
                :bodyStyle="{ padding: 0 }"
                :gridColumn="gridColumn"
                :alertRender="false"
                :defaultParams="{
                    sorts: [{ name: 'createTime', order: 'desc' }],
                }"
                :rowSelection="{
                    selectedRowKeys: _selectedRowKeys,
                }"
            >
                <template #card="slotProps">
                    <div class="card">
                        <j-checkbox
                            :checked="_selectedRowKeys.includes(slotProps?.id)"
                            @change="(e) => onSelect(e, slotProps)"
                        >
                            <j-ellipsis>{{ slotProps.name }}</j-ellipsis>
                        </j-checkbox>
                    </div>
                </template>
            </j-pro-table>
        </j-scrollbar>
    </div>
</template>

<script lang="ts" setup>
import { queryRoleList } from '@/api/system/noticeRule';
import { PropType } from 'vue';

const props = defineProps({
    modelValue: {
        type: Array as PropType<string[]>,
        default: () => [],
    },
    gridColumn: {
        type: Number,
        default: 3,
    },
    type: {
        type: String,
        default: '',
    },
});

const emit = defineEmits(['update:modelValue']);

const params = ref<any>();
const _selectedRowKeys = ref<string[]>([]);
const tableRef = ref();

const dataSource = computed(() => {
    return tableRef.value?._dataSource || [];
});

const indeterminate = computed(() => {
    return (
        dataSource.value.some((item: any) => {
            return _selectedRowKeys.value.includes(item.id);
        }) && !checked.value
    );
});

const checked = computed(() => {
    return dataSource.value.every((item: any) => {
        return _selectedRowKeys.value.includes(item.id);
    });
});

watchEffect(() => {
    _selectedRowKeys.value = props?.modelValue || [];
});

const columns = [
    {
        title: '标识',
        dataIndex: 'id',
        key: 'id',
        ellipsis: true,
        fixed: 'left',
    },
    {
        title: '名称',
        dataIndex: 'name',
        key: 'name',
        ellipsis: true,
        search: {
            type: 'string',
        },
    },
    {
        title: '说明',
        key: 'description',
        ellipsis: true,
        dataIndex: 'description',
    },
];

const onSearch = (val: any) => {
    params.value = {
        terms: [
            {
                column: 'name',
                termType: 'like',
                value: `%${val}%`,
            },
        ],
    };
};

// 取消全选
const cancelSelect = () => {
    _selectedRowKeys.value = [];
    emit('update:modelValue', []);
};

const onSelect = (e: any, record: any) => {
    const _set = new Set(_selectedRowKeys.value);
    const selected = e.target.checked;
    if (selected) {
        _set.add(record.id);
    } else {
        _set.delete(record.id);
    }
    emit('update:modelValue', [..._set.values()]);
};

const onSelectAll = (e: any) => {
    const selected = e.target.checked;
    const _set = new Set(_selectedRowKeys.value);
    const arr = dataSource.value.map((item: any) => item?.id);
    if (selected) {
        arr.map((i: any) => {
            _set.add(i);
        });
    } else {
        arr.map((i: any) => {
            _set.delete(i);
        });
    }
    emit('update:modelValue', [..._set.values()]);
};
</script>

<style lang="less" scoped>
.role {
    :deep(.jtable-content) {
        padding: 2px 0 16px 0;
    }
    :deep(.jtable-card-items) {
        grid-gap: 2px !important;
    }

    .role-alert {
        margin-top: 16px;
    }

    .header {
        display: flex;
    }
}
.card {
    width: 100%;
    background-color: #f8f9fc;
    padding: 10px 16px;
}
</style>