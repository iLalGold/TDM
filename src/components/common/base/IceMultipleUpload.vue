<template>
    <div style="width: 100%">
        <div v-if="showFiles" class="ice-multiple-upload" style="height: 100%;display: flex;flex-direction: column">
            <div v-if="!uploaderDisabled">
                <el-button :icon="icon" :type="uploadBtnStyle" @click="startUpload" unauth>{{btnName}}</el-button>
            </div>
            <div v-if="!uploaderDisabled" class="form-upload-file-list">
                <div class="ice-full-absolute">
                    <vue-scroll :ops="{bar:{background:'#000',opacity:0.4,keepShow:true,size:'6px',onlyShowBarOnScroll:true},scrollPanel:{scrollingX:false}}">
                        <div v-for="tag in filesItem" class="form-upload-file-list__item">
                            <div class="inlineBlock">
                            <span style="color: #ff6949;width: 50px"
                                  v-if="tag.securityLocked&&tag.securityText&&tag.name" title="密级不能修改"> [{{tag.securityText}}]</span>
                                <el-dropdown slot="prefix" placement="bottom"
                                             v-if="tag.securityText&&tag.name&&!tag.securityLocked">
                                    <span style="color: #ff6949;cursor: pointer"> [{{tag.securityText}}]</span>
                                    <el-dropdown-menu slot="dropdown">
                                        <el-dropdown-item
                                                :command="tag"
                                                @click.native="updateSecret(item.value,tag)"
                                                :title="!item.disable?`点击修改附件密级为：${item.name}`:''"
                                                :style="'background: rgb(255, '+(255-30*index)+', '+(255-30*index)+');margin:0 5px;'"
                                                v-for=" (item,index) in secrets.filter(item=>!item.disable)"
                                                :key="item.value">
                                            <i >{{item.name}}</i>
                                        </el-dropdown-item>
                                    </el-dropdown-menu>
                                </el-dropdown>
                            </div>

                            <div class="inlineBlock widthCalc">
                                <div class="inlineBlock">
                                    <i class="el-icon-document"></i>
                                </div>
                                <div style="display: inline-flex;" class="widthCalc">
                                    <span class="textOverFlow" :title="getChangedName(tag.security,tag.name)">
                                        {{getChangedName(tag.security,tag.name)}}
                                    </span>
                                </div>
                                <div class="inlineBlock">
                                    <i class="el-icon-close hoverClose" @click="handleClose(tag)"></i>
                                </div>
                            </div>
                        </div>
                    </vue-scroll>
                </div>
            </div>

            <div v-if="uploaderDisabled" style="flex-grow: 1;position: relative;">
                <div class="ice-full-absolute" v-if="files&&files.length>0">
                    <vue-scroll :ops="{bar:{background:'#000',opacity:0.4,keepShow:true,size:'6px',onlyShowBarOnScroll:true},scrollPanel:{scrollingX:false}}">
                        <div v-for="file in files">
                            <div class="inlineBlock fileSecretColor"
                                 v-if="file.securityText">
                                [{{file.securityText}}]
                            </div>
                            <div class="inlineBlock widthCalc">
                                <div class="inlineBlock">
                                    <i class="el-icon-document"></i>
                                </div>
                                <div style="display: inline-flex;" class="widthCalc">
                                    <span class="textOverFlow" :title="getChangedName(file.security,file.name)">
                                        {{getChangedName(file.security,file.name)}}
                                    </span>
                                </div>
                                <div class="inlineBlock">
                                    <el-button type="text" @click="$downloadFile(file.response.data)"
                                               priorDisabled="false">
                                        下载
                                    </el-button>
                                </div>
                            </div>
                        </div>

                    </vue-scroll>
                </div>
                <div v-if="!files||files.length==0">未上传附件</div>
            </div>
        </div>
        <div v-if="!showFiles">
            <el-button :disabled="disabled" :icon="icon" :type="uploadBtnStyle" @click="startUpload" unauth>
                {{btnName}}
            </el-button>
        </div>

        <ice-dialog :visible.sync="visible" width="580px" title="附件上传" :buttons="buttons">
            <el-upload
                    ref="upload"
                    style="padding:16px;margin: auto;text-align: center"
                    class="ice-multiple-uploader"
                    action="/"
                    :multiple="true"
                    :show-file-list="false"
                    :before-upload="defaultBeforeUpload"
                    :on-change="onchange"
                    :on-exceed="onExceed"
                    :on-success="success"
                    :on-error="error"
                    :disabled="false"
                    :file-list="files"
                    :on-progress="onProgress"
                    :accept="accepts"
                    :auto-upload="false"
                    :on-remove="beforeRemove"
                    :http-request="checkedFile"
                    :before-remove="beforeRemove"
                    :limit="10">
                <div>
                    <i class="el-icon-upload"
                       style="font-size: 67px;color: #c0c4cc;margin: 10px 0px;line-height: 50px;"></i>
                    <div class="el-upload__text"><!--将文件拖到此处，或--><em>点击上传</em></div>
                    <div class="el-upload__tip" slot="tip">单个文件大小不得超过<em>10GB</em></div>
                </div>
            </el-upload>
            <!-- <ice-form-group name="" :expandable="false">-->
            <div class="upload-file-list">
                <vue-scroll :ops="{bar:{background:'#000',opacity:0.4,keepShow:true,size:'6px',onlyShowBarOnScroll:true},scrollPanel:{scrollingX:false}}">
                    <transition-group
                            tag="ul"
                            class="el-upload-list el-upload-list--text"
                            name="el-list">
                        <li v-for="file in files"
                            :class="['el-upload-list__item', 'is-' + file.status,  focusing ? 'focusing' : '']"
                            :key="file.uid"
                            tabindex="0"
                            @keydown.delete="!disabled && handleRemove(null, file)"
                            @focus="focusing = true"
                            @blur="focusing = false"
                            @click="focusing = false"
                            style="text-overflow: ellipsis;overflow: hidden;white-space: nowrap;word-break: keep-all;padding-right:20px;width:500px;  ">
                            <el-popover
                                    v-if="doSecret&&file.status!=='success'&&!file.securityLocked"
                                    placement="right"
                                    :width="85*secrets.length+''"
                                    v-model="file.visible">
                                <el-radio-group v-model="file.security" size="mini"
                                                style="display: flex;justify-content: flex-start;width: 100%">
                                    <el-radio :label="item.value" border
                                              @click.native="selectSecurity(file,item.value)"
                                              :style="'background: rgb(255, '+(255-30*index)+', '+(255-30*index)+');margin: 0;margin-right: 12px;'"
                                              v-for=" (item,index) in secrets" v-if="!item.disable">
                                        {{item.name}}
                                    </el-radio>
                                </el-radio-group>
                                <el-tag slot="reference" :type="file.securityText?'danger':'info'"
                                        style="display: inline-block;cursor: pointer">
                                    {{file.securityText?('[ '+file.securityText+' ]'):'密级选择'}}
                                </el-tag>
                            </el-popover>
                            <el-tag slot="reference" v-if="doSecret&&file.status=='success'&&!file.securityLocked"
                                    :type="file.securityText?'danger':'info'"
                                    style="display: inline-block;cursor: pointer">
                                {{file.securityText?('[ '+file.securityText+' ]'):'密级选择'}}
                            </el-tag>
                            <el-tag v-if="doSecret&&file.securityLocked"
                                    type="danger"
                                    style="display: inline-block;">
                                {{'[ '+file.securityText+' ]'}}
                            </el-tag>
                            <a style="display: inline" class="el-upload-list__item-name" @click="handleClick(file)">
                                <i class="el-icon-document"></i>{{file.name}}
                            </a>
                            <label class="el-upload-list__item-status-label">
                                <i class="el-icon-upload-success el-icon-circle-check"></i>
                            </label>
                            <i class="el-icon-close" v-if="!disabled"
                               @click="(event)=>{handleRemove(file, file);}"></i>
                            <el-progress
                                    v-if="file.status === 'uploading'"
                                    type="line"
                                    :stroke-width="2"
                                    :percentage="parsePercentage(file.percentage)">
                            </el-progress>
                        </li>
                    </transition-group>
                </vue-scroll>
            </div>
            <!-- </ice-form-group>-->
        </ice-dialog>
    </div>
