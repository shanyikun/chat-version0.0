<template>
    <div id="page">
        <div id="container">
            <div id="userInfo">
                <userinfo
                        @user-head-portrait-pop="userHeadPortraitPop"
                        @user-setting-pop="userSettingPop">  <!--监听用户头像弹出框事件,监听用户设置弹出框-->
                </userinfo>   <!--用户详情页  不需要通过路由-->
            </div>
            <div id="userList">
                <router-view name="userInfo" :sendmessageflag="sendMessageFlag" :friendname="friendName"></router-view>
            </div>
            <div id="messageContainer">
                <router-view name="chatPage" :key="flag()"
                             @friend-information-pop="friendInformationPop"> <!--监听好友信息弹窗自定义事件-->
                </router-view>    <!--渲染同一个组件时禁止复用-->
            </div>
        </div>

        <div class="friendInformationPop" v-if="isDisplayFriendInformation"><!--好友信息弹出框，绝对定位，相对于页面，放在这里是为了使其位置可以相对于界面-->
            <div class="popButton">
                <div @click="closeFriendInformationPop" class="closeButton iconfont icon-tubiaoguifan"></div>
            </div>
            <div class="friendInformationContainer">
                <div class="friendName">{{friendName}} </div>
                <img :src="friendUrl" class="friendHeadPortrait" width="45px" height="45px"
                     @click="biggerFriendHeadPortrait">
            </div>
            <div class="messageButtonContainer">
                <div class="messageButton iconfont icon-xiaoxi" @click="sendMessage"></div>
            </div>
            <div class="promptMessage" v-if="isDisplayPromptMessage">此用户不在线！</div>
        </div>

        <div class="biggerFriendHeadPortraitPop" v-if="isDisplayFriendHeadPortrait"> <!--好友头像放大弹出框，绝对定位-->
            <div class="popButton">
                <div @click="closeFriendHeadPortraitPop" class="closeButton iconfont icon-tubiaoguifan"></div>
            </div>
            <img :src="friendUrl" width="250px">
        </div>

        <div class="biggerUserHeadPortraitPop" v-if="isDisplayUserHeadPortrait">  <!--用户头像放大框，绝对定位-->
            <div class="popButton">
                <div @click="closeUserHeadPortraitPop" class="closeButton iconfont icon-tubiaoguifan"></div>
            </div>
            <img :src="$store.state.url" width="250px">
        </div>

        <div class="userSettingPop" v-if="isDisplayUserSettingPop">
            <div class="popButton">
                <div class="popName">设置</div>
                <div @click="closeUserSettingPop" class="closeButton iconfont icon-tubiaoguifan"></div>
            </div>
            <div class="settingContainer">
                <div class="iconfont icon-shangchuantupian">
                    <input type="file" name="userHeadPortrait" multiple ref="fileInput" @change="getFileName">
                </div>
                <div class="uploadFileName">{{imageName}}</div>
                <div class="iconfont icon-shangchuan" @click="updateUserHeadPortrait" title="上传"></div>
            </div>
            <div class="promptMessage" ref="updatePromptMessage"></div>
        </div>

    </div>
</template>

<script>
import userInfo from './userInfo.vue'     //引入用户详情组件   绝对路径引入会出错

