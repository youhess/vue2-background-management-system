<template>
	<div class="bg-white px-3" style="margin: -20px;margin-top: -1rem;margin-bottom: 0!important;">
		<button-search class="pt-3" :showSearch="false">
		<!-- 左边 -->
			<template #left>
				<el-button size="mini" type="success" @click="openModel(false)">添加公告</el-button>
			</template>
		</button-search>
	
	
		<el-table border class="mt-3"
		  :data="tableData"
		  style="width: 100%">
		  <el-table-column
			label="公告标题"
			prop="title">
		  </el-table-column>
		  <el-table-column
			prop="create_time" 
			align="center"
			width="380"
			label="发布时间">
		  </el-table-column>
		  <el-table-column
			align="center"
			label="操作"
			width="150">
			<template slot-scope="scope">
				<el-button-group>
				  <el-button type="primary" size="mini" 
				  plain @click="openModel(scope)">修改</el-button>
				  <el-button type="danger" size="mini" 
				  plain @click="deleteItem(scope.row)"
				  >删除</el-button>
				</el-button-group>
			</template>
		  </el-table-column>
		</el-table>
		<div style="height: 60px;"></div>
		<el-footer class="border-top d-flex align-items-center px-0 position-fixed bg-white" style="bottom: 0;left: 200px;right: 0;z-index: 100;">
		  <div style="flex: 1;" class="px-2">
			  <el-pagination
			  :current-page="page.current"
			  :page-sizes="page.sizes"
			  :page-size="page.size"
			  layout="total, sizes, prev, pager, next, jumper"
			  :total="page.total"
			  @size-change="handleSizeChange"
			  @current-change="handleCurrentChange">
			</el-pagination>
		  </div>
		</el-footer>
<!-- 新增/修改模态框 -->
<el-dialog :title="editIndex > -1 ? '修改公告' : '添加公告'"  :visible.sync="createModel" top="5vh">
	<!-- 表单内容 -->
	<el-form :rules="rules" ref="form" :model="form" label-width="80px">
		<el-form-item label="公告标题" prop="title">
			<el-input v-model="form.title" placeholder="公告标题" size="mini"></el-input>
		</el-form-item>
		<el-form-item label="公告内容" prop="content">
			<el-input type="textarea" v-model="form.content" size="mini" placeholder="公告内容" :rows="5"></el-input>
		</el-form-item>
	</el-form>
	
	
	<div slot="footer" class="dialog-footer">
		<el-button @click="createModel = false">取 消</el-button>
		<el-button type="primary" @click="submit">确 定</el-button>
	</div>
</el-dialog>


	</div>
</template>

<script>
	import buttonSearch from "@/components/common/button-search.vue"
	import common from '@/common/mixins/common.js';
	export default {
		inject:['layout'],
		mixins:[common],
		components: {
			buttonSearch
		},
		data() {
			return {
				preUrl:"notice",
				
				tableData: [],
				
				createModel:false,
				editIndex:-1,
				
				form:{
					title:"",
					content:""
				},
				rules:{
					title:[{ 
						required:true,
						message:"公告标题不能为空",
						trigger:'blur' ,
					}],
					content:[{
						required:true,
						message:"公告内容不能为空",
						trigger:'blur' ,
					}],
				}
			}
		},
		methods: {
			getListResult(e){
				this.tableData = e.list
			},
			// 打开模态框
			openModel(e = false){
				// 增加
				if (!e) {
					// 初始化表单
					this.form = {
						title:"",
						content:""
					}
					this.editIndex = -1
				} else {
					// 修改
					this.form = {
						title:e.row.title,
						content:e.row.content,
					}
					this.editIndex = e.$index
				}
				// 打开dialog
				this.createModel = true
			},
			// 添加规格
			submit(){
				this.$refs.form.validate(res=>{
					if (res) {
						let id = 0
						if (this.editIndex !== -1) {
							id = this.tableData[this.editIndex].id
						}
						this.addOrEdit(this.form,id)
						// 关闭模态框
						this.createModel = false
					}
				})
			},
		},
	}
</script>

<style>
</style>
