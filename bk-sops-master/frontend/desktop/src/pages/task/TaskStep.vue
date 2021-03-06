/**
* Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community
* Edition) available.
* Copyright (C) 2017-2019 THL A29 Limited, a Tencent company. All rights reserved.
* Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
* http://opensource.org/licenses/MIT
* Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
* an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
* specific language governing permissions and limitations under the License.
*/
<template>
    <div :class="['step-wrapper',{ 'hidden-step-wrapper': hiddenBorder }]">
        <div class="step-header">
            <div class="step-section-title">
                <span v-if="isShowBackBtn" class="bk-button bk-button-default" @click.prevent="getHomeUrl()">{{ i18n.return }}</span>
                <span class="task-title">{{ taskTemplateTitle }}</span>
                <span class="task-name">{{ instanceName }}</span>
            </div>
        </div>
        <div class="division-line"></div>
        <div class="step-list">
            <div
                :class="{
                    'step-item': true,
                    'step-item-first': index === 0,
                    'finished': allFinished || index < currentStepIndex,
                    'actived': !allFinished && index === currentStepIndex
                }"
                v-for="(item, index) in list"
                :key="index"
                :style="calChartSize(index)">
                <div class="step-graph">
                    <div class="line" v-if="index !== list.length - 1"></div>
                    <div class="common-icon-done-thin step-done" v-if="allFinished || index < currentStepIndex"></div>
                    <div class="order" v-else>{{index + 1}}</div>
                </div>
                <span class="step-name" :style="calNameSize(item.name)">{{item.name}}</span>
            </div>
        </div>
    </div>
</template>
<script>
    import '@/utils/i18n.js'
    import { mapState } from 'vuex'
    export default {
        name: 'TaskCreateStep',
        props: ['list', 'currentStep', 'allFinished', 'common', 'instanceName', 'cc_id', 'taskStatus'],
        data () {
            return {
                i18n: {
                    newTask: gettext('新建任务'),
                    taskExecution: gettext('任务执行'),
                    return: gettext('返回')
                }
            }
        },
        computed: {
            ...mapState({
                'lang': state => state.lang,
                userType: state => state.userType,
                view_mode: state => state.view_mode
            }),
            currentStepIndex () {
                return this.getCurrentStepIndex()
            },
            hiddenBorder () {
                return this.getCurrentStepIndex() === this.list.length - 1 && this.list.length > 2
            },
            taskTemplateTitle () {
                return this.$route.query.instance_id === undefined ? this.i18n.newTask : this.i18n.taskExecution
            },
            isShowBackBtn () {
                return !(this.view_mode === 'appmaker' && this.$route.path.indexOf('newtask') !== -1)
            }
        },
        methods: {
            getCurrentStepIndex () {
                let currentStepIndex = 0
                this.list.some((item, index) => {
                    if (item.step === this.currentStep) {
                        currentStepIndex = index
                        return true
                    }
                })
                return currentStepIndex
            },
            calChartSize (index) {
                const style = {}
                const pencent = (100.0 / (this.list.length - 1)).toFixed(2) + '%'
                if (index !== this.list.length - 1) {
                    style['flex-basis'] = pencent
                } else {
                    style['max-length'] = pencent
                    style['line-height'] = '22px'
                    style['flex-basis'] = 'auto !important'
                    style['flex-shrink'] = 0
                    style['flex-grow'] = 0
                }
                return style
            },
            calNameSize (name) {
                const style = {}
                if (this.lang === 'en') {
                    // 减去一个空格
                    const nameLength = name.length - 1
                    style['left'] = nameLength * -2 + 'px'
                }
                return style
            },
            /**
             * 返回任务列表
             */
            getHomeUrl () {
                let url = '/'
                const userType = this.userType // 用户类型
                const path = this.$route.fullPath
                const entrance = this.$route.query.entrance || '' // 入口参数
                const ccId = this.$route.params.cc_id
                const actions = [
                    // 编辑模板
                    { userType: 'maintainer', path: '/template/edit/', url: `/periodic/home/${this.cc_id}/` },
                    // 任务记录
                    { userType: 'maintainer', path: '/taskflow/execute/', url: `/taskflow/home/${this.cc_id}/` },
                    // 新建-周期任务
                    { userType: 'maintainer', path: '/template/newtask/', entrance: '0', url: `/periodic/home/${this.cc_id}/` },
                    // 新建-任务记录
                    { userType: 'maintainer', path: '/template/newtask/', entrance: '1', url: `/taskflow/home/${this.cc_id}/` },
                    // 新建-业务流程
                    { userType: 'maintainer', path: '/template/newtask/', entrance: 'commonList', url: `/template/common/${this.cc_id}/` },
                    // 新建-公共流程
                    { userType: 'maintainer', path: '/template/newtask/', entrance: 'businessList', url: `/template/home/${this.cc_id}/` },
                    // maintainer 默认
                    { userType: 'maintainer', path: 'default', url: `/template/home/${this.cc_id}/` },
                    // 职能化
                    { userType: 'functor', url: `/function/home/` },
                    // 审计员
                    { userType: 'auditor', url: `/audit/home/` },
                    // 轻应用
                    { view_mode: 'appmaker', url: `/appmaker/${this.$route.params.app_id}/task_home/${ccId}/` }
                ]
                actions.forEach(key => {
                    const flag_view_mode = key.view_mode ? key.view_mode === this.view_mode : true
                    const flag_userType = key.userType ? key.userType === userType : true
                    const flag_path = key.path ? new RegExp(key.path).test(path) : true
                    const flag_entrance = key.entrance ? key.entrance === entrance : true
                    if (flag_view_mode && flag_userType && flag_path && flag_entrance) url = key.url
                })
                if (this.common && userType === 'maintainer') url += `?common=1&common_template=${this.common}`
                this.$router.push(url)
            }
        }
    }
