<style lang="css" module>
    .container {
        padding: 15px;
    }
    .loadding {
        text-align: center;
        font-size: 42px;
    }
    .loaddingIcon {
        animation-name: "TurnAround";
        animation-duration: 1.4s;
        animation-timing-function: linear;
        animation-iteration-count: infinite;
    }
</style>
<template>
  <div :class="$style.container">
    <div v-show="message.success" class="alert alert-success alert-dismissible" role="alert">
        <button type="button" class="close" @click.prevent="dismisAddAreaError">
            <span aria-hidden="true">&times;</span>
        </button>
        {{ message.success }}
    </div>
    <div v-show="message.error" class="alert alert-danger alert-dismissible" role="alert">
        <button type="button" class="close" @click.prevent="dismisAddAreaError">
            <span aria-hidden="true">&times;</span>
        </button>
        {{ message.error }}
    </div>
    <div class="panel panel-default">
      <div class="panel-body">
      <!-- 标签列表 -->
        <table class="table table-striped">
          <thead>
            <tr>
              <th>分类ID</th>
              <th>分类</th>
              <th>拥有标签数量</th>
              <th>权重(越大越靠前)</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-show="loadding">
                <!-- 加载动画 -->
                <td :class="$style.loadding" colspan="5">
                    <span class="glyphicon glyphicon-refresh" :class="$style.loaddingIcon"></span>
                </td>
            </tr>
            <template v-if="empty">
              <tr><td colspan="6" class="text-center">无相关记录</td></tr>
            </template>
            <template v-else>
            <tr v-for="category in tag_categories" :key="category.id" v-if="edit === category.id">
              <td> {{ category.id }}</td>
              <td>
                <input type="text" v-model="edit_name" v-focus placeholder="标签分类名字" />
              </td>
              <td>
                {{ category.tags_count }}
              </td>
              <td>
                <input type="number" v-model="edit_weight" placeholder="分类权重,数字,越大排序越靠前" />
              </td>
              <td>
                <button type="button" @click="hideEdit()" class="btn btn-danger btn-sm" autocomplete="off">
                  取消
                </button>
                <button type="button" @click="update()" class="btn btn-default" autocomplete="off" :disabled="!canEditSend">
                  修改分类
                </button>
              </td>
            </tr>
            <tr v-for="category in tag_categories" :key="category.id" v-if="edit !== category.id">
              <td>{{ category.id }}</td>
              <td>{{ category.name }}</td>
              <td>
                <router-link :to="`/setting/tags?cate=${category.id}`">
                  {{ category.tags_count }}
                </router-link>
              </td>
              <td>
                {{ category.weight }}
              </td>
              <td>
                <!-- 编辑 -->
                <button type="button" class="btn btn-primary btn-sm" @click="showEdit(category.id)">编辑</button>
                <!-- 删除 -->
                <button type="button" class="btn btn-danger btn-sm" @click="deleteCate(category.id)">删除</button>
              </td>
            </tr>
            </template>  
            <!-- <tr>
              <td></td>
              <td>
                <input type="text" ref="focusinput" v-model="name" placeholder="标签分类名字" />
              </td>
              <td></td>
              <td>
                <input type="number" v-model="weight" placeholder="分类权重,数字,越大排序越靠前" />
              </td>
              <td>
                <button 
                  type="submit" 
                  @click="send()" 
                  id="myButton" 
                  data-complete-text="添加成功" 
                  data-loading-text="提交中..." 
                  class="btn btn-default" 
                  autocomplete="off" 
                  :disabled="!canSend"
                >
                  添加分类
                </button>
              </td>
            </tr> -->
          </tbody>
        </table>
      <table class="table table-striped">
        <tbody>
          <tr>
            <td>
              <div class="input-group-btn">
                <input type="text" ref="focusinput" class="form-control" v-model="name" placeholder="标签分类名字" />
              </div>
            </td>
            <td>
              <div class="input-group-btn">
                <input type="number" v-model="weight" class="form-control" placeholder="分类权重,数字,越大排序越靠前" />
              </div>
            </td>
            <td>
              <div class="input-group-btn">
                <button 
                  type="submit" 
                  @click="send()" 
                  id="myButton" 
                  data-complete-text="添加成功" 
                  data-loading-text="提交中..." 
                  class="btn btn-default" 
                  autocomplete="off"
                >
                  添加分类
                </button>
              </div>
            </td>
            <td>
            </td>
            <td>
            </td>
          </tr>
          <tr>
            <td colspan="5">
                 <span class="text-danger" v-show="error">{{ error }}</span>
            </td>
          </tr>
        </tbody>
      </table>
      <ul class="pager" v-show="page >= 1 && last_page > 1">
        <li class="previous" :class="page <= 1 ? 'disabled' : ''">
          <router-link :to="{ path: '/setting/tag-categories', query: prevQuery }">
            <span aria-hidden="true">&larr;</span>
            上一页
          </router-link>
        </li>
        <li>
          共 {{ total }}个标签，总共{{ last_page }}页，当前为第{{ page }}页
        </li>
        <li class="next" :class="page >= last_page ? 'disabled': ''">
          <router-link :to="{ path: '/setting/tag-categories', query: nextQuery }">
            下一页
            <span aria-hidden="true">&rarr;</span>
          </router-link>
        </li>
      </ul>
      </div>
    </div>
  </div>
