<template>
    <div>
        <Loader v-if="loading"/>
    <div class="app-main-layout" v-else>
        <Navbar @openCloseSidebar="isOpen = !isOpen"/>
        <Sidebar :isOpen="isOpen"/>

        <main class="app-content" :class="{full: !isOpen}">
            <div class="app-page">
                <router-view/>
            </div>
        </main>

        <div class="fixed-action-btn">
            <router-link
                    class="btn-floating btn-large blue"
                    to="/record"
                    v-tooltip="'Создать новую запись'"
                    data-position="left"
            >
                <i class="large material-icons">add</i>
            </router-link>
        </div>
    </div>
    </div>
</template>

<script>
    import Navbar from "@/components/app/Navbar";
    import Sidebar from "@/components/app/Sidebar";
    import Loader from "@/components/app/Loader";
    export default {
        name: 'main-layout',
        components:{Loader, Navbar, Sidebar },
        data: ()=> ({
            isOpen: true,
            loading: true
        }),
        async mounted() {
            if(!Object.keys(this.$store.getters.info).length){
               await this.$store.dispatch('fetchInfo')
            }
            this.loading = false
        }
    }
</script>

<style scoped>

</style>