</template>

<script>
    import IceAdvanceUploader from './IceAdvanceUploader'
    import IceDialog from "./IceDialog";
    import IceFormGroup from "./IceFormGroup";
    import Locale from 'element-ui/src/mixins/locale';
    import VueScroll from 'vuescroll'
    import ElUploadList from 'element-ui/packages/upload/src/upload-list'
    import {mapGetters, mapMutations} from 'vuex'
    import emitter from 'element-ui/src/mixins/emitter';

    export default {
        name: "IceMultipleUpload",
        componentName: "IceMultipleUpload",
        mixins: [IceAdvanceUploader, Locale, emitter],
        model: {
            prop: 'value',
            event: 'filesChange'
        },
        inject: {
            elForm: {
                default: ''
            },
            elFormItem: {
                default: ''
            }
        },
        props: {
            accepts: {
                type: String,
                default: "*"
            },
            value: {
                type: [String, Array],
                default: ""
            },
            doSecret: Boolean,
            disabled: Boolean,
            valueModel: {//返回值：[1,2,3]  "1,2,3"
                type: String,
                default: 'array',
                validator: function (value) {
                    return ['string', 'array'].indexOf(value) !== -1
                }
            },
            uploadBtnStyle: {
                //添加附件按钮的div样式
                type: String,
                default: 'white'
            },
            //优先disabled，优先级高于disabled
            priorDisabled: Boolean,

            showFiles: {//是否展示附件
                type: Boolean,
                default: true
            },
            btnName: {
                //按钮的名称
                type: String,
                default: '添加附件'
            },
            icon: {
                //按钮的图标
                type: String,
                default: 'el-icon-upload'
            },
            isPublic:{//密级是否为不涉密类型
                type:Boolean,
                default:false
            },
            // 父密级控制子密级选择
            controlMj: {
                default: function () {
                    return []
                }
            },
        },
        data() {
            return {
                fileArr: [],
                visible: false,
                buttons: [{
                    iscannel: false, name: '确认上传', click: () => {
                        if(this.filesItem.length>10){
                            this.$message.warning("附件上传不可超过十个");
                            return false
                        }

                        const {files: files} = this;
                        if (this.doSecret) {
                            Object.assign(this.fileArr, files);
                            if (files.filter(file => (file.security == "" || file.security == undefined)).length > 0) {
                                this.$message.warning("请选择附件密级")
                                return false;
                            }
                        }
                        if (files.filter(file => file.status == 'ready').length <= 0 && files.filter(file => file.status == 'success').length == 0) {
                            this.$message.warning("没有找到可以上传的附件")
                            return false;
                        }

                        if (files.filter(file => file.status == 'ready').length <= 0) {

                        } else {
                            this.$refs.upload.submit();
                            //return false;
                        }
                    }
                }, {iscannel: true, name: '取消'}],
                listType: 'text',
                files: [],
                focusing: false,
                handlePreview: () => {
                },
                /*  secrets: [],//附加密级列表*/
                filesItem: [],//展示所用的附件数组
                widthUpload: 0
            }
        },
        methods: {

            ...mapMutations('datamapStore', ['addUndoTypeCodes']),
            ...mapGetters('datamapStore', ['getDataMap', 'getReversedDataMap']),

            /**附件上传事件*/
            startUpload() {
                this.files = [];
                this.filesItem=[];
                this.$nextTick(() => {
                    this.visible = true;
                });
            },
            reset(){
                this.files = [];
                this.filesItem=[];
            },
            /**删除附件*/
            handleClose(tag) {
                const files = [...this.files]
                const filesItem = [...this.filesItem]
                if (files.length > 0) {
                    files.splice(files.indexOf(tag), 1);
                    filesItem.splice(filesItem.indexOf(tag), 1);
                    this.files = files;
                    this.filesItem = filesItem;
                } else {
                    filesItem.splice(filesItem.indexOf(tag), 1);
                    this.filesItem = filesItem;
                }
                this.fireFilesChange(tag);

            },

            /**修改附件密级*/
            updateSecret(value, tag) {
                if (tag && tag.securityLocked) {
                    this.$message.warning("此文件不允许修改密级")
                    return;
                }
                if (tag.response.data) {
                    this.$axios
                        .put(`/resources/attachment/saveOrUpdate`, {oid: tag.response.data, fileSecret: value})
                        .then(response => {
                            /*this.security = commond;*/
                            this.filesItem[this.filesItem.indexOf(tag)].securityText = this.getSecurityText(value);
                            this.filesItem[this.filesItem.indexOf(tag)].security = value;
                            let arr = Object.assign([], this.filesItem);
                            this.filesItem = [];
                            this.filesItem = arr;
                            this.$message.success("密级修改成功")
                            this.dispatch("IceFlowForm", "ice-upload-secret", {
                                "modifyFiles": [{fileid: tag.response.data, secret: value}]
                            });
                        })
                        .catch(error => {
                                console.error(error);
                                this.$message.error("密级修改失败")
                            }
                        );
                }
            },
            defaultBeforeUpload: function (file) {
                this.filename = file.name;
            },
            onchange: function (file, fileList) {

                if (this.doSecret && file.status == 'ready') {
                    if (this.filenameHasSecret(file.name)) {
                        const fileSecretText = this.getSecretByFileName(file.name);
                        file.security = this.getReversedDataMap()("DATA_SECRET_LEVEL")[fileSecretText]
                        this.$set(file, "securityText", fileSecretText);
                        this.$set(file, "securityLocked", true);
                    }
                }


                this.files = fileList.map(item => {
                    this.$set(item, "security", item.security ? item.security : '');
                    this.$set(item, "visible", false);
                    return item;
                })
            },


            /**
             * 组装父组件change回调的方法参数
             * @param array 文件数组
             * @param removedFileId 删除文件的ID
             * @returns {any[]} [id数组,{ removedFileId: 删除文件id,fileNames:文件名数组}]
             */
            assembleIdAndNameArr(array, removedFileId) {
                let id_arr = [], name_arr = [];
                array.map(item => {
                    id_arr.push(item.response.data);
                    name_arr.push(item.name);
                });
                if (this.valueModel == "string") {
                    return [id_arr.join(","), {removedFileId: removedFileId, fileNames: name_arr.join(",")}];
                } else {
                    return [id_arr, {removedFileId: removedFileId, fileNames: name_arr}];
                }
            },
            success: function (response) {
                const index = this.files.findIndex(item => item.status !== 'success');
                if (index == -1) {
                    this.$message.success("上传成功");
                    this.$emit("change", ...this.assembleIdAndNameArr(this.files));
                    this.$emit("successData", this.files);
                    // this.filesItem.push(...this.files);
                    const filesItem = [...this.filesItem, ...this.files];

                    if(filesItem.length>10){
                        this.filesItem = filesItem.filter((item,index)=>{index <=9;});
                    }else{
                        this.filesItem = filesItem;
                    }
                    this.fireFilesChange();
                    this.visible = false;
                }


            },
            error(err, file, fileList) {
                console.log(err, file, fileList)
            },
            onProgress(event, file, fileList) {
                let percent = Number(Number(event.percent).toFixed(0))
            },
            beforeRemove(file) {
                this.uploading = false;
                this.status = null;
                this.removeFile(file)
            },
            parsePercentage(val) {
                if (val) {
                    return parseInt(val, 10);
                } else {
                    return 0;
                }

            },
            handleClick(file) {
                this.handlePreview && this.handlePreview(file);
            },
            handleRemove(file, raw) {
                const index = this.files.findIndex(item => item.uid == file.uid);
                if (index != -1) {
                    this.files.splice(index, 1)
                    if (file.size && this.$refs.upload) {
                        this.$refs.upload.handleRemove(file, raw ? raw : file)
                    }

                }
                const success = this.files.filter(item => item.status == 'success')
                if (success) {
                    this.$emit("change", ...this.assembleIdAndNameArr(success, file.uid));
                }
            },
            selectSecurity(file, security) {
                file.visible = false
                const s = this.secrets.find(item => item.value == security);
                if (s) {
                    /* this.$set(file, 'securityText', s.name);*/
                    this.$set(file, 'securityText', s.name);
                }
            },
            clearFiles() {
                this.$refs.upload.clearFiles();
                this.files = [];
            },
            onExceed(files, fileList) {
                this.$message.warning("超过上传最大数量20");
            },
            fireFilesChange(removeFile) {
                if (this.valueModel == 'array') {
                    this.$emit("filesChange", this.filesItem.map(item => item.response.data), removeFile, this.filesItem.map(item => item.name))
                } else {
                    this.$emit("filesChange", this.filesItem.map(item => item.response.data).join(","), removeFile, this.filesItem.map(item => item.name))
                }
            },
        },
        watch: {
            filesItem(newValue, oldValue) {
                //向上传递密级
                this.dispatch("IceFlowForm", "ice-upload-secret", {
                    "addFiles": newValue.map(item => {
                        return {fileid: item.response.data, secret: item.security}
                    }),
                    "delFiles": oldValue.map(item => {
                        return {fileid: item.response.data}
                    })
                });
            },
            fileIds: {
                handler(newValue, OldValue) {
                    if (!newValue) {
                        newValue = []
                    }

                    if (!OldValue) {
                        OldValue = []
                    }

                    if (newValue.join(",") == OldValue.join(",")) {
                        return
                    }

                    if (newValue.length > 0) {
                        const neesQueryFiles = [];
                        let _this = this;
                        newValue.forEach(item => {
                            if (_this.filesItem.findIndex(file => file.response.data == item || file.uid == item) == -1) {
                                neesQueryFiles.push(item);
                            }
                        })
                        // Promise.all(neesQueryFiles.map(item => this.$axios.get(`/resources/attachment/get?id=${item}`)))
                        if (neesQueryFiles.length > 0) {
                            this.$axios.get(`/resources/attachment/get?ids=${neesQueryFiles.join(",")}`)
                                .then(({data}) => {
                                    if (data && data instanceof Array) {

                                        const results = data.map(result => {
                                            return {
                                                uid: result.oid,
                                                name: result.fileName,
                                                security: result.fileSecret,
                                                status: 'success',
                                                response: {data: result.oid, code: 1},
                                                securityText: this.getSecurityText(result.fileSecret),
                                                //securityLocked: this.filenameHasSecret(result.fileName)
                                            }
                                        })

                                        const filesItem = [];
                                        const files = [];

                                        newValue.forEach(fileid => {
                                            let file = results.find(item => item.uid == fileid)
                                            if (!file) {
                                                file = this.filesItem.find(item => item.response.data == fileid)
                                            }
                                            filesItem.push(file);
                                            files.push(file);
                                        })

                                        this.filesItem = filesItem;
                                        this.files = files;

                                    }

                                }).catch(error => {
                                console.error(error)
                            })
                        }
                    } else {
                        this.filesItem = [];
                        this.files = [];
                    }
                },
                immediate: true
            },
        },
        computed: {
            fileIds() {
                if (!this.value) {
                    return [];
                }
                if (this.valueModel == 'array') {
                    return this.value;
                } else {
                    return this.value.length == 0 ? [] : this.value.split(",");
                }
            },
            uploaderDisabled() {
                return this.priorDisabled ||
                    ((this.elFormItem || {disabled: 'unknown'}).disabled === 'unknown' ? (this.disabled || (this.elForm || {}).disabled) : (this.elFormItem || {}).disabled);
            },
        },
        created() {
            //流程初始化完毕事件
            this.$on('ice-flow-secret-ready', data => {
                //alert(data)
            })
        },
        components: {IceFormGroup, IceDialog, VueScroll, ElUploadList}
    }
