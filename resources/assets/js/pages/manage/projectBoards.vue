<template>
    <div class="page-file">
        <PageTitle :title="$L('项目')"/>
        <div class="file-wrapper" ref="fileWrapper">
            <div class="file-head">
                <div class="file-nav">
                    <div class="common-nav-back portrait" @click="goForward({name: 'manage-application'},true)"><i class="taskfont">&#xe676;</i></div>
                    <h1>{{$L('项目')}}</h1>
                    <div v-if="loadIng == 0" class="file-refresh" @click="getProjectList"><i class="taskfont">&#xe6ae;</i></div>
                </div>
                <div class="file-add">
                    <Button class="manage-box-new" type="primary" icon="md-add" @click="onAddProjectShow">{{$L('新建项目')}}</Button>
                </div>
            </div>

            <div class="file-navigator">
                <ul class="scrollbar-hidden" v-show="showBtnText || (!selectIds.length && !shearFirst)">
                    <li @click="browseFolder(0)">
                        <span>{{$L('进行中项目列表')}}</span>
                    </li>
                </ul>
            </div>

            <div class="mt-8 grid grid-cols-1 gap-4 sm:grid-cols-2 sm:gap-5 lg:grid-cols-3 lg:gap-6 xl:grid-cols-4" style="padding: 16px 32px"
            >
                <div class="card shadow-none" v-for="(item, index) in list" :key="index" @click="toggleRoute('project', {projectId: item.id})">
                    <div class="flex flex-1 flex-col justify-between rounded-lg  p-4 dark:bg-transparent sm:p-5" style="border: 1px solid rgb(243, 243, 243);"
                    >
                        <div>
                            <div class="flex items-start justify-between">
                                <div style="display:flex;flex-direction: column">
                                    <h3 class="mt-3 font-medium text-slate-700 line-clamp-2 dark:text-navy-100">{{item.name}}</h3>
                                    <p class="text-xs+ text-ellipsis-2"> {{item.desc}}</p>
                                </div>
<!--                                <img-->
<!--                                    class="h-12 w-12 rounded-lg object-cover object-center"-->
<!--                                    :src="$A.mainUrl('images/200x200.png')"-->
<!--                                    alt="image"-->
<!--                                />-->
                                <p class="text-xs+" style="min-width: 40px">{{formatDate(item.created_at)}}</p>
                            </div>

                        </div>
                        <div>
                            <div class="mt-4">
                                <p class="text-xs+ text-slate-700 dark:text-navy-100">进度</p>
                                    <Progress :percent="item.task_percent" :stroke-width="6" />
                            </div>
                            <div class="mt-1 flex flex-wrap -space-x-3">
                                <ul class="project-icons" style="display: flex">
<!--                                    <li class="project-avatar" :class="{'cursor-default': projectData.owner_userid !== userId}" @click="projectDropdown('user')">-->
<!--                                    <li class="project-avatar" :class="{'cursor-default': item.projectData.owner_userid !== userId}">-->
                                    <li class="project-avatar">
                                        <ul style="display: flex;align-items: center;">
                                            <UserAvatarTip :userid="item.projectData?.owner_userid" :size="36" :borderWitdh="2" :openDelay="0">
                                               <p>{{$L('项目负责人')}}</p>
                                            </UserAvatarTip>
                                            <li v-if="item.projectData?.project_user?.length > 0" v-for="item2 in item.projectData?.project_user">
                                                <UserAvatarTip :userid="item2.userid" :size="36" :borderWitdh="2" :openDelay="0" v-if="item2.userid!==item.projectData?.owner_userid"/>
                                            </li>
                                        </ul>
                                    </li>
                                </ul>
                            </div>

                            <div class="mt-4 flex items-center justify-between space-x-2">
                                <div class="badge h-5.5 rounded-full bg-info px-2 text-xs+ text-white"> {{calculateTimeAgo(item.created_at)}}</div>
                                <div>
                                    <button class="btn -mr-1.5 h-8 w-8 rounded-full p-0 hover:bg-slate-300/20 focus:bg-slate-300/20 active:bg-slate-300/25 dark:hover:bg-navy-300/20 dark:focus:bg-navy-300/20 dark:active:bg-navy-300/25"
                                            style="display: none" >
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            class="h-5 w-5"
                                            fill="none"
                                            viewBox="0 0 24 24"
                                            stroke="currentColor"
                                            stroke-width="1.5"
                                        >
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
                                            />
                                            <path
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                                d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                                            />
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!--新建项目-->
        <Modal
            v-model="addShow"
            :title="$L('新建项目')"
            :mask-closable="false">
            <Form
                ref="addProject"
                :model="addData"
                :rules="addRule"
                v-bind="formOptions"
                @submit.native.prevent>
                <FormItem prop="name" :label="$L('项目名称')">
                    <Input ref="projectName" type="text" v-model="addData.name" @on-keydown="onKeydown"></Input>
                </FormItem>
                <FormItem v-if="addData.columns" :label="$L('任务列表')">
                    <TagInput v-model="addData.columns"/>
                </FormItem>
                <FormItem v-else :label="$L('项目模板')">
                    <Select :value="0" @on-change="selectChange" :placeholder="$L('请选择模板')">
                        <Option v-for="(item, index) in columns" :value="index" :key="index">{{ item.name }}</Option>
                    </Select>
                </FormItem>
                <FormItem prop="flow" :label="$L('开启工作流')">
                    <RadioGroup v-model="addData.flow">
                        <Radio label="open">{{$L('开启')}}</Radio>
                        <Radio label="close">{{$L('关闭')}}</Radio>
                    </RadioGroup>
                </FormItem>
            </Form>
            <div slot="footer" class="adaption">
                <Button type="default" @click="addShow=false">{{$L('取消')}}</Button>
                <Button type="primary" :loading="loadIng > 0" @click="onAddProject">{{$L('添加')}}</Button>
            </div>
        </Modal>
    </div>
