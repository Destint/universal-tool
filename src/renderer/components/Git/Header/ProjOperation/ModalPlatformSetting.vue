<template>
    <mu-dialog class="modal-proj-setting" :title="modalTitle" scrollable :padding="30" :open.sync="data.showPlatformSetting" :overlay-close=false :esc-press-close=false>
        <mu-flex v-if="checkingConfig">{{lang['Msg Checking Proj Config']}}</mu-flex>
        <mu-flex v-else direction="column">
            <!-- 基础配置 -->
            <mu-flex style="width:100%">
                <mu-text-field v-model="platformName" :label="lang['Platform New Name']" :error-text="errorPlatformName" style="margin-right: 15px; width: 180px;"
                    @change="changePlatformName">
                </mu-text-field>
                <mu-text-field v-model="platformAlias" :label="lang['Platform Alias']" style="margin-right: 15px; width: 180px;"></mu-text-field>
                <mu-text-field v-model="platformSuffix" :label="lang['Platform Suffix']" style="margin-right: 30px; width: 250px;"></mu-text-field>
                <mu-flex fill justify-content="end">
                    <mu-select :label="lang['Platform Select']" filterable v-model="platformNameSelected" style="width: 100%; max-width: 256px;"
                        @change="selectPlatform">
                        <mu-option v-for="(platformName,index) in platformNames" :key="index" :label="platformAliasList[index] + '(' + platformName + ')'" :value="platformName"></mu-option>
                    </mu-select>
                </mu-flex>
            </mu-flex>
            <!-- 不打包的第三方库 -->
            <mu-select :label="lang['Third Party No Pack']" filterable multiple chips v-model="thirdPartysNoPackSelected" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(thirdPartyName, index) in thirdPartys" :key="index" :label="thirdPartyName" :value="thirdPartyName"></mu-option>
            </mu-select>
            <!-- 模块选择 -->
            <div v-if="checkingModules">{{lang['Msg Checking Modules']}}</div>
            <mu-select v-else :label="lang['Platform Module Pick']" filterable multiple chips v-model="modulesSelected" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(mod, name) in modules" :key="name" :label="name" :value="name"></mu-option>
            </mu-select>
            <!-- 环境选择 -->
            <mu-select :label="lang['Platform Select Env']" filterable v-model="selectEnv">
                <mu-option :label="lang['Platform Env Inter']" value="InterNet"></mu-option>
                <mu-option :label="lang['Platform Env Outer Test']" value="OuterNetTest"></mu-option>
                <mu-option :label="lang['Platform Env Outer']" value="OuterNet"></mu-option>
                <mu-option :label="lang['Platform Env Personal LSD']" value="PersonalTestLSD"></mu-option>
                <mu-option :label="lang['Platform Env Personal CZW']" value="PersonalTestCZW"></mu-option>
                <mu-option :label="lang['Platform Env Personal ZXJ']" value="PersonalTestZXJ"></mu-option>
                <mu-option :label="lang['Platform Env Personal YHH']" value="PersonalTestYHH"></mu-option>
            </mu-select>
            <!-- 服务器配置 -->
            <!-- CDN完整路径 -->
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNet'" :label="lang['Platform CDN']" v-model="cdnPath" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'InterNet'" :label="lang['Platform CDN Inner Test']" v-model="cdnPathInnerTest" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform CDN Outer Test']" v-model="cdnPathOuterTest" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform CDN Personal Test']" v-model="cdnPathPersonalTestLSD" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform CDN Personal Test']" v-model="cdnPathPersonalTestCZW" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform CDN Personal Test']" v-model="cdnPathPersonalTestZXJ" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform CDN Personal Test']" v-model="cdnPathPersonalTestYHH" full-width></mu-text-field>
            </mu-flex>
            <!-- 入口文件名 -->
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNet'" :label="lang['Platform Index WeChat']" v-model="platformIndex" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'InterNet'" :label="lang['Platform Index Inner']" v-model="platformIndexInner" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform Index Outer']" v-model="platformIndexOuter" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform Index Personal']" v-model="platformIndexPersonalLSD" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform Index Personal']" v-model="platformIndexPersonalCZW" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform Index Personal']" v-model="platformIndexPersonalZXJ" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform Index Personal']" v-model="platformIndexPersonalYHH" prefix="/" full-width></mu-text-field>
            </mu-flex>
            <!-- 入口文件服务器选择 -->
            <mu-select v-show="selectEnv == 'OuterNet'" :label="lang['Platform Server Index Pick Outer']" filterable multiple chips v-model="serverIndexSelectedOuter" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersOuter" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'InterNet'" :label="lang['Platform Server Index Pick Inter']" filterable multiple chips v-model="serverIndexSelectedInter" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersInter" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform Server Index Pick Outer Test']" filterable multiple chips v-model="serverIndexSelectedOuterTest" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersOuterTest" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform Server Index Pick Personal Test']" filterable multiple chips v-model="serverIndexSelectedPersonalTestLSD" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersPersonalTestLSD" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform Server Index Pick Personal Test']" filterable multiple chips v-model="serverIndexSelectedPersonalTestCZW" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersPersonalTestCZW" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform Server Index Pick Personal Test']" filterable multiple chips v-model="serverIndexSelectedPersonalTestZXJ" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersPersonalTestZXJ" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform Server Index Pick Personal Test']" filterable multiple chips v-model="serverIndexSelectedPersonalTestYHH" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in indexServersPersonalTestYHH" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <!-- 代码资源路径 -->
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNet'" :label="lang['Platform Server Path']" v-model="serverPath" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNet'" :label="lang['Platform Server Path B']" v-model="serverPath_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'InterNet'" :label="lang['Platform Server Path Inner Test']" v-model="serverPathInner" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'InterNet'" :label="lang['Platform Server Path Inner Test B']" v-model="serverPathInner_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform Server Path Outer Test']" v-model="serverPathOuter" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform Server Path Outer Test B']" v-model="serverPathOuter_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform Server Path Personal Test']" v-model="serverPathPersonalLSD" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform Server Path Personal Test B']" v-model="serverPathPersonalLSD_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform Server Path Personal Test']" v-model="serverPathPersonalCZW" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform Server Path Personal Test B']" v-model="serverPathPersonalCZW_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform Server Path Personal Test']" v-model="serverPathPersonalZXJ" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform Server Path Personal Test B']" v-model="serverPathPersonalZXJ_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform Server Path Personal Test']" v-model="serverPathPersonalYHH" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <mu-flex style="width:100%">
                <mu-text-field v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform Server Path Personal Test B']" v-model="serverPathPersonalYHH_B" prefix="/" suffix="/" full-width></mu-text-field>
            </mu-flex>
            <!-- 代码资源服务器 -->
            <mu-select v-show="selectEnv == 'OuterNet'" :label="lang['Platform Server Pick Outer']" filterable multiple chips v-model="serverSelectedOuter" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersOuter" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'InterNet'" :label="lang['Platform Server Pick Inter']" filterable multiple chips v-model="serverSelectedInter" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersInter" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'OuterNetTest'" :label="lang['Platform Server Pick Outer Test']" filterable multiple chips v-model="serverSelectedOuterTest" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersOuterTest" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestLSD'" :label="lang['Platform Server Pick Personal Test']" filterable multiple chips v-model="serverSelectedPersonalTestLSD" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersPersonalTestLSD" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestCZW'" :label="lang['Platform Server Pick Personal Test']" filterable multiple chips v-model="serverSelectedPersonalTestCZW" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersPersonalTestCZW" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestZXJ'" :label="lang['Platform Server Pick Personal Test']" filterable multiple chips v-model="serverSelectedPersonalTestZXJ" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersPersonalTestZXJ" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <mu-select v-show="selectEnv == 'PersonalTestYHH'" :label="lang['Platform Server Pick Personal Test']" filterable multiple chips v-model="serverSelectedPersonalTestYHH" full-width :no-data-text="langMsg['Msg No Data']">
                <mu-option v-for="(server, index) in resourceServersPersonalTestYHH" :key="index" :label="server.name" :value="server.name"></mu-option>
            </mu-select>
            <!-- 保存按钮 -->
            <mu-button round full-width color="primary" @click="onConfigPlatform">
                <mu-icon value="bug_report" style="margin-right: 5px"></mu-icon>{{btnConfig}}
            </mu-button>
        </mu-flex>
        <mu-button slot="actions" flat color="primary" @click="closeModal">{{lang['Close']}}</mu-button>
    </mu-dialog>
