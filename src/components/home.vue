<template>
        <el-container style="height:100%">
            <el-header style="padding:0px; background-color:#444444">
                <div style="width:80px; height:60px; float:left; padding-left:50px; line-height:60px; vertical-align:middle; color:white; text-align:left">LOGO</div>i
                <div style="width:300px; height:56px; float:left; line-height:56px; vertical-align:middle; color:white; text-align:left; font-size:20px">管理平台</div>
                <div style="width:300px; height:60px; float:right; color:white; line-height:60px; vertical-align:middle; text-align:right; padding-right:60px">
                    <span>欢迎您</span><span style="margin-left:10px">{{username}}</span>
                    <span style="margin-left:20px"><a @click="doLogout()" href="#" class="logout">注销</a></span>
                </div>
            </el-header>
            <el-container>
                <el-aside width="200px" style="background-color:#444444; text-align:left; padding:0px; margin:0px">
                    <el-menu style="border-right:0px"
                             default-active="2"
                             class="el-menu-vertical-demo"
                             background-color="#444444"
                             text-color="#fff"
                             active-text-color="#ffd04b">
                        <el-submenu v-for="menu in menuList" :key="menu.menuCode" :index="menu.menuCode">
                            <template slot="title">
                                <i class="el-icon-location"></i>
                                <span>{{menu.menuShowName}}</span>
                            </template>
                            <el-menu-item-group v-if="menu.children && menu.children.length > 0">
                                <el-menu-item v-for="childMenu in menu.children"
                                              :key="childMenu.menuCode"
                                              :index="childMenu.menuCode" @click="addTab(childMenu.menuShowName, childMenu.menuComponent)">
                                    {{childMenu.menuShowName}}
                                </el-menu-item>
                            </el-menu-item-group>
                        </el-submenu>
                    </el-menu>
                </el-aside>
                <el-container>
                    <el-main style="padding:5px">
                        <el-tabs v-model="editableTabsValue" type="border-card" editable @tab-remove="removeTab">
                            <el-tab-pane
                                    :key="item.name"
                                    v-for="(item) in editableTabs"
                                    :label="item.title"
                                    :name="item.name"
                            >
                                <component :is=item.content></component>
                            </el-tab-pane>
                        </el-tabs>
                    </el-main>
                </el-container>
            </el-container>
        </el-container>

</template>

<script>
    import firstPage from '@/components/modules/firstPage.vue';

    import * as cookieApi from "@/js/utils/cookie.js";
    import * as userApi from "@/js/api/user.js";

    export default {
        name: "home",
        components: {
            firstPage:()=>import('@/components/modules/firstPage.vue'),
            org:()=>import('@/components/modules/system/org.vue'),
            menua:()=>import('@/components/modules/system/menua.vue'),
            role:()=>import('@/components/modules/system/role.vue'),
            user:()=>import('@/components/modules/system/user.vue'),
            sysconfig:()=>import('@/components/modules/system/config.vue'),
            dictType:()=>import('@/components/modules/system/dictType.vue'),
            dictEntry:()=>import('@/components/modules/system/dictEntry.vue'),
            job:()=>import('@/components/modules/system/job.vue')
        },
        data() {
            const userData = userApi.getUserData();
            return {
                editableTabsValue: 'firstPage',
                editableTabs: [{
                    title: '首页',
                    name: 'firstPage',
                    content: 'firstPage'
                }],
                tabIndex: 1,
                username:userData.userFullName,
                menuList:userData.menuList
            }
        },
        methods: {
            addTab(menuName, menuComponent) {
                let thisTab = this.getTabByName(menuName);
                if (thisTab != null) {
                    this.editableTabsValue = thisTab.name;
                    return;
                }
                let newTabName = ++this.tabIndex + '';
                this.editableTabs.push({
                    title: menuName,
                    name: newTabName,
                    content: menuComponent
                });
                this.editableTabsValue = newTabName;
            },
            removeTab(targetName) {
                if (targetName == "firstPage") {
                    return false;
                }
                let tabs = this.editableTabs;
                let activeName = this.editableTabsValue;
                if (activeName === targetName) {
                    tabs.forEach((tab, index) => {
                        if (tab.name === targetName) {
                            let nextTab = tabs[index + 1] || tabs[index - 1];
                            if (nextTab) {
                                activeName = nextTab.name;
                            }
                        }
                    });
                }

                this.editableTabsValue = activeName;
                this.editableTabs = tabs.filter(tab => tab.name !== targetName);
            },
            getTabByName(tabName) {
                let ret = null;
                this.editableTabs.forEach((tab, index) => {
                    if (tab.title == tabName) {
                        ret = tab;
                    }
                });
                return ret;

            },
            doLogout() {
                sessionStorage.removeItem("userData");
                sessionStorage.removeItem("accessToken");
                localStorage.removeItem("userData");
                localStorage.removeItem("accessToken");
                cookieApi.delCookie("accessToken");

                this.$router.push("/login");
            }
        }
    }
</script>

<style scoped>
    .logout{
        color:white;
        text-decoration:none;
    }
</style>
