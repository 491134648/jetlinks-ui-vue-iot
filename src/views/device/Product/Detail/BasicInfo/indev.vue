<!-- 配置信息 -->
<template>
    <j-descriptions bordered>
        <template #title>
            <div style="display: flex">
                <h3>配置信息</h3>
                <!-- <div style="margin: 0 0px 0 15px; color: #1d39c4">
                        <AIcon type="EditOutlined" @click="editConfig" />
                    </div> -->
                <PermissionButton
                    type="link"
                    @click="editConfig"
                    hasPermission="device/Product:update"
                >
                    <template #icon><AIcon type="EditOutlined" /></template>
                </PermissionButton>
            </div>
        </template>

        <j-descriptions-item label="ID">
            <Ellipsis>{{ productStore.current.id }}</Ellipsis>
        </j-descriptions-item>
        <j-descriptions-item label="产品分类">
            <Ellipsis>{{ productStore.current.classifiedName }}</Ellipsis>
        </j-descriptions-item>
        <j-descriptions-item label="设备类型">{{
            productStore.current.deviceType?.text
        }}</j-descriptions-item>

        <j-descriptions-item label="接入方式">
            <PermissionButton
                type="link"
                @click="changeTables"
                hasPermission="device/Product:update"
                >{{
                    productStore.current.accessName
                        ? productStore.current.accessName
                        : '配置接入方式'
                }}</PermissionButton
            >
        </j-descriptions-item>
        <j-descriptions-item label="创建时间">{{
            dayjs(productStore.current.createTime).format('YYYY-MM-DD HH:mm:ss')
        }}</j-descriptions-item>
        <j-descriptions-item label="更新时间">{{
            dayjs(productStore.current.modifyTime).format('YYYY-MM-DD HH:mm:ss')
        }}</j-descriptions-item>

        <j-descriptions-item label="说明" :span="3">
            {{ productStore.current.describe }}
        </j-descriptions-item>
    </j-descriptions>

    <!-- 编辑 -->
    <Save ref="saveRef" :isAdd="isAdd" :title="title" @success="refresh" />
</template>

<script lang="ts" setup>
import { useProductStore } from '@/store/product';
import Save from '../../Save/index.vue';
import dayjs from 'dayjs';
import { useRoute } from 'vue-router';
import { useMenuStore } from '@/store/menu';
const menuSotre = useMenuStore();
const productStore = useProductStore();
const route = useRoute();
const saveRef = ref();
const isAdd = ref(2);
const title = ref('编辑');
/**
 * 编辑配置信息
 */
const editConfig = () => {
    saveRef.value.show(productStore.current);
};
/**
 * 切换tabs
 */
const changeTables = () => {
    productStore.tabActiveKey = 'Device';
};
/**
 * 修改成功刷新
 */
const refresh = () => {
    productStore.refresh(route.params.id as string);
};
</script>
