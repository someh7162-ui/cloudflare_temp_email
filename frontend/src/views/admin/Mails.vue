<script setup>
import { ref } from 'vue';
import { useScopedI18n } from '@/i18n/app'
import { DeleteFilled } from '@vicons/material'

import { useGlobalState } from '../../store'
import { api } from '../../api'
import MailBox from '../../components/MailBox.vue';

const { adminMailTabAddress, loading } = useGlobalState()
const message = useMessage()

const { t } = useScopedI18n('views.admin.Mails')

const mailBoxKey = ref("")

const queryMail = () => {
    adminMailTabAddress.value = adminMailTabAddress.value.trim();
    mailBoxKey.value = Date.now();
}

const fetchMailData = async (limit, offset) => {
    return await api.fetch(
        `/admin/mails`
        + `?limit=${limit}`
        + `&offset=${offset}`
        + (adminMailTabAddress.value ? `&address=${encodeURIComponent(adminMailTabAddress.value)}` : '')
    );
}

const deleteMail = async (curMailId) => {
    await api.fetch(`/admin/mails/${curMailId}`, { method: 'DELETE' });
};

const clearMails = async () => {
    try {
        adminMailTabAddress.value = adminMailTabAddress.value.trim();
        const res = await api.fetch(
            `/admin/mails`
            + (adminMailTabAddress.value ? `?address=${encodeURIComponent(adminMailTabAddress.value)}` : ''),
            { method: 'DELETE' }
        );
        message.success(t('clearMailsSuccess', { count: res.changes ?? 0 }));
        queryMail();
    } catch (error) {
        message.error(error.message || "error");
    }
}
</script>

<template>
    <div style="margin-top: 10px;">
        <n-space align="center">
            <n-input-group class="mail-query-group">
                <n-input v-model:value="adminMailTabAddress" :placeholder="t('addressQueryTip')"
                    @keydown.enter="queryMail" clearable />
                <n-button @click="queryMail" type="primary" tertiary>
                    {{ t('query') }}
                </n-button>
            </n-input-group>
            <n-popconfirm @positive-click="clearMails">
                <template #trigger>
                    <n-button type="error" tertiary :loading="loading">
                        <template #icon>
                            <n-icon :component="DeleteFilled" />
                        </template>
                        {{ adminMailTabAddress ? t('clearAddressMails') : t('clearAllMails') }}
                    </n-button>
                </template>
                {{ adminMailTabAddress ? t('clearAddressMailsTip', { address: adminMailTabAddress }) : t('clearAllMailsTip') }}
            </n-popconfirm>
        </n-space>
        <div style="margin-top: 10px;"></div>
        <MailBox :key="mailBoxKey" :enableUserDeleteEmail="true" :fetchMailData="fetchMailData"
            :deleteMail="deleteMail" :showFilterInput="true" />
    </div>
</template>

<style scoped>
.mail-query-group {
    flex: 1 1 420px;
    max-width: 720px;
}
</style>