</script>
<style lang="scss" scoped>
@import '@/scss/config.scss';
.step-wrapper {
    background: #f4f7fa;
    border-bottom: 1px solid #cacedb;
    .step-header {
        background-color: #f4f7fa;
        .bk-button-default {
            float: right;
            position: relative;
            top: 16px;
            right: 20px;
            width: 90px
        }
    }
    .step-section-title {
        height: 67px;
        margin: 0;
        color: #313238;
        line-height: 67px;
        text-align: left;
    }
    .task-title {
        padding-left: 30px;
        font-size: 14px;
        font-weight: 600;
        &:before {
            content: '';
            display: inline-block;
            position: relative;
            top: 4px;
            right: 10px;
            width: 2px;
            height: 20px;
            background: #a3c5fd;
        }
    }
    .task-name {
        font-size: 14px;
        font-weight: 600;
    }
    .division-line {
        margin: 0 20px 10px 20px;
        border: 0;
        height: 1px;
        background-color: #dde4eB;
    }
    .step-list {
        display: flex;
        margin: 0 0 16px 0;
        min-width: 1320px;
            
    }
    .step-item {
        display: inline-block;
        height: 44px;
        color: $greyDark;
        font-size: 11px;
        font-weight: 400;
        line-height: 16px;
        text-align: left;
        width: 90px;
        &:last-child {
            .order {
                margin-top: 4px;
            }
            .step-name {
                top: 0px;
                white-space: nowrap;
            }
        }
        .step-done {
            display: inline-block;
            width: 18px;
            height: 18px;
            font-size: 12px;
            font-weight: 400;
            color: #3a84ff;
            border: 2px solid #3a84ff;
            border-radius: 50%;
            background: #3a84ff;
        }
        .order {
            position: relative;
            width: 18px;
            height: 18px;
            line-height: 18px;
            border-radius: 50%;
            font-size: 12px;
            color: #ffffff;
            background-color: #e1e4e8;
            text-align: center;
            vertical-align: middle;
            z-index: 3;
        }
        .step-name {
            position: relative;
            width: 100px;
            top: 5px;
            left: -15px;
            font-size: 12px;
            line-height:17px;
            color: #313238;
            text-align: left;
        }
        .line {
            position: relative;
            top: 10px;
            margin-left: 1px;
            margin-right: 0px;
            width: calc(100% + 1px);
            height: 4px;
            display: block;
            background-color: #e1e4e8;
            z-index: 1;
        }
        &.actived {
            color: $blueDefault;
            .order {
                line-height: 14px;
                color: #3a84ff;
                border: 2px solid $blueDefault;
                background-color: #ffffff;
            }
            .name {
                color: #3a84ff;
            }
        }
        &.finished {
            color: $blueDefault;
            .name {
                color: #3a84ff;
            }
            .line {
                background-color: #3a84ff;
                z-index: 2;
            }
        }
    }
    .step-item-first {
        margin-left: 55px;
    }
}
.hidden-step-wrapper {
    border: 0px;
}
</style>