</template>
<script>
  import request, { createRequestURI } from '../../util/request';
  import plusMessageBundle from 'plus-message-bundle';
  const TagCategories = {
    data: () => ({
      tag_categories: [],
      last_page: 1,
      per_page: 20,
      page: 1,
      total: 1,
      loadding: true,
      error: null,
      message: '',
      name: '',
      weight: 0,
      edit: 0,
      edit_name: '',
      edit_cate: {},
      edit_weight: null,
      message: {
        error: null,
        success: null,
      },
    }),

    methods: {
      getTagCategories () {
        request.get(createRequestURI(`site/tags/tag_categories`) , {
          params: { ...this.queryParams }
        }, {
          validateStatus: status => status === 200
        }).then(({ data = {} }) => {
          this.tag_categories = data.data;
          this.last_page = data.last_page;
          this.page = data.current_page;
          this.total = data.total;
          this.loadding = false;
        }).catch(({ response: { data: { message = '加载失败' } = {} } = {} }) => {
          this.loadding = false;
          this.message.error = message;
        });
      },

      showEdit(id) {
        if(!id){
          return false;
        }

        this.edit = id;
        let index = _.findIndex(this.tag_categories, (cate) => {
          return cate.id === id;
        });
        this.edit_cate = { ...this.tag_categories[index] };
        this.edit_weight = this.edit_cate.weight;
        this.edit_name = this.edit_cate.name;
      },

      hideEdit() {
        this.edit = 0;
        this.edit_name = '';
        this.edit_cate = {};
        this.edit_weight = null;
      },

      // add tag cate
      send () {
        const { name, weight } = this;
        let btn = $("#myButton").button('loading');
        request.post(createRequestURI('site/tags/tag_categories'), {
          name,
          weight
        }, {
          validateStatus: status => status === 201
        })
        .then(({ data = {} }) => {
          const newCate = {
            id: data.id,
            tags_count: 0,
            name: name,
            weight: weight
          };
          btn.button('complete');
          setTimeout(() => {
            btn.button('reset');
            this.name = '',
            this.category = 0;
            this.weight = 0;
            this.tag_categories.unshift(newCate);
          }, 1500);
          this.error = '';
        })
        .catch(({ response: { data: { message = '添加分类失败' } = {} } = {} }) => {
          btn.button('reset');
          this.error = message;
        })
      },

      update() {
        const { edit_name, edit, edit_weight } = this;
        if(!edit_name || !edit ) {
          return false;
        }

        let data = {};
        if (edit_name != this.edit_cate.name) {
          data.name = edit_name;
        }
        if (edit_weight != this.edit_cate.weight) {
          data.weight = edit_weight;
        }
        this.dismisAddAreaError();
        request.patch(createRequestURI(`site/tags/tag_categories/${edit}`), {
          ...data
        }, {
          validateStatus: status => status === 201
        })
        .then(({ data = {} }) => {
          let index = _.findIndex(this.tag_categories, (cate) => {
            return cate.id === edit;
          });

          this.tag_categories[index].name = edit_name;
          this.tag_categories[index].weight = edit_weight;
          this.edit = 0;
          this.edit_name = '';
          this.edit_cate = {};
          this.message.success = '修改成功';
        })
        .catch(({ response: { data: { message = '修改失败' } = {} } = {} }) => {
          this.message.error = message;
        })
      },

      dismisAddAreaError () {
        let msg = this.message;
        msg.error = msg.success = null;
      },

      addFocus() {
        this.error = null;
        this.$refs.focusinput.focus();
      },

      // 删除分类
      deleteCate (id) {
        if(!id) {
          return false;
        }
        this.dismisAddAreaError();
        request.delete(createRequestURI(`site/tags/tag_categories/${id}`), {
          validateStatus: status => status === 204
        })
        .then( () => {
          let index = _.findIndex(this.tag_categories, cate => {
            return cate.id === id;
          });
          this.tag_categories.splice(index, 1);
          this.message.success = '删除成功';
        })
        .catch(({ response: { data: { message = '未知错误' } = {} } = {} }) => {
          this.message.error = message;
        });
      }
    },

    watch: {
      '$route' (to) {
        const {
          last_page = 1,
          per_page = 20,
          page = 1
        } = to.query;

        this.last_page = parseInt(last_page);
        this.per_page = parseInt(per_page);
        this.page = parseInt(page);

        this.getTagCategories();
      },
    },

    computed: {
      empty () {
        return !(this.tag_categories.length > 0);
      },
      canEditSend () {
        return ((this.edit_name != this.edit_cate.name) && this.edit_name != '') || (this.edit_weight !== null && (this.edit_weight != this.edit_cate.weight));
      },
      canSend() {
        return this.name.length > 0;
      },
      queryParams () {
        const { per_page, page } = this;
        return { per_page, page };
      },
      prevQuery () {
        const page = parseInt(this.page);
        return {
          ...this.queryParams,
          last_page: this.last_page,
          page: page > 1 ? page - 1 : page
        };
      },
      nextQuery () {
        const page = parseInt(this.page);
        const last_page = parseInt(this.last_page);
        return {
          ...this.queryParams,
          last_page: last_page,
          page: page < last_page ? page + 1 : last_page
        };
      },
    },

    created () {
      const {
        per_page = 20,
        last_page = 1,
        page = 1
      } = this.$route.query;
      
      this.last_page = last_page;
      this.current_page = page;
      this.per_page = per_page;

      this.getTagCategories();
    }
  }

  export default TagCategories;
</script>