</template>

<script>
import {mapState} from "vuex";
import {sortBy} from "lodash";
import DrawerOverlay from "../../components/DrawerOverlay";
import UserSelect from "../../components/UserSelect.vue";
import UserAvatarTip from "../../components/UserAvatar/tip.vue";


export default {
    components: {UserAvatarTip, UserSelect, DrawerOverlay},
    data() {
        return {
            addShow: false,
            addData: {
                name: '',
                columns: '',
                flow: 'open',
            },
            addRule: {
                name: [
                    { required: true, message: this.$L('请填写项目名称！'), trigger: 'change' },
                    { type: 'string', min: 2, message: this.$L('项目名称至少2个字！'), trigger: 'change' }
                ]
            },

            list: [],
            loadIng: 0,

        }
    },

    created() {

    },

    mounted() {

    },

    activated() {
        this.getProjectList();
    },
    deactivated() {
        this.addShow = false;
    },
    computed: {
        ...mapState(['systemConfig', 'userIsAdmin', 'userInfo', 'fileLists', 'windowWidth', 'filePackLists','formOptions']),

        pid() {
            const {folderId} = this.$route.params;
            return parseInt(/^\d+$/.test(folderId) ? folderId : 0);
        },



        headers() {
            return {
                fd: $A.getSessionStorageString("userWsFd"),
                token: this.userToken,
            }
        },

        shareAlready() {
            let data = this.shareList ? this.shareList.map(({userid}) => userid) : [];
            if (this.shareInfo.userid) {
                data.push(this.shareInfo.userid);
            }
            return data
        },

        fileList() {
            const {fileLists, searchKey, hideShared, pid, selectIds, userId} = this;
            const list = $A.cloneJSON(sortBy(fileLists.filter(file => {
                if (hideShared && file.userid != userId && file.created_id != userId) {
                    return false
                }
                if (searchKey) {
                    return file.name.indexOf(searchKey) !== -1;
                }
                return file.pid == pid;
            }), file => {
                return (file.type == 'folder' ? 'a' : 'b') + file.name;
            }));
            return list.map(item => {
                item._checked = selectIds.includes(item.id)
                return item;
            })
        },

        navigator() {
            let {pid, fileLists} = this;
            let array = [];
            while (pid > 0) {
                let file = fileLists.find(({id, permission}) => id == pid && permission > -1);
                if (file) {
                    array.unshift(file);
                    pid = file.pid;
                } else {
                    pid = 0;
                }
            }
            return array;
        },
        showBtnText(){
            return this.windowWidth > 600;
        }
    },

    watch: {
        pid() {
            this.getProjectList();
        },

        fid() {
            this.openFileJudge();
        },

        tableMode(val) {
            $A.IDBSave("fileTableMode", val)
        },
    },

    methods: {
        selectChange(index) {
            this.$nextTick(() => {
                this.$set(this.addData, 'columns', this.columns[index].columns.join(','));
            })
        },
        //获取项目人员头像信息
        getProjectUser(project_id){
            this.$store.dispatch("call", {
                url: 'project/one',
                data: {
                    project_id,
                },
            }).then(({data}) => {
                console.log("获取项目信息返回-->",data)
                const projectIndex = this.list.findIndex(p => p.id === project_id);
                console.log("获取项目信息返回231111-->"+projectIndex)
                console.log("获取项目信息返回22111-->",this.list)
                if (projectIndex !== -1) {
                    // Vue.set(this.list[projectIndex], 'projectData', data);
                    this.$set(this.list[projectIndex], 'projectData', data);
                }
                console.log("获取项目信息返回111112-->",this.list)
            }).catch(({msg}) => {
                $A.noticeWarning(msg);
            }).finally(_ => {
                // 强制更新视图
                this.$forceUpdate();
            });
        },

        async toggleRoute(path, params) {
            this.showMobileMenu = false;
            let location = {name: 'manage-' + path, params: params || {}};
            let fileFolderId = await $A.IDBInt("fileFolderId");
            if (path === 'file' && fileFolderId > 0) {
                location.params.folderId = fileFolderId
            }
            this.goForward(location);
        },
        // 计算时间距离现在的时间
        calculateTimeAgo(time,type=1) {
            const timeDiff = $A.dayjs().unix() - $A.dayjs(time).unix(); // convert to seconds
            if (timeDiff < 60) {
                return type == 2 ? "0" + this.$L('分钟') : this.$L('刚刚');
            } else if (timeDiff < 3600) {
                const minutes = Math.floor(timeDiff / 60);
                return type == 2 ? `${minutes}${this.$L('分钟')}` : `${minutes} ${this.$L('分钟前')}`;
            } else if (timeDiff < 3600 * 24) {
                const hours = Math.floor(timeDiff / 3600);
                return type == 2 ? `${hours}${this.$L('小时')}` : `${hours} ${this.$L('小时前')}`;
            } else if (timeDiff < 3600 * 24 * 30) {
                const days = Math.floor(timeDiff / 3600 / 24);
                return type == 2 ? `${days + 1}${this.$L('天')}` : `${days + 1} ${this.$L('天前')}`;
            } else {
                const days = Math.floor(timeDiff / 3600 / 720);
                return type == 2 ? `${days + 1}${this.$L('月')}` : `${days + 1} ${this.$L('月前')}`;
            }
        },

        formatDate(dateString) {
            // return ; // 使用空格分割字符串并返回第一部分（日期）
            return dateString.split(' ')[0].split('-').slice(1).join('-');
        },
        //监听回车事件
        onKeydown(e) {
            if (e.keyCode === 13) {
                if (e.shiftKey) {
                    return;
                }
                e.preventDefault();
                if(this.addShow){
                    this.onAddProject()
                }

                if(this.addTaskShow){

                }
            }
        },
        //新建项目的弹出框显示
        onAddProjectShow(){
            this.$store.dispatch("getColumnTemplate").catch(() => {})
            this.addShow = true;
            this.$nextTick(() => {
                this.$refs.projectName.focus();
            })
        },
        //实际添加的网络请求
        onAddProject() {
            this.$refs.addProject.validate((valid) => {
                if (valid) {
                    this.loadIng++;
                    this.$store.dispatch("call", {
                        url: 'project/add',
                        data: this.addData,
                    }).then(({data, msg}) => {
                        $A.messageSuccess(msg);
                        this.addShow = false;
                        this.$refs.addProject.resetFields();
                        this.$store.dispatch("saveProject", data);
                        this.toggleRoute('project', {projectId: data.id})
                    }).catch(({msg}) => {
                        $A.modalError(msg);
                    }).finally(_ => {
                        this.loadIng--;
                    });
                }
            });
        },

        getProjectList() {
            // if (this.$route.name !== 'manage-file') {
            //     return;
            // }
            this.loadIng++;
            this.$store.dispatch("call", {
                url: 'project/lists',
                data: {
                    // keys: this.keys,
                    all: 1,
                    archived:"all",
                    type:"all"
                    // page: Math.max(this.page, 1),
                    // pagesize: Math.max($A.runNum(this.pageSize), 10),
                },
            }).then(({data}) => {
                this.list = data.data;
                //遍历获取人员信息
                this.list.forEach(item => {
                    this.getProjectUser(item.id);
                    item.projectData = null;
                });
                console.log("response--->111111111111111111111111111122222");
                console.log(this.list);
            }).catch(() => {
                // this.noText = '数据加载失败';
            }).finally(_ => {
                this.loadIng--;
            })
        }
    }
}
</script>
<style>

.project-icons > li.project-avatar > ul > li:first-child {
    margin-left: 0;
}
.project-icons > li.project-avatar > ul > li {
    list-style: none;
    margin-left: -6px;
    transition: transform 0.2s;
}


</style>
