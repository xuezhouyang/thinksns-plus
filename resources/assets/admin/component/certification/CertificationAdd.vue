<style lang="css" module>
    .container {
        padding: 15px;
    }
    .avatar {
      height:20px;
      width:20px;
      display:inline-block;
      margin-right:20px;
    }
</style>

<template>
        <div :class="$style.container">
          <div class="panel panel-default">
              <div class="panel-heading">
                <router-link type="button" class="btn btn-primary btn-sm" :to="{name: 'certification:users'}">返回</router-link>
              </div>
              <div class="panel-body">
                <div class="col-md-6 col-md-offset-3" v-show="!loadding">
                    <div class="form-group">
                    <label><span class="text-danger">*</span>用户ID：</label>
                     <div class="input-group-btn">
                        <div class="row">
                            <div class="col-md-6">
                              <input type="text" class="form-control" placeholder="用户ID" v-model="certification.user_id" disabled="disabled">
                            </div>
                            <div class="col-md-6 dropdown" :class="dropdownMenuClass">
                                <input type="text" class="form-control" placeholder="输入用户名搜索" @input="searchUser" v-model="username">
                                <ul class="dropdown-menu" style="margin-left:15px;">
                                  <template v-if="users.length">
                                    <li  v-for="user in users" @click.prevent="choiceUser(user.id)">
                                      <a href="javascript:;"><img :src="user.avatar+'?s=40'" class="img-circle" :class="$style.avatar">
                                        <span>{{ user.name }}</span>
                                      </a>
                                    </li>
                                  </template>
                                  <template v-else>
                                    <li @click.prevent="choiceUser(0)"><a href="javascript:;">无相关记录</a></li>
                                  </template>
                                </ul>
                            </div>
                        </div>
                      </div>
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>真实姓名：</label>
                        <input type="text" class="form-control" v-model="certification.name">
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>手机号：</label>
                        <input type="text" class="form-control" v-model="certification.phone">
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>身份证号：</label>
                        <input type="text" class="form-control" v-model="certification.number">
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>认证类型：</label>
                        <select class="form-control" v-model="certification.type">
                            <option :value="categroy.name" v-for="categroy in categories">{{ categroy.display_name }}</option>
                        </select>
                    </div>
                    <div class="form-group" v-show="certification.type == 'org'">
                        <label><span class="text-danger">*</span>组织名称：</label>
                        <input type="text" class="form-control" v-model="certification.org_name">
                    </div>
                    <div class="form-group" v-show="certification.type == 'org'">
                        <label><span class="text-danger">*</span>组织地址：</label>
                        <input type="text" class="form-control" v-model="certification.org_address">
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>认证描述：</label>
                        <textarea class="form-control" v-model="certification.desc"></textarea>
                    </div>
                    <div class="form-group">
                        <label><span class="text-danger">*</span>认证附件：</label>
                        <img :src="fileBase64" class="img-responsive" style="max-width:200px;margin-bottom: 10px;">
                        <input type="file" @change="uploadAttachment" accept="image/gif,image/jpeg,image/jpg,image/png">
                        <span class="help-block" style="font-size:12px;">附件格式：gif, jpg, jpeg, png； 附件大小：不超过10M</span>
                    </div>
                    <div class="form-group">
                        <button class="btn btn-primary btn-sm" 
                        @click.prevent="createCertification" data-loading-text="提交中" autocomplete="off" id="add-btn">确认</button>
                        <div class="pull-right">
                            <span class="text-danger" v-show="message.error">{{ message.error }}</span>
                            <span class="text-success" v-show="message.success">{{ message.success }}</span>
                        </div>
                    </div>
                </div>
              </div>
          </div>
        </div>
</template>

<script>
import request, { createRequestURI } from '../../util/request';
import plusMessageBundle from 'plus-message-bundle';
const PersonalCertificationEdit = {
    data: () => ({
        loadding: true,
        users: [],
        categories: [],
        fileBase64: '',
        username: '',
        dropdownMenuClass: '',
        certification: {
          user_id: '',
          name: '',
          phone: '',
          number: '',
          files: [],
          org_name: '',
          org_address: '',
          desc: '',
          type:'user',
        },
        message: {
          error: null,
          success: null,
        },
    }),
    methods: {
        /**
         * 获取认证类型
         * @return {[type]} [description]
         */
        getCertificationCategories () {
          request.get(
            createRequestURI('certification/categories'),
            {validateStatus: status => status === 200}
          ).then(response => {
            this.categories = response.data;
            this.loadding = false;
          }).catch(({ response: { data: { errors = ['加载认证详情失败'] } = {} } = {} }) => {
            let Message = new plusMessageBundle(data);
            this.message.error = Message.getMessage();
          }); 
        },
        /**
         * 创建用户认证
         * @return {[type]} [description]
         */
        createCertification () {
          $('#add-btn').button('loading');
          request.post(
            createRequestURI('certifications'),
            { ...this.certification },
            { validateStatus: status => status === 201 }
          ).then(({ data: { message: [ message ] = [] } }) => {
            $('#add-btn').button('reset');
            this.message.success = message;
          }).catch(({ response: { data = {} } = {} }) => {
            $('#add-btn').button('reset');
            let Message = new plusMessageBundle(data);
            this.message.error = Message.getMessage();
          });
        },
        /**
         * 搜索用户
         */
        searchUser () {
          request.get(
            createRequestURI('find/nocertification/users?keyword=' + this.username),
            { validateStatus: status => status === 200 }
          ).then(response => {
            this.dropdownMenuClass = 'open';
            this.users = response.data;
          }).catch(({ response: { data: { message: [ message ] = [] } = {} } = {} }) => {
            this.search.message = message;
          });
        },
        /**
         * 上传附件
         */
        uploadAttachment (e) {
          var that = this;
          let file = e.target.files[0]; 
          let param = new FormData();
          param.append('file', file);
          // 设置请求头
          let config = {
            headers: { 
              'Content-Type': 'multipart/form-data',
              'Authorization': 'Bearer ' + window.TS.token 
            }
          };
          let reader = new FileReader(); 
          reader.readAsDataURL(file); 
          reader.onload = function(e) {
           that.fileBase64 = e.target.result;
           request.post('/api/v2/files', param, config)
            .then((response) => {
                const { id: id, message: [message] = [] } = response.data;
                that.certification.files = [id];
            }).catch((error) => {
              let Message = new plusMessageBundle(data);
              this.message.error = Message.getMessage();
            });
          }
        },
        choiceUser (userId) {
          let id = parseInt(userId);
          this.dropdownMenuClass = this.username =  '';
          this.certification.user_id = id ? id : '';
        },
    },
    created () {
      this.getCertificationCategories();
    },

};
export default PersonalCertificationEdit;
</script>
