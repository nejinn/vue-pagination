1、直接引入的vue.js做的分页。
2、分页式样用的datetable，因为之前用datatable写分页，发现搞不明白他，就自己写了一个。
3、由于后端是django，前端模板有django模板语言，去掉就行。
4、不懂得欢迎咨询

效果如下：
![GIF.gif](https://upload-images.jianshu.io/upload_images/9915084-84e077c2f436e4b1.gif?imageMogr2/auto-orient/strip)

html
```
{% extends 'ssityadmin/base.html' %}
{% load static %}

{% block title %}
SsityaAdmin | 访客详情
{% endblock title %}

{% block css %}
<link rel="stylesheet" href="{% static "bower_components/datatables.net-bs/css/dataTables.bootstrap.min.css" %}">
{% endblock css %}

{% block content-wrapper %}
    <section class="content-header">
        <h1>
            访客详情
            <small>表单数据</small>
        </h1>
        <ol class="breadcrumb">
            <li><a><i class="fa fa-street-view"></i> SsityAdmin</a></li>
            <li class="active">访客详情</li>
        </ol>
    </section>

    <section class="content">
        <div class="row">
            <div class="col-xs-12">
                <div class="box">
                    <div class="box-header">
                        <h3 class="box-title">访客数据表</h3>
                    </div>
                    <!-- /.box-header -->
                    <div class="box-body" id="visitor-table">
                        <div id="example1_wrapper" class="dataTables_wrapper form-inline dt-bootstrap">
                            <div class="row">
                                <div class="col-sm-6">
                                    <div class="dataTables_length" id="example1_length">
                                        <label>选择每页显示
                                            <select v-model="size" class="form-control input-sm" @change="changesize">
                                                <option value="10">10</option>
                                                <option value="20">20</option>
                                                <option value="30">30</option>
                                                <option value="40">40</option>
                                                <option value="50">50</option>
                                            </select> 条
                                        </label>
                                    </div>
                                </div>

                            </div>
                            <div class="row">
                                <div class="col-sm-12">
                                    <table class="table table-bordered table-striped dataTable" >
                                        <thead>
                                        <tr class="odd">
                                            <th class="dataTables_empty" >id</th>
                                            <th class="dataTables_empty" >ip</th>
                                            <th class="dataTables_empty" >country</th>
                                            <th class="dataTables_empty" >province</th>
                                            <th class="dataTables_empty" >city</th>
                                            <th class="dataTables_empty" >time</th>
                                        </tr>
                                        </thead>
                                        <!-- 加载数据显示过度 -->
                                        <tbody v-show="loaddata">
                                        <tr class="odd" >
                                            <td valign="top" colspan="6" class="dataTables_empty" style="color: #e23838;">数 据 加 载 中
                                            </td>
                                        </tr>
                                        </tbody>
                                        <!-- 无数据显示 -->
                                        <tbody v-show="tabledata == undefined || tabledata.length <= 0">
                                        <tr class="odd" v-show="!loaddata">
                                            <td valign="top" colspan="6" class="dataTables_empty" style="color: #0925ef;">暂 无 数 据
                                            </td>
                                        </tr>
                                        </tbody>
                                        <tbody v-for="data in tabledata">

                                        <tr class="odd">
                                            {% verbatim %}
                                            <td class="dataTables_empty" >{{ data.id }}</td>
                                            <td class="dataTables_empty" >{{ data.ip }}</td>
                                            <td class="dataTables_empty" >{{ data.country }}</td>
                                            <td class="dataTables_empty" >{{ data.province }}</td>
                                            <td class="dataTables_empty" >{{ data.city }}</td>
                                            <td class="dataTables_empty" >{{ data.time }}</td>
                                            {% endverbatim %}
                                        </tr>
                                        </tbody>

                                    </table>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-sm-5">
                                    {% verbatim %}
                                    <div class="dataTables_info">显示{{ size }}条一页/共{{ count }}条</div>
                                    {% endverbatim %}
                                </div>
                                <div class="col-sm-7">

                                    <div class="dataTables_paginate paging_simple_numbers">
                                        <ul class="pagination">
                                            <li class="page" v-show="currentPage !== 1 " @click="prevOrNext(-1)"><span class="fa fa-chevron-left" aria-hidden="true"></span></li>
                                            <li class="paginate_button" v-show="tabledata !==undefined && tabledata.length > 0" v-for="(item, index) in pages" :key="index" :class="{active: item === currentPage}" @click="select(item)">
                                                {% verbatim %}
                                                <span>{{item}}</span>
                                                {% endverbatim %}
                                            </li>
                                            <li class="page" v-show="currentPage !== totalPages" @click="prevOrNext(1)"><span class="fa fa-chevron-right" aria-hidden="true"></span></li>
                                        </ul>
                                    </div>

                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- /.box-body -->
                </div>
                <!-- /.box -->
            </div>
            <!-- /.col -->
        </div>
        <!-- /.row -->
    </section>



{% endblock content-wrapper %}

{% block js %}
<script src="{% static "vue/vue.min.js" %}"></script>
<script src="{% static "vue/axios.min.js" %}"></script>
<script>
var vm = new Vue({
    el: '#visitor-table',
    data:{
        'page': '',
        'size': '',
        'tabledata': '',
        'currentPage': 1,//定义默认页面为1
        'totalPages': 1,//定义默认总页面为1
        'count': '',
        'loaddata': true,
    },
    mounted () {

        // 获取cookie，并加载第一页数据
        var value = '; ' + document.cookie;
        var parts = value.split('; ' + 'csrftoken' + '=');
        if (parts.length === 2)
            var getCookie = parts.pop().split(';').shift();
        axios
      		.get('/ssityadmin/visitor/list/',{headers: {'X-CSRFToken': this.getCookie}})
      		.then((response) =>{
      		    vm.loaddata = false;
      			vm.tabledata = response.data.results;
      			vm.count = response.data.count;
      			console.log(vm.count);
      			vm.currentPage = 1;
      			vm.size = 10;
      			if (vm.count === 0){
      			    vm.totalPages = 1;
      			    console.log(vm.totalPages)
                }else{
      			    vm.totalPages = Math.ceil(vm.count/vm.size);
      			    console.log(vm.totalPages);
                }
      		})
  		},
    computed: {
        //计算显示的页面
        pages() {
            const c = this.currentPage;
            const t = this.totalPages;
            var left = 1;
			var right = this.totalPages;
			var arr = [];
			if(this.totalPages>=7){
				if(this.currentPage>4 && this.currentPage<this.totalPages-3){
					left = this.currentPage-3;
					right = this.currentPage+3;
				}else if(this.currentPage<=4){
                    left=1;
                    right=7;
				}else{
					left=this.totalPages-6;
					right=this.totalPages;
				}
			}
			while(left<=right){
				arr.push(left);
				left++;
			}
			return arr;
        }
    },
    methods: {
        //选择页面,并加载数据
        select(n) {
            if (n === this.currentPage) return;
            if (typeof n === 'string') return;
            this.currentPage = n;
            console.log(this.currentPage);
            vm.tabledata = '';
            vm.loaddata = true;
            var value = '; ' + document.cookie;
            var parts = value.split('; ' + 'csrftoken' + '=');
            if (parts.length === 2)
                var getCookie = parts.pop().split(';').shift();
            axios
                .get('/ssityadmin/visitor/list/' + '?page=' + this.currentPage + '&size=' + this.size, {headers: {'X-CSRFToken': this.getCookie}})
                .then((response) => {
                    vm.loaddata = false;
                    console.log(response);
                    vm.tabledata = response.data.results;
                    console.log(vm.tabledata);
                    vm.count = response.data.count;
                    console.log(vm.count);
                    if (vm.count === 0){
                        vm.totalPages = 1;
                        console.log(vm.totalPages)
                    }else{
                        vm.totalPages = Math.ceil(vm.count/vm.size);
                        console.log(vm.totalPages);
                    }
                })
        },
        //左右两边的< > ,b=并加载数据
        prevOrNext(n) {
            this.currentPage += n;
            console.log(this.currentPage);
            vm.tabledata = '';
            vm.loaddata = true;
            var value = '; ' + document.cookie;
            var parts = value.split('; ' + 'csrftoken' + '=');
            if (parts.length === 2)
                var getCookie = parts.pop().split(';').shift();
            axios
                .get('/ssityadmin/visitor/list/' + '?page=' + this.currentPage + '&size=' + this.size, {headers: {'X-CSRFToken': this.getCookie}})
                .then((response) => {
                    vm.loaddata = false;
                    console.log(response);
                    vm.tabledata = response.data.results;
                    console.log(vm.tabledata);
                    vm.count = response.data.count;
                    console.log(vm.count);
                    if (vm.count === 0){
                        vm.totalPages = 1;
                        console.log(vm.totalPages)
                    }else{
                        vm.totalPages = Math.ceil(vm.count/vm.size);
                        console.log(vm.totalPages);
                    }
                })
        },
        // 选择每条条数，并加载数据
        changesize:function(){
            vm.tabledata = '';
            vm.loaddata = true;
            var value = '; ' + document.cookie;
            var parts = value.split('; ' + 'csrftoken' + '=');
            if (parts.length === 2)
                var getCookie = parts.pop().split(';').shift();
            axios
                .get('/ssityadmin/visitor/list/' + '?page=' + this.currentPage + '&size=' + this.size, {headers: {'X-CSRFToken': this.getCookie}})
                .then((response) => {
                    if (response.status === 200){
                        vm.loaddata = false;
                        console.log(response.status);
                        console.log(response);
                        vm.tabledata = response.data.results;
                        console.log(vm.tabledata);
                        vm.count = response.data.count;
                        console.log(vm.count);
                        if (vm.count === 0){
                            vm.totalPages = 1;
                            console.log(vm.totalPages)
                        }else{
                            vm.totalPages = Math.ceil(vm.count/vm.size);
                            console.log(vm.totalPages);
                        }
                    }else{
                        vm.tabledata = '';
                        console.log(response.status);
                        alert('无效页面')
                    }
                })
                .catch(error => {
                    vm.tabledata = '';
                    vm.loaddata = false;
                    if (error.response.status === 404){
                        console.log(error.response.status);
                        alert('无效页面')
                    }else{
                        alert('无效页面')
                    }
                });

        }
    }
})
</script>
{% endblock js %}