<template>
    <div >
        <div @mouseover="showBg=true" @mouseleave="showBg=false">
            <div class="bg" v-if="showBg" :style="`line-height:${imgHeight}`">点击上传</div>
            <input type="file" class="input-file" :style="`width:${imgWidth};height:${imgHeight};`" name="avatar" ref="avatarInput" @change="changeImage($event)" accept="image/gif,image/jpeg,image/jpg,image/png">
            <img  alt="" :style="`width:${imgWidth};height: ${imgHeight};`" name="avatar">
        </div>
        <div class="text" @click="upload" v-if="file">确定上传</div>
    </div>
</template>

<script>
    /*:src="avatar?avatar:require('')"*/
    export default {
        name: "UserUpLoad",
        data(){
            return{
                avatar: '',
                file: '',
                showBg: false
            }
        },
        props: ["uploadType", "imgUrl", "imgWidth", "imgHeight"],
        created(){
            this.avatar = this.imgUrl
        },
        methods: {
            changeImage: function(e){
                let file = e.target.files[0];
                if(file) {
                    this.file = file
                    // console.log(this.file)
                    let reader = new FileReader()
                    let that = this
                    reader.readAsDataURL(file)

                    reader.onload= function(e){
                        // 这里的this 指向reader
                        that.avatar = this.result
                    }
                }
            },
            upload: function(){
                let files = this.$refs.avatarInput.files
                let fileData = {}
                if(files instanceof Array) {
                    fileData = files[0]
                } else {
                    fileData = this.file
                }
                // console.log('fileData', typeof fileData, fileData)
                let data = new FormData()
                data.append('multfile', fileData)
                data.append('operaType', this.uploadType)
                this.$store.dispatch('UPLOAD_HEAD', data)
                    .then(res => {
                        // console.log(res)
                        this.file = '';
                        let data = res.data.data;
                        this.$emit("uploadAttachment.vue", data );
                        this.$message({
                            type: "success",
                            message: "上传成功！"
                        })
                    }).catch(err => {
                    // console.log(err)
                    if(err.data.msg){
                        this.$message({
                            type: "error",
                            message: err.data.msg
                        })
                    }else {
                        this.$message({
                            type: "error",
                            message: "上传失败"
                        })
                    }
                })
            }
        }
    }



</script>

<style lang="less" scoped>
    .avatar {
        cursor: pointer;
        position: relative;
        .input-file {
            position: absolute;
            top: 0;
            left: 0;
            opacity: 0;
            cursor: pointer;
        }
        .bg {
            width: 100%;
            height: 100%;
            color: #fff;
            background-color: rgba(0,0,0,0.3);
            text-align: center;
            cursor: pointer;
            position: absolute;
            top: 0;
            left: 0;

        }
        .text {
            padding-top: 10px;
            color: lightblue;
        }
    }
</style>
