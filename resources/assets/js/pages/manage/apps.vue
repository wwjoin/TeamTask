<template>
    <div class="apps-container">
        <div class="apps-header" v-if="!loading && $route.name == 'manage-apps'">
            <img src="/images/logo.svg" class="apps-logo" alt="logo" />
        </div>
        <MicroApps :url="appUrl" :path="path" v-if="!loading && $route.name == 'manage-apps'" />
    </div>
</template>

<style lang="scss" scoped>
.apps-container {
    .apps-header {
        padding: 16px;
        display: flex;
        align-items: center;
        
        .apps-logo {
            width: 32px;
            height: 32px;
            margin-right: 12px;
        }
    }
}
</style>

<script>
import MicroApps from "../../components/MicroApps.vue";

export default {
    components: { MicroApps },
    data() {
        return {
            loading: false,
            appUrl: '',
            path: '',
        }
    },

    deactivated() {
        this.loading = true;
    },

    watch: {
        '$route': {
            handler(to) {
                this.loading = true;
                if (to.name == 'manage-apps') {
                    this.$nextTick(() => {
                        this.loading = false;
                        this.appUrl = import.meta.env.VITE_OKR_WEB_URL || $A.mainUrl("apps/okr")
                        this.path = this.$route.query.path || '';
                    })
                }else{
                    this.appUrl = '';
                }
            },
            immediate: true
        }
    }
}
</script>
