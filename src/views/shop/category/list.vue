<template>
	<div class="bg-white px-3" style="margin: -20px;margin-top: -1rem;margin-bottom: 0!important;">

		<div class="py-2 border-bottom d-flex align-items-center">
			<el-button type="primary" size="mini" @click="createTop">创建顶级分类</el-button>
		</div>


		<el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick" default-expand-all :expand-on-click-node="false"
		 draggable @node-drop="nodeDrop" @node-drag-end="nodeDragEnd">

			<span class="custom-tree-node" slot-scope="{ node, data }">
				<div>
					<el-input v-if="data.editStatus" v-model="data.name" size="mini"></el-input>
					<span v-else>{{ node.label }}</span>
				</div>
				<span>
					<el-button v-if="data.category_id == 0" type="text" size="mini" @click="connectGoods(data)">
						推荐产品
					</el-button>
					<el-button :type="data.status ? 'primary' : 'danger'" size="mini" @click.stop="showOrHide(data)">
						{{data.status ? '显示' : '隐藏'}}
					</el-button>
					<el-button type="success" size="mini" @click="append(data)">增加</el-button>
					<el-button :type="data.editStatus ? 'default':'warning'" size="mini" @click="edit(data)">{{data.editStatus ? '完成':'修改'}}</el-button>
					<el-button type="danger" size="mini" @click="remove(node,data)">删除</el-button>
				</span>
			</span>

		</el-tree>

		
		<el-dialog title="推荐产品" :visible.sync="createModel">
			<el-table
			    :data="tableData"
			    height="250"
			    border
			    style="width: 100%">
			    <el-table-column
			      prop="goods_id"
			      label="商品ID"
			      width="60">
			    </el-table-column>
			    <el-table-column
			      prop="cover"
			      label="商品封面"
			      width="180">
				  <template slot-scope="scope">
				  	 <img :src="scope.row.cover" style="width: 64px;height: 64px;" />
				  </template>
			    </el-table-column>
				<el-table-column
				  prop="name"
				  label="商品名称"
				  width="180">
				</el-table-column>
			    <el-table-column
			      label="操作">
				  <template slot-scope="scope">
					  <el-button type="danger" size="small" @click="deleteCategoryItem(scope)">删除</el-button>
				  </template>
			    </el-table-column>
			  </el-table>
			  <div slot="footer" class="dialog-footer">
			  	<el-button size="mini" @click="createModel = false">取 消</el-button>
			  	<el-button size="mini" type="primary" @click="connectCategoryItem">关 联</el-button>
			  </div>
		</el-dialog>

		<goods-dialog ref="goodsDialog"></goods-dialog>

	</div>
</template>