</template>

<script>
    const fs = require('fs')
    const pathModule = require('path')
    const mkdirp = require('mkdirp')

    export default {
        props: [
            "lang",
            "langMsg",
            "data"
        ],
        data () {
            return {
                platformName: "",
                platformAlias: "",
                platformNameSelected: "",
                platformSuffix: "",
                checkingConfig: false,
                checkingModules: false,
                modulesSelected: [],
                selectEnv: "InterNet",
                errorPlatformName: "",
                modules: {},
                platformNames: [""],
                platformAliasList: [""],
                gmmConfigPlatform: {},
                cdnPath: "",
                cdnPathOuterTest: "",
                cdnPathInnerTest: "",
                cdnPathPersonalTestLSD: "",
                cdnPathPersonalTestCZW: "",
                cdnPathPersonalTestZXJ: "",
                cdnPathPersonalTestYHH: "",
                platformIndex: "",
                platformIndexInner: "",
                platformIndexOuter: "",
                platformIndexPersonalLSD: "",
                platformIndexPersonalCZW: "",
                platformIndexPersonalZXJ: "",
                platformIndexPersonalYHH: "",
                serverPath: "",
                serverPath_B: "",
                serverPathInner: "",
                serverPathInner_B: "",
                serverPathOuter: "",
                serverPathOuter_B: "",
                serverPathPersonalLSD: "",
                serverPathPersonalLSD_B: "",
                serverPathPersonalCZW: "",
                serverPathPersonalCZW_B: "",
                serverPathPersonalZXJ: "",
                serverPathPersonalZXJ_B: "",
                serverPathPersonalYHH: "",
                serverPathPersonalYHH_B: "",
                indexServersOuter: [],
                indexServersInter: [],
                indexServersOuterTest: [],
                indexServersPersonalTestLSD: [],
                indexServersPersonalTestCZW: [],
                indexServersPersonalTestZXJ: [],
                indexServersPersonalTestYHH: [],
                resourceServersOuter: [],
                resourceServersInter: [],
                resourceServersOuterTest: [],
                resourceServersPersonalTestLSD: [],
                resourceServersPersonalTestCZW: [],
                resourceServersPersonalTestZXJ: [],
                resourceServersPersonalTestYHH: [],
                serverSelectedOuter: [],
                serverSelectedInter: [],
                serverSelectedOuterTest: [],
                serverSelectedPersonalTestLSD: [],
                serverSelectedPersonalTestCZW: [],
                serverSelectedPersonalTestZXJ: [],
                serverSelectedPersonalTestYHH: [],
                serverIndexSelectedOuter: [],
                serverIndexSelectedInter: [],
                serverIndexSelectedOuterTest: [],
                serverIndexSelectedPersonalTestLSD: [],
                serverIndexSelectedPersonalTestCZW: [],
                serverIndexSelectedPersonalTestZXJ: [],
                serverIndexSelectedPersonalTestYHH: [],
                thirdPartysNoPackSelected: [],
                thirdPartys: []
            }
        },
        computed: {
            modalTitle: function() {
                if (this.data.type == 'rel') return this.lang['Platform Setting']
                return this.lang['Platform Unknown']
            },
            btnConfig: function() {
                if (this.platformNameSelected.length > 0) return this.lang['Save']
                else return this.lang['Platform Rel Create']
            },
            iconCreateConfig: function() {
                if (this.data.type == 'rel') return 'launch'
                else if (this.data.type == 'dev') return 'bug_report'
                return 'help_outline'
            },
            platformFullName: function() {
                return this.platformName
            }
        },
        watch: {
            data: {
                deep: true,
                handler(val) {
                    this.checkingConfig = true
                    if (val.showPlatformSetting && val.repo) {
                        this.initData()
                        let that = this
                        this.JStorage.setDataPath(val.repo.path)
                        // 模态框弹出时检测平台配置文件
                        this.JStorage.has(this._GmmConfigFilePlatform, function (error, hasKey) {
                            if (error) {
                                that.checkingConfig = false
                                throw error
                            }
                            if (hasKey) {
                                that.JStorage.get(that._GmmConfigFilePlatform, function (error, data) {
                                    if (error) throw error
                                    if (data) {
                                        that.gmmConfigPlatform = that.deepCopyObj(data)
                                        for(var platformName of Object.keys(that.gmmConfigPlatform)) {
                                            let alias = that.gmmConfigPlatform[platformName].config.alias
                                            alias = alias || platformName
                                            that.platformNames.push(platformName)
                                            that.platformAliasList.push(alias)
                                        }
                                        that.checkingConfig = false
                                    }
                                })
                            }
                            else {
                                that.checkingConfig = false
                            }
                        })
                        // 模态框弹出时检测模块设置
                        this.checkingModules = true
                        this.Git(val.repo.path).show([this.data.branch + ":" + this._GmmConfigFileOrigin], (err, result) => {
                            if (err) throw err
                            that.modules = JSON.parse(result)
                            console.log(this.data.branch + ":" + this._GmmConfigFileOrigin)
                            that.checkingModules = false
                        })
                        // 模态框弹出时获取服务器列表
                        this.JStorage.has(this._GmmServerFile, function (error, hasKey) {
                            if (error) {
                                throw error
                            }
                            if (hasKey) {
                                that.JStorage.get(that._GmmServerFile, function (error, data) {
                                    if (error) throw error
                                    if (data) {
                                        let servers = that.deepCopyObj(data).servers
                                        that.resourceServersOuter = []
                                        that.resourceServersInter = []
                                        that.resourceServersOuterTest = []
                                        that.resourceServersPersonalTestLSD = []
                                        that.resourceServersPersonalTestCZW = []
                                        that.resourceServersPersonalTestZXJ = []
                                        that.resourceServersPersonalTestYHH = []
                                        that.indexServersOuter = []
                                        that.indexServersInter = []
                                        that.indexServersOuterTest = []
                                        that.indexServersPersonalTestLSD = []
                                        that.indexServersPersonalTestCZW = []
                                        that.indexServersPersonalTestZXJ = []
                                        that.indexServersPersonalTestYHH = []
                                        for (var server of servers) {
                                            if (server.type == "Resource") {
                                                if (server.env == "OuterNet") that.resourceServersOuter.push(server)
                                                else if (server.env == "InterNet") that.resourceServersInter.push(server)
                                                else if (server.env == "OuterNetTest") that.resourceServersOuterTest.push(server)
                                                else if(server.env == "PersonalTestLSD") that.resourceServersPersonalTestLSD.push(server)
                                                else if(server.env == "PersonalTestCZW") that.resourceServersPersonalTestCZW.push(server)
                                                else if(server.env == "PersonalTestZXJ") that.resourceServersPersonalTestZXJ.push(server)
                                                else if(server.env == "PersonalTestYHH") that.resourceServersPersonalTestYHH.push(server)
                                            } else if (server.type == "Index") {
                                                if (server.env == "OuterNet") that.indexServersOuter.push(server)
                                                else if (server.env == "InterNet") that.indexServersInter.push(server)
                                                else if (server.env == "OuterNetTest") that.indexServersOuterTest.push(server)
                                                else if (server.env == "PersonalTestLSD") that.indexServersPersonalTestLSD.push(server)
                                                else if (server.env == "PersonalTestCZW") that.indexServersPersonalTestCZW.push(server)
                                                else if (server.env == "PersonalTestZXJ") that.indexServersPersonalTestZXJ.push(server)
                                                else if (server.env == "PersonalTestYHH") that.indexServersPersonalTestYHH.push(server)
                                            }
                                        }
                                    }
                                })
                            }
                        })
                    }
                }
            }
        },
        methods: {
            closeModal: function() {
                this.errorPlatformName = ""
                this.$emit('closeModal')
            },
            initData: function() {
                this.selectEnv = "InterNet"
                this.platformName = ""
                this.platformAlias = ""
                this.platformNameSelected = ""
                this.platformSuffix = ""
                this.cdnPath = ""
                this.cdnPathOuterTest = ""
                this.cdnPathPersonalTestLSD = ""
                this.cdnPathPersonalTestCZW = ""
                this.cdnPathPersonalTestZXJ = ""
                this.cdnPathPersonalTestYHH = ""
                this.cdnPathInnerTest = ""
                this.serverPath = ""
                this.serverPath_B = ""
                this.serverPathInner = ""
                this.serverPathInner_B = ""
                this.serverPathOuter = ""
                this.serverPathOuter_B = ""
                this.serverPathPersonalLSD = ""
                this.serverPathPersonalLSD_B = ""
                this.serverPathPersonalCZW = ""
                this.serverPathPersonalCZW_B = ""
                this.serverPathPersonalZXJ = ""
                this.serverPathPersonalZXJ_B = ""
                this.serverPathPersonalYHH = ""
                this.serverPathPersonalYHH_B = ""
                this.modulesSelected = []
                this.modules = {}
                this.platformNames = [""]
                this.platformAliasList = [""]
                this.platformIndex = ""
                this.platformIndexInner = ""
                this.platformIndexOuter = ""
                this.platformIndexPersonalLSD = ""
                this.platformIndexPersonalCZW = ""
                this.platformIndexPersonalZXJ = ""
                this.platformIndexPersonalYHH = ""
                this.serverSelectedOuter = [],
                this.serverSelectedInter = [],
                this.serverSelectedOuterTest = [],
                this.serverSelectedPersonalTestLSD = [],
                this.serverSelectedPersonalTestCZW = [],
                this.serverSelectedPersonalTestZXJ = [],
                this.serverSelectedPersonalTestYHH = [],
                this.serverIndexSelectedOuter = [],
                this.serverIndexSelectedInter = [],
                this.serverIndexSelectedOuterTest = [],
                this.serverIndexSelectedPersonalTestLSD = [],
                this.serverIndexSelectedPersonalTestCZW = [],
                this.serverIndexSelectedPersonalTestZXJ = [],
                this.serverIndexSelectedPersonalTestYHH = [],
                this.thirdPartysNoPackSelected = []
                this.thirdPartys = []
            },
            selectPlatform: function() {
                if (this.platformNameSelected.length > 0) {
                    this.serverSelectedOuter = []
                    this.serverSelectedInter = []
                    this.serverSelectedOuterTest = []
                    this.serverSelectedPersonalTestLSD = []
                    this.serverSelectedPersonalTestCZW = []
                    this.serverSelectedPersonalTestZXJ = []
                    this.serverSelectedPersonalTestYHH = []
                    this.serverIndexSelectedOuter = []
                    this.serverIndexSelectedInter = []
                    this.serverIndexSelectedOuterTest = []
                    this.serverIndexSelectedPersonalTestLSD = []
                    this.serverIndexSelectedPersonalTestCZW = []
                    this.serverIndexSelectedPersonalTestZXJ = []
                    this.serverIndexSelectedPersonalTestYHH = []
                    let platformConfig = this.gmmConfigPlatform[this.platformNameSelected]
                    this.platformAlias = platformConfig.config.alias
                    this.platformSuffix = platformConfig.config.suffix
                    this.cdnPath = platformConfig.config.cdn
                    this.cdnPathOuterTest = platformConfig.config.cdnOuterTest
                    this.cdnPathPersonalTestLSD = platformConfig.config.cdnPersonalTestLSD
                    this.cdnPathPersonalTestCZW = platformConfig.config.cdnPersonalTestCZW
                    this.cdnPathPersonalTestZXJ = platformConfig.config.cdnPersonalTestZXJ
                    this.cdnPathPersonalTestYHH = platformConfig.config.cdnPersonalTestYHH
                    this.cdnPathInnerTest = platformConfig.config.cdnInnerTest
                    this.serverPath = platformConfig.config.serverPath
                    this.serverPath_B = platformConfig.config.serverPath_B
                    this.serverPathInner = platformConfig.config.serverPathInner
                    this.serverPathInner_B = platformConfig.config.serverPathInner_B
                    this.serverPathOuter = platformConfig.config.serverPathOuter
                    this.serverPathOuter_B = platformConfig.config.serverPathOuter_B
                    this.serverPathPersonalLSD = platformConfig.config.serverPathPersonalLSD
                    this.serverPathPersonalLSD_B = platformConfig.config.serverPathPersonalLSD_B
                    this.serverPathPersonalCZW = platformConfig.config.serverPathPersonalCZW
                    this.serverPathPersonalCZW_B = platformConfig.config.serverPathPersonalCZW_B
                    this.serverPathPersonalZXJ = platformConfig.config.serverPathPersonalZXJ
                    this.serverPathPersonalZXJ_B = platformConfig.config.serverPathPersonalZXJ_B
                    this.serverPathPersonalYHH = platformConfig.config.serverPathPersonalYHH
                    this.serverPathPersonalYHH_B = platformConfig.config.serverPathPersonalYHH_B
                    this.modulesSelected = platformConfig.dependencies
                    this.platformIndex = platformConfig.config.index
                    this.platformIndexInner = platformConfig.config.indexInner
                    this.platformIndexOuter = platformConfig.config.indexOuter
                    this.platformIndexPersonalLSD = platformConfig.config.indexPersonalLSD
                    this.platformIndexPersonalCZW = platformConfig.config.indexPersonalCZW
                    this.platformIndexPersonalZXJ = platformConfig.config.indexPersonalZXJ
                    this.platformIndexPersonalYHH = platformConfig.config.indexPersonalYHH
                    for (let server of platformConfig.config.servers) {
                        if (server.indexOf("正式") != -1) this.serverSelectedOuter.push(server);
                        else if (server.indexOf("内测") != -1) this.serverSelectedInter.push(server);
                        else if (server.indexOf("外测") != -1) this.serverSelectedOuterTest.push(server);
                        else if(server.indexOf("本地楼") != -1) this.serverSelectedPersonalTestLSD.push(server);
                        else if(server.indexOf("本地蔡") != -1) this.serverSelectedPersonalTestCZW.push(server);
                        else if(server.indexOf("本地郑") != -1) this.serverSelectedPersonalTestZXJ.push(server);
                        else if(server.indexOf("本地杨") != -1) this.serverSelectedPersonalTestYHH.push(server);
                    }
                    for (let server of platformConfig.config.indexServers) {
                        if (server.indexOf("正式") != -1) this.serverIndexSelectedOuter.push(server);
                        else if (server.indexOf("内测") != -1) this.serverIndexSelectedInter.push(server);
                        else if (server.indexOf("外测") != -1) this.serverIndexSelectedOuterTest.push(server);
                        else if (server.indexOf("本地楼") != -1) this.serverIndexSelectedPersonalTestLSD.push(server);
                        else if (server.indexOf("本地蔡") != -1) this.serverIndexSelectedPersonalTestCZW.push(server);
                        else if (server.indexOf("本地郑") != -1) this.serverIndexSelectedPersonalTestZXJ.push(server);
                        else if (server.indexOf("本地杨") != -1) this.serverIndexSelectedPersonalTestYHH.push(server);
                    }
                    this.thirdPartysNoPackSelected = platformConfig.config.thirdPartyNoPack
                    this.setThirdPartys()
                }
                else {
                    this.platformAlias = ""
                    this.platformSuffix = ""
                    this.cdnPath = ""
                    this.cdnPathOuterTest = ""
                    this.cdnPathInnerTest = ""
                    this.cdnPathPersonalTestLSD = ""
                    this.cdnPathPersonalTestCZW = ""
                    this.cdnPathPersonalTestZXJ = ""
                    this.cdnPathPersonalTestYHH = ""
                    this.serverPath = ""
                    this.serverPath_B = ""
                    this.serverPathInner = ""
                    this.serverPathInner_B = ""
                    this.serverPathOuter = ""
                    this.serverPathOuter_B = ""
                    this.serverPathPersonalLSD = ""
                    this.serverPathPersonalLSD_B = ""
                    this.serverPathPersonalCZW = ""
                    this.serverPathPersonalCZW_B = ""
                    this.serverPathPersonalZXJ = ""
                    this.serverPathPersonalZXJ_B = ""
                    this.serverPathPersonalYHH = ""
                    this.serverPathPersonalYHH_B = ""
                    this.platformIndex = ""
                    this.platformIndexInner = ""
                    this.platformIndexOuter = ""
                    this.platformIndexPersonalLSD = ""
                    this.platformIndexPersonalCZW = ""
                    this.platformIndexPersonalZXJ = ""
                    this.platformIndexPersonalYHH = ""
                    this.modulesSelected = []
                    this.serverSelectedOuter = [],
                    this.serverSelectedInter = [],
                    this.serverSelectedOuterTest = [],
                    this.serverSelectedPersonalTestLSD = [],
                    this.serverSelectedPersonalTestCZW = [],
                    this.serverSelectedPersonalTestZXJ = [],
                    this.serverSelectedPersonalTestYHH = [],
                    this.serverIndexSelectedOuter = []
                    this.serverIndexSelectedInter = []
                    this.serverIndexSelectedOuterTest = []
                    this.serverIndexSelectedPersonalTestLSD = []
                    this.serverIndexSelectedPersonalTestCZW = []
                    this.serverIndexSelectedPersonalTestZXJ = []
                    this.serverIndexSelectedPersonalTestYHH = []
                    this.thirdPartysNoPackSelected = []
                    this.thirdPartys = []
                }
            },
            onConfigPlatform: function(event) {
                if (this.platformName.length == 0 && this.platformNameSelected.length == 0) {
                    this.errorPlatformName = this.lang['Error platform name empty']
                    return
                }
                
                if (this.platformNameSelected.length > 0) {
                    this.gmmConfigSavePlatform()
                }
                else {
                    let that = this
                    let waring = this.lang['Comfirm Module Create'].replace("%branch%", this.data.branch).replace("%platform%", this.platformFullName)
                    this.$confirm(waring, '', {
                        width: 500,
                        type: 'warning'
                    }).then(({ result }) => {
                        if (result) {
                            that.gmmConfigCreatePlatform()
                        }
                    })
                }
            },
            changePlatformName: function() {
                if (this.platformName.length > 0) this.errorPlatformName = ""
            },
            gmmConfigCreatePlatform: function() {
                let that = this
                let name = this.platformName.length > 0 ? this.platformName : this.platformNameSelected
                this.gmmConfigPlatform[name] = {}
                this.gmmConfigPlatform[name].type = this.data.type
                this.gmmConfigPlatform[name].dependencies = this.modulesSelected
                this.gmmConfigPlatform[name].config = {}
                this.gmmConfigPlatform[name].config['alias'] = this.platformAlias
                this.gmmConfigPlatform[name].config['suffix'] = this.platformSuffix
                this.gmmConfigPlatform[name].config['cdn'] = this.cdnPath
                this.gmmConfigPlatform[name].config['cdnOuterTest'] = this.cdnPathOuterTest
                this.gmmConfigPlatform[name].config['cdnInnerTest'] = this.cdnPathInnerTest
                this.gmmConfigPlatform[name].config['cdnPersonalTestLSD'] = this.cdnPathPersonalTestLSD
                this.gmmConfigPlatform[name].config['cdnPersonalTestCZW'] = this.cdnPathPersonalTestCZW
                this.gmmConfigPlatform[name].config['cdnPersonalTestZXJ'] = this.cdnPathPersonalTestZXJ
                this.gmmConfigPlatform[name].config['cdnPersonalTestYHH'] = this.cdnPathPersonalTestYHH
                this.gmmConfigPlatform[name].config['serverPath'] = this.serverPath
                this.gmmConfigPlatform[name].config['serverPath_B'] = this.serverPath_B
                this.gmmConfigPlatform[name].config['serverPathInner'] = this.serverPathInner
                this.gmmConfigPlatform[name].config['serverPathInner_B'] = this.serverPathInner_B
                this.gmmConfigPlatform[name].config['serverPathOuter'] = this.serverPathOuter
                this.gmmConfigPlatform[name].config['serverPathOuter_B'] = this.serverPathOuter_B
                this.gmmConfigPlatform[name].config['serverPathPersonalLSD'] = this.serverPathPersonalLSD
                this.gmmConfigPlatform[name].config['serverPathPersonalLSD_B'] = this.serverPathPersonalLSD_B
                this.gmmConfigPlatform[name].config['serverPathPersonalCZW'] = this.serverPathPersonalCZW
                this.gmmConfigPlatform[name].config['serverPathPersonalCZW_B'] = this.serverPathPersonalCZW_B
                this.gmmConfigPlatform[name].config['serverPathPersonalZXJ'] = this.serverPathPersonalZXJ
                this.gmmConfigPlatform[name].config['serverPathPersonalZXJ_B'] = this.serverPathPersonalZXJ_B
                this.gmmConfigPlatform[name].config['serverPathPersonalYHH'] = this.serverPathPersonalYHH
                this.gmmConfigPlatform[name].config['serverPathPersonalYHH_B'] = this.serverPathPersonalYHH_B
                this.gmmConfigPlatform[name].config['index'] = this.platformIndex
                this.gmmConfigPlatform[name].config['indexInner'] = this.platformIndexInner
                this.gmmConfigPlatform[name].config['indexOuter'] = this.platformIndexOuter
                this.gmmConfigPlatform[name].config['indexPersonalLSD'] = this.platformIndexPersonalLSD
                this.gmmConfigPlatform[name].config['indexPersonalCZW'] = this.platformIndexPersonalCZW
                this.gmmConfigPlatform[name].config['indexPersonalZXJ'] = this.platformIndexPersonalZXJ
                this.gmmConfigPlatform[name].config['indexPersonalYHH'] = this.platformIndexPersonalYHH
                this.gmmConfigPlatform[name].config['servers'] = this.serverSelectedOuter.concat(this.serverSelectedInter.concat(this.serverSelectedOuterTest.concat(this.serverSelectedPersonalTestLSD.concat(this.serverSelectedPersonalTestCZW.concat(this.serverSelectedPersonalTestZXJ.concat(this.serverSelectedPersonalTestYHH))))))
                this.gmmConfigPlatform[name].config['indexServers'] = this.serverIndexSelectedOuter.concat(this.serverIndexSelectedInter.concat(this.serverIndexSelectedOuterTest.concat(this.serverIndexSelectedPersonalTestLSD.concat(this.serverIndexSelectedPersonalTestCZW.concat(this.serverIndexSelectedPersonalTestZXJ.concat(this.serverIndexSelectedPersonalTestYHH))))))
                this.gmmConfigPlatform[name].config['thirdPartyNoPack'] = this.thirdPartysNoPackSelected
                // 生成配置文件
                this.JStorage.setDataPath(this.data.repo.path)
                this.JStorage.set(this._GmmConfigFilePlatform, this.gmmConfigPlatform, function (error) {
                    if (error) {
                        that.$toast.error({message: that.lang['Error'], time: 1000})
                        throw error
                    }
                    else that.$toast.success({message: that.lang['Success'], time: 1000})
                })
            },
            gmmConfigSavePlatform: function() {
                let newName = this.platformName.length > 0
                let name = newName ? this.platformName : this.platformNameSelected
                // 保存数据
                if (newName) this.gmmConfigPlatform[name] = {}
                this.gmmConfigPlatform[name].type = this.data.type
                this.gmmConfigPlatform[name].dependencies = this.modulesSelected
                if (newName) this.gmmConfigPlatform[name].config = {}
                this.gmmConfigPlatform[name].config['alias'] = this.platformAlias
                this.gmmConfigPlatform[name].config['suffix'] = this.platformSuffix
                this.gmmConfigPlatform[name].config['cdn'] = this.cdnPath
                this.gmmConfigPlatform[name].config['cdnOuterTest'] = this.cdnPathOuterTest
                this.gmmConfigPlatform[name].config['cdnInnerTest'] = this.cdnPathInnerTest
                this.gmmConfigPlatform[name].config['cdnPersonalTestLSD'] = this.cdnPathPersonalTestLSD
                this.gmmConfigPlatform[name].config['cdnPersonalTestCZW'] = this.cdnPathPersonalTestCZW
                this.gmmConfigPlatform[name].config['cdnPersonalTestZXJ'] = this.cdnPathPersonalTestZXJ
                this.gmmConfigPlatform[name].config['cdnPersonalTestYHH'] = this.cdnPathPersonalTestYHH
                this.gmmConfigPlatform[name].config['serverPath'] = this.serverPath
                this.gmmConfigPlatform[name].config['serverPath_B'] = this.serverPath_B
                this.gmmConfigPlatform[name].config['serverPathInner'] = this.serverPathInner
                this.gmmConfigPlatform[name].config['serverPathInner_B'] = this.serverPathInner_B
                this.gmmConfigPlatform[name].config['serverPathOuter'] = this.serverPathOuter
                this.gmmConfigPlatform[name].config['serverPathOuter_B'] = this.serverPathOuter_B
                this.gmmConfigPlatform[name].config['serverPathPersonalLSD'] = this.serverPathPersonalLSD
                this.gmmConfigPlatform[name].config['serverPathPersonalLSD_B'] = this.serverPathPersonalLSD_B
                this.gmmConfigPlatform[name].config['serverPathPersonalCZW'] = this.serverPathPersonalCZW
                this.gmmConfigPlatform[name].config['serverPathPersonalCZW_B'] = this.serverPathPersonalCZW_B
                this.gmmConfigPlatform[name].config['serverPathPersonalZXJ'] = this.serverPathPersonalZXJ
                this.gmmConfigPlatform[name].config['serverPathPersonalZXJ_B'] = this.serverPathPersonalZXJ_B
                this.gmmConfigPlatform[name].config['serverPathPersonalYHH'] = this.serverPathPersonalYHH
                this.gmmConfigPlatform[name].config['serverPathPersonalYHH_B'] = this.serverPathPersonalYHH_B
                this.gmmConfigPlatform[name].config['index'] = this.platformIndex
                this.gmmConfigPlatform[name].config['indexInner'] = this.platformIndexInner
                this.gmmConfigPlatform[name].config['indexOuter'] = this.platformIndexOuter
                this.gmmConfigPlatform[name].config['indexPersonalLSD'] = this.platformIndexPersonalLSD
                this.gmmConfigPlatform[name].config['indexPersonalCZW'] = this.platformIndexPersonalCZW
                this.gmmConfigPlatform[name].config['indexPersonalZXJ'] = this.platformIndexPersonalZXJ
                this.gmmConfigPlatform[name].config['indexPersonalYHH'] = this.platformIndexPersonalYHH
                this.gmmConfigPlatform[name].config['servers'] = this.serverSelectedOuter.concat(this.serverSelectedInter.concat(this.serverSelectedOuterTest.concat(this.serverSelectedPersonalTestLSD.concat(this.serverSelectedPersonalTestCZW.concat(this.serverSelectedPersonalTestZXJ.concat(this.serverSelectedPersonalTestYHH))))))
                this.gmmConfigPlatform[name].config['indexServers'] = this.serverIndexSelectedOuter.concat(this.serverIndexSelectedInter.concat(this.serverIndexSelectedOuterTest.concat(this.serverIndexSelectedPersonalTestLSD.concat(this.serverIndexSelectedPersonalTestCZW.concat(this.serverIndexSelectedPersonalTestZXJ.concat(this.serverIndexSelectedPersonalTestYHH))))))
                this.gmmConfigPlatform[name].config['thirdPartyNoPack'] = this.thirdPartysNoPackSelected
                // 更换平台昵称
                let index = this.platformNames.indexOf(this.platformNameSelected)
                this.$set(this.platformAliasList, index, this.platformAlias || name)
                // 更换平台名
                if (newName) {
                    delete this.gmmConfigPlatform[this.platformNameSelected]

                    // 更新平台列表
                    this.$set(this.platformNames, index, name)
                    this.platformNameSelected = name
                }
                // 保存文件
                let that = this
                this.JStorage.setDataPath(this.data.repo.path)
                this.JStorage.set(this._GmmConfigFilePlatform, this.gmmConfigPlatform, function (error) {
                    if (error) throw error
                    that.$toast.success({message: that.lang['Success'], time: 1000})
                })
            },
            gmmConfigGetRequiredModules: function() {
                let mods = []
                for (var modName of this.modulesSelected) { // 选中的模块
                    if (mods.indexOf(modName) < 0) mods.push(modName)
                }
                for (var modName of mods) { // 选中模块的依赖
                    for (var parentName of this.modules[modName].dependencies) {
                        if (mods.indexOf(parentName) < 0) mods.push(parentName)
                    }
                }
                return mods
            },
            setThirdPartys:function()
            {
                let projPath = this.data.repo.path
                if (projPath) {
                    // 根据 bin/indexGmm.html 获取第三方库模块
                    let fileIndex = pathModule.join(projPath, "bin/index.html")
                    if (fs.existsSync(fileIndex)) {
                        // 开始处理模块
                        let content = fs.readFileSync(fileIndex).toString().split("\n")
                        // 获取各模块文件名
                        let thirdPartyFileNames = []
                        let modName = ""
                        let isComment = false
                        let isThirdParty = false
                        for (var line of content) {
                            line = line.trim()
                            if (isComment) {
                                if (line.indexOf("-->") > -1) isComment = false
                            }
                            else if (line.indexOf("<!--") == 0) {
                                if (line.indexOf("Mod:") > -1) { // 模块标签
                                    let index = line.indexOf("Mod:")
                                    if (modName.length == 0) {
                                        if (line.indexOf(":start", index + 4) > -1) { // 模块开头
                                            let index2 = line.indexOf(":start", index + 4)
                                            modName = line.substring(index + 4, index2).trim()
                                        }
                                    }
                                    else if (modName.length > 0 && line.indexOf(":end", index + 4) > -1) { // 非平台依赖模块结尾
                                        modName = ""
                                    }
                                }
                                else if (line.indexOf("-->") == -1) {
                                    isComment = true
                                }
                            }
                            else if (line.indexOf("<script ") == 0) {
                                if(modName == "third_party")
                                {
                                    // 案例：script src="libs/laya.core.js?v=1.0.11" ><\/script>
                                    // 第一次截取：src="libs/laya.core.js?v=1.0.11" >
                                    let startIndex = line.indexOf("script ")
                                    let endIndex = line.lastIndexOf("<\/script>")
                                    let file = line.substring(startIndex + 'script '.length, endIndex)
                                    // 第二次截取：libs/laya.core.js?v=1.0.11" 
                                    startIndex = file.indexOf('src="')
                                    endIndex = file.lastIndexOf('>')
                                    file = file.substring(startIndex + 'src="'.length, endIndex).trim()
                                    // 第三次截取：libs/laya.core.js
                                    endIndex = file.lastIndexOf("?")
                                    if (endIndex == -1) endIndex = file.lastIndexOf('"')
                                    file = file.substring(0, endIndex)
                                    // 获取文件名
                                    endIndex = file.lastIndexOf('/')
                                    let fileName = file.substr(endIndex + 1)
                                    thirdPartyFileNames.push(fileName) // 非模块代码
                                }
                            }
                        }
                        this.thirdPartys = thirdPartyFileNames;
                    }
                }
            },
        }
    }
</script>