/*var userInfo={    //在vue文件中使用组件模板对象也无效
    template: '<h1>asdasd</h1>'
}*/

 export default {
     data: function(){
         return {
             isDisplayFriendInformation: false,  //是否显示好友信息弹窗
             isDisplayFriendHeadPortrait: false,  //是否显示好友头像弹出框
             isDisplayUserHeadPortrait: false,    //是否显示用户头像弹出框
             isDisplayPromptMessage: false,     //是否显示用户不在线提示消息
             isDisplayUserSettingPop: false,
             timeOutReturnValue: null,      //消息提示定时器返回值
             friendName: '',   //好友名
             friendUrl: '',     //好友头像链接
             sendMessageFlag: false,    //好友信息弹出框发送信息标志位，用于向onlineUserList子组件传递发送信息标志以触发子组件内的getChatPage函数
             imageName: ''   //上传图片名字
         }
     },
     methods: {
         flag: function(){  // 为每一个聊天框加上一个唯一标识，防止组件复用
             return Date.now()
         },
         friendInformationPop : function(friend){  //第一个参数默认是子组件传递过来的参数
             this.friendName=friend.name
             this.friendUrl=friend.url
             this.isDisplayFriendInformation=true
         },
         biggerFriendHeadPortrait: function(){    //显示放大版好友头像
             this.isDisplayFriendInformation=false
             this.isDisplayFriendHeadPortrait=true
         },
         userHeadPortraitPop: function(){    //显示放大版用户头像
             this.isDisplayUserHeadPortrait=true
         },
         closeFriendInformationPop: function(){   //关闭好友信息弹窗
             this.isDisplayFriendInformation=false
         },
         closeFriendHeadPortraitPop: function(){   //关闭好友头像弹窗
             this.isDisplayFriendHeadPortrait=false
         },
         closeUserHeadPortraitPop: function(){    //关闭用户头像弹窗
             this.isDisplayUserHeadPortrait=false
         },
         userSettingPop: function(){
             this.isDisplayUserSettingPop=true
         },
         closeUserSettingPop: function(){
             this.isDisplayUserSettingPop=false
             this.imageName=''    //清空图片名
         },
         updateUserHeadPortrait: function(){
             let request=new XMLHttpRequest()
             let formData=new FormData()    /*格式化数据，是key/value对*/
             let files=this.$refs.fileInput.files
             if(!files[0]){
                 this.$refs.updatePromptMessage.textContent='请先上传文件！'
                 if(this.timeOutReturnValue){
                     clearTimeout(this.timeOutReturnValue)
                 }
                 this.timeOutReturnValue=setTimeout(()=>{
                     this.$refs.updatePromptMessage.textContent=''
                 }, 2000)
                 return undefined
             }
             formData.append('userHeadPortrait', files[0])
             formData.append('name', this.$store.state.name)
             request.open('post', '/updateUserHeadPortrait', true)
             request.onreadystatechange=()=>{
                 if(request.readyState===4){
                     this.$refs.updatePromptMessage.textContent=request.responseText
                     if(this.timeOutReturnValue){
                         clearTimeout(this.timeOutReturnValue)
                     }
                     this.timeOutReturnValue=setTimeout(()=>{
                         this.$refs.updatePromptMessage.textContent=''
                     }, 2000)
                 }
             }
             request.send(formData)
         },
         getFileName: function(){
             let pathArray=event.target.value.split('\\')   //获取路径
             this.imageName=pathArray[pathArray.length-1]   //获取图片名
         },
         sendMessage: function(){
             if(this.$store.state.onlineUserList.find((item)=>{
                 return item.name===this.friendName
             })!==undefined){   //判断好友是否在线，若在线则发送跳转至发送消息界面
                 if(this.$route.path==='/onlineUserList'||this.$route.path==='/chatPage'){ //判断是在聊天界面还是在好友详情界面触发
                     this.sendMessageFlag=!this.sendMessageFlag  //改变好友信息弹出框发送信息标志位，用于向onlineUserList子组件传递发送信息标志以触发子组件内的getChatPage函数
                     this.isDisplayFriendInformation=false
                 }
                 else {  //若是在好友详情界面触发
                     this.$router.push({path: '/onlineUserList', query: {name: this.friendName}})
                     this.isDisplayFriendInformation=false
                 }
             }
             else {  //若好友不在线则显示消息提示，并定时2s消失
                 this.isDisplayPromptMessage=true
                 if(this.timeOutReturnValue){
                     clearTimeout(this.timeOutReturnValue)
                 }
                 this.timeOutReturnValue=setTimeout(()=>{
                     this.isDisplayPromptMessage=false
                 }, 2000)
             }
         }
     },
     components: {
         userinfo: userInfo
     },
     created: function(){
     },
     beforeMount: function(){
     }
 }
</script>