<script>
	import goodsDialog from '@/components/shop/goods-dialog.vue';
	export default {
		components: {
			goodsDialog
		},
		inject:['layout'],
		data() {
			return {
				data: [],
				defaultProps: {
					children: 'child',
					label: 'name'
				},
				createModel:false,
				connect:{
					category_id:0
				},
				tableData: []
			};
		},
		created() {
			this.__init()
		},
		computed: {
			// 排序后的数据
			sortData() {
				let data = []
				let sort = function(arr){
					arr.forEach(item=>{
						data.push(item)
						if(item.child.length){
							sort(item.child)
						}
					})
				}
				// 多维数组转一维数组
				sort(this.data)
				// 排序
				data = data.map((item,index)=>{
					return {
						id:item.id,
						order:index,
						category_id:item.category_id
					}
				})
				return data
			}
		},
		methods: {
			connectCategoryItem(){
				this.$refs.goodsDialog.chooseGoods((res)=>{
					this.axios.post(`/admin/app_category_item`,{
						category_id:this.connect.category_id,
						goods_ids:res.map(o=>o.id)
					},{
						token:true
					}).then(res=>{
						this.getCategoryItems()
						this.$message({
							message: '关联成功',
							type: 'success'
						});
						this.layout.hideLoading()
					}).catch(err=>{
						this.layout.hideLoading()
					})
				},10)
			},
			deleteCategoryItem(scope){
				let { row,$index } = scope
				this.$confirm('是否要删除该关联?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					this.layout.showLoading()
					this.axios.post(`/admin/app_category_item/${row.id}/delete`,{},{
						token:true
					}).then(res=>{
						this.tableData.splice($index,1)
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.layout.hideLoading()
					}).catch(err=>{
						this.layout.hideLoading()
					})
				})
			},
			connectGoods(data){
				this.connect.category_id = data.id
				this.getCategoryItems()
			},
			getCategoryItems(){
				this.layout.showLoading()
				this.axios.get('/admin/app_category_item/list?category_id='+this.connect.category_id,{
					token:true
				}).then(res=>{
					this.tableData = res.data.data
					this.layout.hideLoading()
					this.createModel = true
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			// 初始化
			__init() {
				this.layout.showLoading()
				this.axios.get('/admin/category',{
					token:true
				}).then(res=>{
					let data = res.data.data
					let addEditStatus = function(arr){
						arr.forEach(item=>{
							item.editStatus = false
							if(item.child.length){
								addEditStatus(item.child)
							}
						})
					}
					addEditStatus(data)
					this.data = data
					this.layout.hideLoading()
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			handleNodeClick(data) {
				console.log(data);
			},
			// 显示/隐藏
			showOrHide(data) {
				this.layout.showLoading()
				let status = data.status ? 0 : 1
				let msg = status ? '显示' : '隐藏'
				this.axios.post('/admin/category/'+data.id+'/update_status',{ status },{ token:true }).then(res=>{
					data.status = status
					this.layout.hideLoading()
					this.$message({
						message: msg+'成功',
						type: 'success'
					});
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			// 编辑|提交
			edit(data) {
				if(!data.editStatus){
					return data.editStatus = true
				}
				if(data.name == ''){
					return this.$message({
						message: '分类名称不能为空',
						type: 'error'
					});
				}
				this.layout.showLoading()
				this.axios.post('/admin/category/'+data.id,{
					status:data.status,
					name:data.name,
					category_id:data.category_id
				},{
					token:true
				}).then(res=>{
					this.$message({
						message: '修改成功',
						type: 'success'
					});
					data.editStatus = false
					this.layout.hideLoading()
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			// 删除
			remove(node, data) {
				this.$confirm('此操作将删除该分类, 是否继续?', '提示', {
					confirmButtonText: '删除',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					this.layout.showLoading()
					this.axios.delete('/admin/category/'+data.id,{
						token:true
					}).then(res=>{
						this.__init()
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.layout.hideLoading()
					}).catch(err=>{
						this.layout.hideLoading()
					})
				})

			},
			// 增加子分类
			append(data) {
				this.layout.showLoading()
				this.axios.post('/admin/category',{
					status:0,
					name:"新分类",
					category_id:data.id
				},{
					token:true
				}).then(res=>{
					let obj = res.data.data
					obj.editStatus = true
					obj.child = []
					data.child.push(obj)
					this.layout.hideLoading()
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			// 拖拽结束
			nodeDragEnd(...options){
				// 被拖拽节点对应的数据
				let item = options[0].data
				// 结束拖拽时最后进入的节点数据
				let obj = options[1].data
				if(obj){
					if(options[2] === 'before' || options[2] === "after"){
						item.category_id = obj.category_id
					} else {
						item.category_id = obj.id
					}
				}
			},
			// 拖拽
			nodeDrop(...options) {
				this.layout.showLoading()
				this.axios.post('/admin/category/sort',{
					sortdata:JSON.stringify(this.sortData)
				},{
					token:true
				}).then(res=>{
					this.__init()
					this.layout.hideLoading()
				}).catch(err=>{
					this.layout.hideLoading()
				})
			},
			// 创建顶级分类
			createTop() {
				this.$prompt('请输入顶级分类名称', '提示', {
					confirmButtonText: '创建',
					cancelButtonText: '取消',
					inputValidator: function(val) {
						if (val === '' || val === null) {
							return '顶级分类名称不能为空'
						}
						return true
					}
				}).then(({
					value
				}) => {
					this.layout.showLoading()
					this.axios.post('/admin/category',{
						status:0,
						category_id:0,
						name:value
					},{
						token:true
					}).then(res=>{
						this.__init()
						this.$message({
							message: '创建成功',
							type: 'success'
						});
						this.layout.hideLoading()
					}).catch(err=>{
						this.layout.hideLoading()
					})
				})
			}

		}
	}
</script>

<style>
	.custom-tree-node {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 14px;
		padding-right: 8px;
	}

	.el-tree-node__content {
		padding: 20px 0;
	}
</style>
