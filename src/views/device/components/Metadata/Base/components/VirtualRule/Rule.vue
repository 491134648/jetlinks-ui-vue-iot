<template>
    <j-button @click="visible = true" style="width: 100%" type="dashed">
        编辑规则
    </j-button>
    <FRuleEditor :aggList="aggList" @close="onClose" v-if="visible" :value="value" @save="onChange" :id="id" :virtualRule="virtualRule" />
</template>

<script setup lang="ts" name="Rule">
import FRuleEditor from '@/components/FRuleEditor/index.vue';
import {Form} from "jetlinks-ui-components";

const formItemContext = Form.useInjectFormItemContext();

interface Emits {
    (e: 'update:value', data: string | undefined): void;
    (e: 'change', data: string | undefined): void;
}
const emit = defineEmits<Emits>();

const props = defineProps({
    value: String,
    id: String,
    virtualRule: Object,
    aggList: Array
});

const visible = ref<boolean>(false);

const onChange = (val: string | undefined) => {
    emit('change', val)
    emit('update:value', val)
    formItemContext.onFieldChange()
    onClose()
}

const onClose = () => {
    visible.value = false
}
</script>