<style scoped>
    @import '../public/stylesheets/closeButton-icon-font/iconfont.css';  /*引入关闭按钮字体图标*/
    @import '../public/stylesheets/message-icon-font/iconfont.css';   /*引入消息按钮字体图标*/
    @import '../public/stylesheets/uploadimage-icon-font/iconfont.css';  /*引入上传图片字体图标*/
    @import '../public/stylesheets/uploadbutton-icon-font/iconfont.css';  /*引入上传按钮字体图标*/

    #page{                           /*页面*/
        display: flex;
        justify-content: space-around;
        position: relative;
    }
    #container{                     /*内容容器*/
        width: 55%;
        display: flex;
        justify-content: space-around;
        height: 500px;
    }
    #userInfo{                     /*用户详情模块*/
        width: 6%;
        background-color: #27292C;
        text-align: center;
    }
    #userList{                     /*用户列表模块*/
        width: 28%;
        background-color: #EAE8E7;
    }
    #messageContainer{           /*聊天内容模块*/
        width: 66%;
        background-color: #F5F5F5;
    }
    .friendInformationPop{    /*好友信息弹窗模块*/
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translateX(-50%) translateY(-50%);   /*使元素位于页面中间*/
        width: 250px;
        height: 200px;
        box-shadow: 0px 0px 3px gray;
        border-radius: 3px;
        background-color: white;
    }
    .friendInformationPop .friendInformationContainer{
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin: 20px 30px 0 30px;
        padding-bottom: 20px;
        border-bottom: solid 1px lightgray;
    }
    .friendInformationPop .friendInformationContainer .friendHeadPortrait:hover{
        cursor: pointer;
    }
    .friendInformationPop .popButton{
        display: flex;
        justify-content: flex-end;
    }
    .friendInformationPop .popButton .closeButton{
        padding: 4px;
    }
    .friendInformationPop .popButton .closeButton:hover{
        background-color: red;
        color: white;
    }
    .friendInformationPop .messageButtonContainer{
        display: flex;
        justify-content: flex-end;
        margin: 30px 30px 0 30px;
    }
    .friendInformationPop .messageButtonContainer .messageButton{
        font-size: 30px;
        color: gray;
    }
    .friendInformationPop .messageButtonContainer .messageButton:hover{
        color: black;
        cursor: pointer;
    }
    .friendInformationPop .promptMessage{
        margin: 0 30px;
        text-align: right;
        color: red;
        font-size: 14px;
    }
    .biggerFriendHeadPortraitPop, .biggerUserHeadPortraitPop{
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translateX(-50%) translateY(-50%);   /*使元素位于页面中间*/
        width: 250px;
        height: 275px;
        box-shadow: 0px 0px 3px gray;
        border-radius: 3px;
        background-color: white;
    }
    .biggerFriendHeadPortraitPop .popButton, .biggerUserHeadPortraitPop .popButton{
        display: flex;
        justify-content: flex-end;
    }
    .biggerFriendHeadPortraitPop .popButton .closeButton, .biggerUserHeadPortraitPop .popButton .closeButton{
        padding: 4px;
    }
    .biggerFriendHeadPortraitPop .popButton .closeButton:hover, .biggerUserHeadPortraitPop .popButton .closeButton:hover{
        background-color: red;
        color: white;
    }
    .userSettingPop{
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translateX(-50%) translateY(-50%);
        width: 300px;
        height: 200px;
        box-shadow: 0px 0px 3px gray;
        border-radius: 3px;
        background-color: white;
    }
    .userSettingPop .popButton{
        display: flex;
        justify-content: space-between;
    }
    .userSettingPop .popButton .popName{
        color: gray;
        padding: 4px;
        font-size: 14px;
    }
    .userSettingPop .popButton .closeButton{
        padding: 4px;
    }
    .userSettingPop .popButton .closeButton:hover{
        background-color: red;
        color: white;
    }
    .userSettingPop .settingContainer{
        margin: 20px 30px 0 30px;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .userSettingPop .settingContainer .icon-shangchuantupian{
        font-size: 40px;
        position: relative;
    }
    .userSettingPop .settingContainer .icon-shangchuantupian:hover{
        color:limegreen;
    }
    .userSettingPop .settingContainer .icon-shangchuantupian input{
        position: absolute;
        display: inline-block;
        left: 0px;
        top: 0px;
        width: 40px;
        height: 40px;
        opacity: 0;
    }
    .userSettingPop .settingContainer .uploadFileName{
    }
    .userSettingPop .settingContainer .icon-shangchuan{
        font-size: 30px;
        font-weight: lighter;
        color: gray;
    }
    .userSettingPop .settingContainer .icon-shangchuan:hover{
        color: black;
    }
    .userSettingPop .promptMessage{
        margin: 20px 30px 0 30px;
        color: red;
    }
</style>
<!--
 +++关于div p li ul标签嵌套的问题， div嵌套div时，内层div没有margin, div嵌套p时，p有margin
    p不能嵌套div, p也不可以嵌套p, li可以嵌套div且div没有margin, li也可以嵌套p且p没有margin
    ul嵌套li时，li也没有margin, 只是此时ul有padding-left*, li之间有margin, 默认都是没有
    padding的,ul嵌套li时ul的padding-left除外
 +++关于内联元素的一些特性， 内联元素即使在同一行，之间也会有间隔，这个间隔既不是margin也不是
    padding,若本身不是内联元素，而是设置display为内联，则之间的间隔属于padding的作用范畴，
    比如背景色会显示出来,若设置为inline-block则背景色不会显示出来，本身是内联元素的一律不会
    显示背景色,但是并不能通过设置padding为零来消除，可以通过float来消除内联元素之间的间隔
-->