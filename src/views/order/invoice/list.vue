<template>
	<div class="bg-white px-3 pt-1" style="margin: -20px;margin-top: -1rem;margin-bottom: 0!important;">
		<button-search :showSearch="false">
		<!-- 左边 -->
			<template #left>
				<el-button type="danger" size="mini" @click="deleteAll">批量删除</el-button>
			</template>
		</button-search>
		
		<el-table border class="mt-3"
		  :data="tableData"
		  style="width: 100%"
		   @selection-change="handleSelectionChange">
		  <el-table-column
			type="selection"
			width="45"
			align="center">
		  </el-table-column>
		  
		  <el-table-column
			label="订单编号"
			prop="order_no">
		  </el-table-column>
		 
		<el-table-column
			align="center"
			prop="order_status"
			label="订单状态">
		</el-table-column>
		 
		<el-table-column
			align="center"
			prop="price"
			label="开票金额">
		</el-table-column>
		 
		  <el-table-column
			align="center"
			prop="username"
			label="用户名">
		  </el-table-column>
		  
		  <el-table-column
			align="left"
			label="抬头"
			width="300">
			<template slot-scope="{ row }">
				<div v-if="row.type == 0">
					<small class="d-block">姓名：{{ row.name }}</small>
					<small class="d-block">手机：{{ row.phone }}</small>
					<small class="d-block">邮箱：{{ row.email }}</small>
				</div>
				<div v-else>
					<small class="d-block">公司名称：{{ row.name }}</small>
					<small class="d-block">税号：{{ row.code }}</small>
					<small class="d-block">单位地址：{{ row.path }}</small>
					<small class="d-block">开户行：{{ row.bankname }}</small>
					<small class="d-block">银行账号：{{ row.bankno }}</small>
					<small class="d-block">手机：{{ row.phone }}</small>
					<small class="d-block">邮箱：{{ row.email }}</small>
				</div>
			</template>
		  </el-table-column>
		  
		  <el-table-column
			align="center"
			label="发票状态">
			<template slot-scope="scope">
				{{ scope.row.status ? '已开票' : '未开票' }}
			</template>
		  </el-table-column>
		  
		  <el-table-column
			align="center"
			prop="create_time"
			label="申请时间">
		  </el-table-column>
		  
		  <el-table-column
			align="center"
			prop="update_time"
			label="开票时间">
		  </el-table-column>
		  
		  <el-table-column
			align="center"
			label="操作"
			width="150">
			<template slot-scope="scope">
				<el-button-group v-if="!scope.row.status">
				  <el-button type="primary" size="mini" plain @click="submit(scope.row)">开票</el-button>
				</el-button-group>
				<span v-else>暂无操作</span>
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
	</div>
</template>

<script>
	import buttonSearch from "@/components/common/button-search.vue"
	import common from '@/common/mixins/common.js';
	export default {
		components: {
			buttonSearch
		},
		mixins:[common],
		inject:['layout'],
		data() {
			return {
				preUrl:"invoice",
				
				tableData: [],
				currentPage:1,
				multipleSelection: [],
			}
		},
		created() {

		},
		methods: {
			// 获取请求列表分页url
			getListUrl(){
				return `/admin/${this.preUrl}/${this.page.current}?limit=${this.page.size}`
			},
			// 处理获取列表结果
			getListResult(e){
				console.log(e);
				this.tableData = e.list
			},
			// 选中
			handleSelectionChange(val) {
				this.multipleSelection = val;
			},
			submit(row){
				this.$confirm('是否要开票?', '提示', {
					confirmButtonText: '开票',
					cancelButtonText: '取消',
					type: 'info'
				}).then(() => {
					this.layout.showLoading()
					this.axios.post('/admin/invoice/'+row.id,{},{
						token:true,
					}).then(res=>{
						this.$message.success("开票成功")
						this.getList()
					}).finally(()=>{
						this.layout.hideLoading()
					})
				})
			}
		},
	}
</script>

<style>
.sku-list-item>i{
	display: none;
	cursor: pointer;
}
.sku-list-item:hover{
	background-color: #f4f4f4;
}
.sku-list-item:hover>font{
	display: none;
}
.sku-list-item:hover>i{
	display: inline-block;
}
</style>
