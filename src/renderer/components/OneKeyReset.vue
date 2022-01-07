<template>
    <mu-flex direction="column" style="width: 100%">
        <!-- 提示 -->
        <mu-appbar style="width: 100%; height: 40px;" :title="lang['Tip1']" color="primary" z-depth="0"></mu-appbar>
        <mu-flex style="padding: 30px 50px;">
            <!-- 选择平台 -->
            <mu-select :label="lang['Platform Select']" filterable v-model="platformNameSelected" @change="changePlatformOrEnv">
                <mu-option v-for="(platformName,index) in platformNames" :key="index" :label="platformAliasList[index] + '(' + platformName + ')'" :value="platformName"></mu-option>
            </mu-select>
            <!-- 运行环境 -->
            <mu-select :label="lang['Run Env']" filterable v-model="runEnv" style="margin-left: 10px;" @change="changePlatformOrEnv">
                <mu-option :label="lang['Run Env Inter']" value="InterNet"></mu-option>
                <mu-option :label="lang['Run Env Outer Test']" value="OuterNetTest"></mu-option>
                <mu-option :label="lang['Run Env Outer']" value="OuterNet"></mu-option>
                <mu-option :label="lang['Run Env Personal Test LSD']" value="PersonalTestLSD"></mu-option>
                <mu-option :label="lang['Run Env Personal Test CZW']" value="PersonalTestCZW"></mu-option>
                <mu-option :label="lang['Run Env Personal Test ZXJ']" value="PersonalTestZXJ"></mu-option>
                <mu-option :label="lang['Run Env Personal Test YHH']" value="PersonalTestYHH"></mu-option>
            </mu-select>
            <!-- 当前资源入口 -->
            <mu-text-field :label="lang['Current Resource Entry']" v-model="currentResourceEntry" style="margin-left: 30px;"></mu-text-field>
        </mu-flex>
        <mu-button large full-width color="lime800" @click="clickOneKeyReset">{{lang['One Key Reset']}}</mu-button>
        <!-- 回滚进度框 -->
        <mu-dialog :title="lang['Reset Type']" :open="resetType" :width=500 :overlay-close=false :esc-press-close=false>
            <mu-alert v-if="!resetTypeFinish" color="info">{{lang['Info Running']}}</mu-alert>
            <mu-alert v-else color="success">{{lang['Finish']}}</mu-alert>
            <mu-button slot="actions" flat color="primary" @click="resetType = false">{{lang['Close']}}</mu-button>
        </mu-dialog>
    </mu-flex>
</template>

