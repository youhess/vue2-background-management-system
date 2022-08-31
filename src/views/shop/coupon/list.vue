<template>
	<div class="bg-white px-3 pt-1" style="margin: -20px;margin-top: -1rem;margin-bottom: 0!important;">
		<button-search :showSearch="false">
		<!-- 左边 -->
			<template #left>
				<el-button size="mini" type="success" @click="openModel(false)">添加优惠券</el-button>
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
			label="优惠券名称"
			prop="name">
		  </el-table-column>
		<el-table-column
			align="center"
			prop="type"
			label="类型">
			<template slot-scope="scope">
				{{ scope.row.type ? '满减' : '折扣' }}
			</template>
		</el-table-column>
		<el-table-column
			align="center"
			prop="value"
			label="优惠金额/折扣">
		</el-table-column>
		<el-table-column
			align="center"
			prop="total"
			label="发放数量">
		</el-table-column>
		<el-table-column
			align="center"
			prop="used"
			label="已使用">
		</el-table-column>
		<el-table-column
			align="center"
			prop="start_time"
			label="开始时间">
		</el-table-column>
		<el-table-column
			align="center"
			prop="end_time"
			label="结束时间">
		</el-table-column>
		<el-table-column
			align="center"
			prop="status"
			label="状态">
			<template slot-scope="scope">
				{{ scope.row | getStatus }}
			</template>
		</el-table-column>
		<el-table-column
			align="center"
			label="操作"
			width="150">
			<template slot-scope="scope">
				<el-button-group v-if="showBtn(scope.row)">
				  <el-button v-if="showEditBtn(scope.row)" type="primary" size="mini" plain @click="openModel(scope)">修改</el-button>
				  <el-button type="danger" size="mini" plain @click="changeStatus(scope.row)">失效</el-button>
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
		
		<!-- 新增/修改模态框 -->
		<el-dialog :title="editIndex > -1 ? '修改优惠券' : '添加优惠券'" 
		:visible.sync="createModel" top="5vh">
			<!-- 表单内容 -->
			<el-form :rules="rules" ref="form" :model="form" label-width="100px">
				<el-form-item label="优惠券名称" prop="name">
					<el-input v-model="form.name" placeholder="优惠券名称" size="mini" style="width: 30%;"></el-input>
				</el-form-item>
				<el-form-item label="类型">
					<el-radio-group v-model="form.type" size="mini">
						<el-radio :label="0" border>满减</el-radio>
						<el-radio :label="1" border>折扣</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="面值">
					<el-input type="number" size="mini" v-model="form.value" class="w-25">
						<template slot="append">{{ form.type ? '折' : '元' }}</template>
					</el-input>
				</el-form-item>
				<el-form-item label="发行量">
					<el-input-number size="mini" v-model="form.total" :min="0"></el-input-number>
				</el-form-item>
				<el-form-item label="最低使用价格">
					<el-input-number size="mini" v-model="form.min_price" :min="0"></el-input-number>
				</el-form-item>
				<el-form-item label="排序">
					<el-input-number size="mini" v-model="form.order" :min="0"></el-input-number>
				</el-form-item>
				<el-form-item label="状态">
					<el-radio-group v-model="form.status" size="mini">
						<el-radio :label="1" border>启用</el-radio>
						<el-radio :label="0" border>禁用</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="活动时间">
					<el-date-picker v-model="timerange" type="datetimerange" align="right" start-placeholder="开始日期" end-placeholder="结束日期"
					value-format="yyyy-MM-dd HH:mm:ss" :default-value="(new Date())">
					</el-date-picker>
				</el-form-item>
				<el-form-item label="描述" prop="desc">
					<el-input
					  type="textarea"
					  :rows="3"
					  v-model="form.desc">
					</el-input>
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
	let formatStatus = (row)=>{
		let s = '领取中'
		let start = (new Date(row.start_time)).getTime()
		let now = (new Date()).getTime()
		let end = (new Date(row.end_time)).getTime()
		if(start > now){
			s = '未开始'
		} else if(end < now){
			s = '已结束'
		}
		if(row.status == 0){
			s = '已失效'
		}
		return s;
	}
	export default {
		components: {
			buttonSearch
		},
		mixins:[common],
		inject:['layout'],
		filters: {
			getStatus(row) {
				return formatStatus(row)
			},
		},
		data() {
			return {
				preUrl:"coupon",
				
				tableData: [],
				currentPage:1,
				multipleSelection: [],
				
				createModel:false,
				editIndex:-1,

				form:{
					name:"",
					type:0,
					value:0,
					total:100,
					min_price:0,
					start_time:"",
					end_time:"",
					status:1,
					order:50,
					desc:""
				},
				rules:{
					name:[{ 
						required:true,
						message:"优惠券名称不能为空",
						trigger:'blur' ,
					}],
				}
			}
		},
		computed: {
			timerange: {
				get() {
					if(this.form.start_time){
						return [this.form.start_time, this.form.end_time]
					}
					return []
				},
				set(val) {
					this.form.start_time = val[0]
					this.form.end_time = val[1]
				}
			}
		},
		methods: {
			showEditBtn(row){
				let s = formatStatus(row)
				return s == '未开始'
			},
			showBtn(row){
				let s = formatStatus(row)
				return s == '未开始' || s == '领取中'
			},
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
			// 打开模态框
			openModel(e = false){
				// 增加
				if (!e) {
					// 初始化表单
					this.form = {
						name:"",
						type:0,
						value:0,
						total:100,
						min_price:0,
						start_time:"",
						end_time:"",
						status:1,
						order:50,
						desc:""
					}
					this.editIndex = -1
				} else {
					// 修改
					const r = e.row
					this.form = {
						name:r.name,
						type:r.type,
						value:r.value,
						total:r.total,
						min_price:r.min_price,
						start_time:r.start_time,
						end_time:r.end_time,
						status:r.status,
						order:r.order,
						desc:r.desc,
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
						const d = JSON.parse(JSON.stringify(this.form))
						d.start_time = d.start_time ? ((new Date(d.start_time)).getTime())/1000 : ''
						d.end_time = d.end_time ? ((new Date(d.end_time)).getTime())/1000 : ''
						this.addOrEdit(d,id)
						// 关闭模态框
						this.createModel = false
					}
				})
			},
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