</script>

<style lang="less" scoped>
    .fileSecretColor {
        //展示密级时的密级字体颜色
        color: #ff6949;
        height: 32px;
        line-height: 32px;
    }

    .widthCalc {
        //减去固定值
        width: calc(100% - 50px);
    }

    .textOverFlow {
        //文本内容超出容器范围时以省略号展示
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
        width: 100%
    }

    .inlineBlock {
        //行内块
        display: inline-block;
    }

    .ice-multiple-upload {
        width: 100%;
        min-height: 120px;

        .file-list {
            /*min-height: 100px;*/
        }
    }

    .hoverClose:hover {
        color: #20a0ff;
        cursor: pointer;
    }

    .upload-file-list {
        height: 180px;
        padding: 16px;

        .el-upload-list{
            padding: 10px 0 0 0
        }
    }

</style>
<style lang="less">
    .ice-multiple-uploader {
        .el-upload-dragger {
            width: 448px;
            height: 200px;
        }

    }

    .ice-multiple-upload {
        .el-upload-list__item:focus {
            outline: none;
        }
    }

    .upload-file-list .el-upload-list__item:focus {
        outline: none;
    }
    .form-upload-file-list{
        flex-grow: 1;
        position: relative;
        .ice-full-absolute{
            padding-top: 10px;
        }
        .form-upload-file-list__item{
            box-sizing: content-box;
            padding: 2px 0;
            line-height: 21px;
        }
    }

</style>