<script>
    const fs = require('fs');
    const pathMod = require('path');
    const SFTPClient = require('ssh2-sftp-client');

    export default {
        name: "app-oneKeyReset",
        data () {
            return {
                lang: this.i18n.__('One Key Reset'),
                platformNameSelected: "",
                platformNames: [""],
                platformAliasList: [""],
                runEnv: "",
                gmmConfigPlatform: {},
                gmmConfigServer: {},
                currentResourceEntry: "",
                sdk: "WeChat",
                resetType: false,
                resetTypeFinish: false
            }
        },
        mounted () {
            let that = this;
            let projPath = that.$route.params.projPath;
            if (!projPath) {
                that.$toast.error(that.lang['Error Compress JS Path Empty']);
                return;
            }
            that.JStorage.setDataPath(projPath); // 设置数据路径
            // 检测平台配置文件
            that.JStorage.has(that._GmmConfigFilePlatform, function (error, hasKey) {
                if (error) {
                    throw error;
                }
                if (hasKey) {
                    that.JStorage.get(that._GmmConfigFilePlatform, function (error, data) {
                        if (error) {
                            throw error;
                        }
                        if (data) {
                            that.gmmConfigPlatform = that.deepCopyObj(data);
                            for(var platformName of Object.keys(that.gmmConfigPlatform)) {
                                let alias = that.gmmConfigPlatform[platformName].config.alias;
                                alias = alias || platformName;
                                that.platformNames.push(platformName);
                                that.platformAliasList.push(alias);
                            }
                        }
                    })
                }
            })
            // 检测服务器配置文件
            that.JStorage.has(that._GmmServerFile, function (error, hasKey) {
                if (error) {
                    throw error;
                }
                if (hasKey) {
                    that.JStorage.get(that._GmmServerFile, function (error, data) {
                        if (error) throw error;
                        if (data) {
                            let servers = that.deepCopyObj(data).servers;
                            for (var server of servers) {
                                that.gmmConfigServer[server.name] = server;
                            }
                        }
                    })
                }
            })
        },
        methods: {
            /**
             * 改变平台或环境时触发事件
             */
            changePlatformOrEnv: async function() {
                let that = this;
                let currentResourceEntryIndex; // 获取当前资源入口索引
                if(that.platformNameSelected && that.runEnv) {
                    if(that.platformNameSelected.indexOf("App") != -1) {
                        that.currentResourceEntry = "无";
                        return;
                    }
                    currentResourceEntryIndex = await that.getCurrentResourcePathIndex();
                    if(!currentResourceEntryIndex) that.$toast.error(that.lang['Error Get Resource Path']);
                    that.currentResourceEntry = currentResourceEntryIndex;
                } else {
                    that.currentResourceEntry = "";
                }
            },

            /**
             * 获取当前资源路径索引
             * @param return currentResourcePathIndex 当前资源路径索引(A or B or 未知)
             */
            getCurrentResourcePathIndex: async function() {
                let that = this;
                if(!that.platformNameSelected || !that.runEnv) return;
                let platformConfig = that.gmmConfigPlatform[that.platformNameSelected].config; // 获取所选平台的配置信息
                let indexFilePath; // 配置的入口文件路径
                let indexFileName; // 入口文件名
                let projPath = that.$route.params.projPath; // 项目路径
                let localIndexFilePath; // 本地的入口文件路径
                let indexServers; // 入口服务器
                let resourceFilePath_A; // 资源路径A
                let resourceFilePath_B;
                let resourceFolderName_A; // 资源路径A的文件夹名字
                let resourceFolderName_B;
                let currentResourcePathIndex; // 当前资源路径索引(A or B)
                switch(that.runEnv) {
                    case 'InterNet':
                        indexFilePath = platformConfig.indexInner;
                        resourceFilePath_A = platformConfig.serverPathInner;
                        resourceFilePath_B = platformConfig.serverPathInner_B;
                        break;
                    case 'OuterNetTest':
                        indexFilePath = platformConfig.indexOuter;
                        resourceFilePath_A = platformConfig.serverPathOuter;
                        resourceFilePath_B = platformConfig.serverPathOuter_B;
                        break;
                    case 'OuterNet':
                        indexFilePath = platformConfig.index;
                        resourceFilePath_A = platformConfig.serverPath;
                        resourceFilePath_B = platformConfig.serverPath_B;
                        break;
                    case 'PersonalTestLSD':
                        indexFilePath = platformConfig.indexPersonalLSD;
                        resourceFilePath_A = platformConfig.serverPathPersonalLSD;
                        resourceFilePath_B = platformConfig.serverPathPersonalLSD_B;
                        break;
                    case 'PersonalTestCZW':
                        indexFilePath = platformConfig.indexPersonalCZW;
                        resourceFilePath_A = platformConfig.serverPathPersonalCZW;
                        resourceFilePath_B = platformConfig.serverPathPersonalCZW_B;
                        break;
                    case 'PersonalTestZXJ':
                        indexFilePath = platformConfig.indexPersonalZXJ;
                        resourceFilePath_A = platformConfig.serverPathPersonalZXJ;
                        resourceFilePath_B = platformConfig.serverPathPersonalZXJ_B;
                        break;
                    case 'PersonalTestYHH':
                        indexFilePath = platformConfig.indexPersonalYHH;
                        resourceFilePath_A = platformConfig.serverPathPersonalYHH;
                        resourceFilePath_B = platformConfig.serverPathPersonalYHH_B;
                        break;
                    default: return;
                }
                indexFileName = indexFilePath.substring(indexFilePath.lastIndexOf("/") + 1, indexFilePath.length);
                localIndexFilePath = pathMod.join(projPath, indexFileName);
                indexServers = that.getServersByEnvAndSdk(platformConfig.indexServers, that.runEnv, that.sdk);
                if(indexServers.length == 0) {
                    // 提示平台未配置服务器
                    that.$toast.error(that.lang['Error Zero Servers']);
                    return;
                }
                let server = indexServers[0];
                let serverIndexFilePath = "/" + indexFilePath;
                let sftp = new SFTPClient();
                let pm = new Promise((resolve, reject) => {
                    sftp.connect({
                        host: server.host,
                        port: server.port,
                        username: server.user,
                        password: server.password
                    }).then(() => {
                        return sftp.get(serverIndexFilePath, localIndexFilePath); // 下载入口服务器文件到本地路径
                    }).then(() => {
                        let indexFileContent = fs.readFileSync(localIndexFilePath).toString(); // 入口文件内容
                        if(resourceFilePath_A) resourceFolderName_A = resourceFilePath_A.substring(resourceFilePath_A.lastIndexOf("/") + 1, resourceFilePath_A.length) + "/";
                        if(resourceFilePath_B) resourceFolderName_B = resourceFilePath_B.substring(resourceFilePath_B.lastIndexOf("/") + 1, resourceFilePath_B.length) + "/";
                        if(indexFileContent.indexOf(resourceFolderName_A) != -1) {
                            currentResourcePathIndex = "A";
                            console.log("当前所选的资源服务器路径:" + resourceFilePath_A);
                        } else if(indexFileContent.indexOf(resourceFolderName_B) != -1) {
                            console.log("当前所选的资源服务器路径:" + resourceFilePath_B);
                            currentResourcePathIndex = "B";
                        } else {
                            console.log("当前所选的资源服务器路径:" + "未知");
                            currentResourcePathIndex = "未知";
                            that.$toast.error(that.lang['Error Get Resource Path']);
                        }
                        sftp.end();
                        fs.unlinkSync(localIndexFilePath); // 删除本地入口文件
                        resolve(true);
                    }).catch((err) => {
                        sftp.end();
                        console.log("Err: Updating " + server.name);
                        console.log(err);
                        that.$toast.error(that.lang['Error SFTP']);
                        reject(err);
                    })
                })
                await pm;
                return currentResourcePathIndex;
            },

            /**
             * 通过所选环境和sdk获取服务器信息
             */
            getServersByEnvAndSdk: function(serverNames, env, sdk) {
                let that = this;
                let servers = [];
                for(var serverName of serverNames) {
                    let server = that.gmmConfigServer[serverName];
                    if (server.env == env && server.sdk == sdk) servers.push(server);
                }
                return servers;
            },

            /**
             * 一键回滚按钮事件
             */
            clickOneKeyReset: function() {
                let that = this;
                if (that.platformNameSelected.length == 0) {
                    that.$toast.error({message: that.lang['Error Unselect Platform'], time: 3000});
                    return;
                }
                if (that.runEnv.length == 0) {
                    that.$toast.error({message: that.lang['Error Unselect RunEnv'], time: 3000});
                    return;
                }
                let indexFilePath; // 配置的入口文件地址
                let resourceFilePath_A; // 资源路径A
                let resourceFilePath_B;
                let resourceFolderName_A; // 资源路径A的文件夹名字
                let resourceFolderName_B;
                let platformConfig = that.gmmConfigPlatform[that.platformNameSelected].config; // 获取所选平台的配置信息
                switch(that.runEnv) {
                    case 'InterNet':
                        indexFilePath = platformConfig.indexInner;
                        resourceFilePath_A = platformConfig.serverPathInner;
                        resourceFilePath_B = platformConfig.serverPathInner_B;
                        break;
                    case 'OuterNetTest':
                        indexFilePath = platformConfig.indexOuter;
                        resourceFilePath_A = platformConfig.serverPathOuter;
                        resourceFilePath_B = platformConfig.serverPathOuter_B;
                        break;
                    case 'OuterNet':
                        indexFilePath = platformConfig.index;
                        resourceFilePath_A = platformConfig.serverPath;
                        resourceFilePath_B = platformConfig.serverPath_B;
                        break;
                    case 'PersonalTestLSD':
                        indexFilePath = platformConfig.indexPersonalLSD;
                        resourceFilePath_A = platformConfig.serverPathPersonalLSD;
                        resourceFilePath_B = platformConfig.serverPathPersonalLSD_B;
                        break;
                    case 'PersonalTestCZW':
                        indexFilePath = platformConfig.indexPersonalCZW;
                        resourceFilePath_A = platformConfig.serverPathPersonalCZW;
                        resourceFilePath_B = platformConfig.serverPathPersonalCZW_B;
                        break;
                    case 'PersonalTestZXJ':
                        indexFilePath = platformConfig.indexPersonalZXJ;
                        resourceFilePath_A = platformConfig.serverPathPersonalZXJ;
                        resourceFilePath_B = platformConfig.serverPathPersonalZXJ_B;
                        break;
                    case 'PersonalTestYHH':
                        indexFilePath = platformConfig.indexPersonalYHH;
                        resourceFilePath_A = platformConfig.serverPathPersonalYHH;
                        resourceFilePath_B = platformConfig.serverPathPersonalYHH_B;
                        break;
                    default:
                        that.$toast.error({message: that.lang['Error No RunEnv'], time: 3000});
                        console.log(that.runEnv);
                        return;
                }
                if(resourceFilePath_A) resourceFolderName_A = resourceFilePath_A.substring(resourceFilePath_A.lastIndexOf("/") + 1, resourceFilePath_A.length) + "/";
                if(resourceFilePath_B) resourceFolderName_B = resourceFilePath_B.substring(resourceFilePath_B.lastIndexOf("/") + 1, resourceFilePath_B.length) + "/";
                if(!indexFilePath) {
                    that.$toast.error({message: that.lang['Error No Config Entry Path'], time: 3000});
                    return;
                }
                if(!resourceFolderName_A) {
                    that.$toast.error({message: that.lang['Error No Config Resource Path A'], time: 3000});
                    return;
                }
                if(!resourceFolderName_B) {
                    that.$toast.error({message: that.lang['Error No Config Resource Path B'], time: 3000});
                    return;
                }
                that.$confirm(that.lang['Comfirm One Key Reset'], '', {
                    width: 500,
                    type: 'warning'
                }).then(({ result }) => {
                    if (result) {
                        console.log("开始回滚")
                        that.startReset(platformConfig.indexServers, indexFilePath, resourceFolderName_A, resourceFolderName_B);
                    }
                })
            },

            /**
             * 开始回滚(修改入口服务器上的资源路径)
             * @param {string} indexServersConfig 入口服务器配置
             * @param {string} indexFilePath 入口服务器文件地址
             * @param {string} resourceFolderName_A 资源路径A的路径名
             * @param {string} resourceFolderName_B 资源路径B的路径名
             */
            startReset: async function(indexServersConfig, indexFilePath, resourceFolderName_A, resourceFolderName_B) {
                let that = this;
                let projPath = that.$route.params.projPath; // 项目路径
                let indexFileName = indexFilePath.substring(indexFilePath.lastIndexOf("/") + 1, indexFilePath.length); // 入口文件名
                let localIndexFilePath = pathMod.join(projPath, indexFileName); // 本地的入口文件路径
                let indexServers = that.getServersByEnvAndSdk(indexServersConfig, that.runEnv, that.sdk); // 入口服务器
                let unfinishCount = indexServers.length;

                if (unfinishCount == 0) {
                    that.$toast.error(that.lang['Error Zero Servers']);
                    return;
                }
                that.resetType = true;
                for (var server of indexServers) {
                    let serverIndex = "/" + indexFilePath;
                    let sftp = new SFTPClient();
                    let pm = new Promise((resolve, reject) => {
                        sftp.connect({
                            host: server.host,
                            port: server.port,
                            username: server.user,
                            password: server.password
                        }).then(() => { // 下载index.php
                            return sftp.get(serverIndex, localIndexFilePath);
                        }).then(() => { // 更改下载的index.php
                            let content = fs.readFileSync(localIndexFilePath).toString();
                            if(resourceFolderName_A && resourceFolderName_B) {
                                if(content.indexOf(resourceFolderName_A) != -1) {
                                    content = content.replace(new RegExp(resourceFolderName_A, "g"), resourceFolderName_B);
                                    that.currentResourceEntry = "B";
                                    console.log("即将修改入口服务器资源路径索引为:B");
                                } else if(content.indexOf(resourceFolderName_B) != -1) {
                                    content = content.replace(new RegExp(resourceFolderName_B, "g"), resourceFolderName_A);
                                    that.currentResourceEntry = "A";
                                    console.log("即将修改入口服务器资源路径索引为:A");
                                } else {
                                    console.log("配置的资源文件夹A名称:" + resourceFolderName_A + " 配置的资源文件夹B名称:" + resourceFolderName_B);
                                    that.currentResourceEntry = "未知";
                                }
                            }
                            fs.writeFileSync(localIndexFilePath, content);
                        }).then(() => { // 上传修改的index.php
                            return sftp.put(localIndexFilePath, serverIndex);
                        }).then(() => {
                            sftp.end();
                            unfinishCount--;
                            if (unfinishCount == 0) { // 所有服务器更新完毕
                                fs.unlink(localIndexFilePath, () => {});
                                that.resetTypeFinish = true;
                                console.log("本次回滚完成:" + that.getFormatDate());
                            }
                            resolve(true);
                        }).catch((err) => {
                            sftp.end();
                            console.log("Err: Updating " + server.name);
                            console.log(err);
                            that.$toast.error(that.lang['Error SFTP']);
                            reject(err);
                        })
                    })
                    await pm;
                }
            },

            /**
             * 获取格式化的日期
             */
            getFormatDate: function() {
                // 返回 yyyymmddhhmmss
                let date = new Date();
                let year = date.getFullYear();
                let month = date.getMonth() + 1;
                month = month < 10 ? "0" + month : month;
                let day = date.getDate();
                day = day < 10 ? "0" + day : day;
                let hour = date.getHours();
                hour = hour < 10 ? "0" + hour : hour;
                let minute = date.getMinutes();
                minute = minute < 10 ? "0" + minute : minute;
                let second = date.getSeconds();
                second = second < 10 ? "0" + second : second;
                let formatDate = "" + year + month + day + hour + minute + second;
                return formatDate;
            }
        }
    }
</script>