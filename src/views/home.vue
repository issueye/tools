<template>
    <div class="home-warp">
        <el-container>
            <el-header>
                <div class="header-title">日志采集工具</div>
                <!-- 用户名下拉菜单 -->
                <div class="user-name">
                    <el-dropdown trigger="click" @command="handleCommand">
                        <span class="el-dropdown-link">
                            admin
                        </span>
                        <template #dropdown>
                            <el-dropdown-menu>
                                <el-dropdown-item divided command="loginout">退出登录</el-dropdown-item>
                            </el-dropdown-menu>
                        </template>
                    </el-dropdown>
                </div>
            </el-header>
            <el-container class="main-container">
                <el-aside width="202px" class="sidebar">
                    <el-menu :router='true' default-active="2" class="el-menu-vertical-demo" background-color="#b0d5df"
                        text-color="#4c1f24" active-text-color="#ccccd6">
                        <el-submenu index="1">
                            <el-menu-item index="/param">
                                <i class="el-icon-menu"></i>
                                <template #title>
                                    参数设置
                                </template>
                            </el-menu-item>
                            <el-menu-item index="/monitor">
                                <i class="el-icon-menu"></i>
                                <template #title>
                                    采集管理
                                </template>
                            </el-menu-item>
                        </el-submenu>
                    </el-menu>
                </el-aside>
                <el-main class="content-box">
                    <router-view v-slot="{ Component }">
                        <keep-alive>
                            <component :is="Component"></component>
                        </keep-alive>
                    </router-view>
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script setup name="home">
import router from '../router/router';

const handleCommand = (e) => {
    if (e == 'loginout') {
        localStorage.clear()
        router.push('/login')
    }
}
</script>

<style>
.home-warp {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.main-container {
    height: 100%;
}

.el-menu-item.is-active {
    background-color: #5cb3cc !important;
}

.user-name {
    display: flex;
    align-items: center;
}

.el-dropdown-link {
    color: #fff;
    cursor: pointer;
    display: flex;
    align-items: center;
}

.content-box {
    position: absolute;
    left: 200px;
    right: 0;
    top: 40px;
    bottom: 0;
    padding-bottom: 30px;
    background: #f0f0f0;
}

.el-header {
    background-color: #1a94bc;
    display: flex;
    justify-content: space-between;

    --el-header-height: 40px !important;
}

.header-title {
    color: #fff;
    font-size: 20px;
    display: flex;
    align-items: center;
}

.sidebar {
    display: block;
    position: absolute;
    left: 0;
    top: 40px;
    bottom: 0;
    background: #b0d5df;
}

.sidebar::-webkit-scrollbar {
    width: 0;
}
</style>
