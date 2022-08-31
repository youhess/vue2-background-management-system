<template>
	<el-dialog title="商品管理" :visible.sync="goodsModel" width="80%" top="5vh">

		<el-container style="position: relative;height: 70vh;margin: -30px -20px;">
			<el-header class="d-flex align-items-center border-bottom">
				<!-- 头部 -->
				<div class="d-flex mr-auto">
					<el-input class="mr-2" size="mini" placeholder="输入商品名称" v-model="searchForm.keyword"
						style="width: 150px;"></el-input>
					<el-button type="success" size="mini" @click="getGoodsList">搜索</el-button>
				</div>

			</el-header>
			<el-container>
				<el-main style="position: absolute;top: 60px;left:0;bottom: 60px;right: 0;" v-loading="mainLoading">
					<!-- 主内容 -->
					<el-row :gutter="10">
						<el-col :span="24" :lg="4" :md="6" :sm="8" v-for="(item,index) in goodsList" :key="index">
							<el-card class="box-card mb-3 position-relative" style="cursor: pointer;" :body-style="{'padding':'0'}" shadow="hover">
								<div class="border" :class="{'border-danger':item.ischeck}" @click="choose(item)">
									<span class="badge badge-danger" style="position: absolute;right: 0;top: 0;"
										v-if="item.ischeck">
										{{item.checkOrder}}</span>


									<img :src="item.cover" class="w-100" style="height: 100px;" />
									<div class="w-100 text-white px-1"
										style="background: rgba(0,0,0,0.5);margin-top: -25px;position: absolute;">
										<span class="small">{{item.title}}</span>
									</div>
									<div class="text-danger p-2">
										￥{{ item.min_price }}
									</div>
								</div>

							</el-card>
						</el-col>
					</el-row>

				</el-main>
			</el-container>
			<el-footer class="border-top d-flex align-items-center px-0">
				<!-- 底部 -->
				<div class="px-2">
					<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
						:current-page="currentPage" :page-sizes="pageSizes" :page-size="pageSize"
						layout="total, sizes, prev, pager, next, jumper" :total="total">
					</el-pagination>
				</div>
			</el-footer>
		</el-container>



		<div slot="footer" class="dialog-footer">
			<el-button @click="hide">取 消</el-button>
			<el-button type="primary" @click="confirm">确 定</el-button>
		</div>
	</el-dialog>
</template>

<script>
	export default {
		props: {
			// 选中数量限制
			max: {
				type: Number,
				default: 9
			},
		},
		data() {
			return {
				goodsModel: false,
				callback: false,

				searchForm: {
					order: "",
					keyword: ""
				},


				goodsList: [],
				// 选中的数组
				chooseList: [],
				currentPage: 1,
				pageSize: 10,
				pageSizes: [10, 20, 50, 100],
				total: 10,

				asideLoading: false,
				mainLoading: false
			}
		},
		computed: {
			// 当前选中相册的图片列表URL
			getGoodsListUrl() {
				let other = ''
				if (this.searchForm.keyword != '') {
					other = `&title=${this.searchForm.keyword}`
				}
				
				return `/admin/goods/${this.currentPage}?limit=${this.pageSize}&tab=saling${other}`
			}
		},
		methods: {
			// 打开弹出层
			chooseGoods(callback) {
				// 取消选中
				this.__init()
				this.callback = callback
				this.goodsModel = true
				this.unChoose()
			},
			// 确定
			confirm() {
				// 选中的图片url
				if (typeof this.callback === 'function') {
					this.callback(this.chooseList)
				}
				// 隐藏
				this.hide()
			},
			// 关闭弹出层
			hide() {
				this.goodsModel = false
				this.callback = false
			},


			// 取消选中
			unChoose() {
				this.goodsList.forEach(img => {
					// 找到所有选中的图片
					let i = this.chooseList.findIndex(item => {
						return item.id === img.id
					})
					if (i > -1) {
						// 取消选中样式，选中排序归零
						img.ischeck = false
						img.checkOrder = 0
						// 从chooseList中移除
						this.chooseList.splice(i, 1)
					}
				})
			},
			// 选中图片
			choose(item) {
				// 选中
				if (!item.ischeck) {
					// 限制选中数量
					if (this.chooseList.length >= this.max) {
						return this.$message({
							message: '最多选择' + this.max + '个商品',
							type: 'warning'
						});
					}
					// 加入选中
					this.chooseList.push({
						...item
					})
					// 计算序号
					item.checkOrder = this.chooseList.length
					// 修改状态
					item.ischeck = true
					return;
				}
				// 取消选中
				// 找到在chooseList中的索引，
				let i = this.chooseList.findIndex(v => v.id === item.id)
				if (i === -1) return;
				// 重新计算序号
				let length = this.chooseList.length
				// 取消选中中间部分
				if (i + 1 < length) {
					// 重新计算goodsList选中序号
					for (let j = i; j < length; j++) {
						let no = this.goodsList.findIndex(v => v.id === this.chooseList[j].id)
						if (no > -1) {
							this.goodsList[no].checkOrder--
						}
					}
				}
				// 删除
				this.chooseList.splice(i, 1)
				// 修改状态
				item.ischeck = false
				// 重置序号
				item.checkOrder = 0
			},
			__init() {
				// 获取相册列表
				this.getGoodsList()
			},
			// 获取对应相册下的图片列表
			getGoodsList() {
				this.mainLoading = true
				this.axios.get(this.getGoodsListUrl, {
					token: true
				}).then(res => {
					let result = res.data.data
					console.log(result);
					this.goodsList = result.list.map(item => {
						return {
							id: item.id,
							cover: item.cover,
							title: item.title,
							min_price:item.min_price,
							min_oprice:item.min_oprice,
							desc:item.desc,
							ischeck: false,
							checkOrder: 0
						}
					})
					this.total = result.totalCount
					this.mainLoading = false
				}).catch(err => {
					this.mainLoading = false
				})
			},
			handleSizeChange(val) {
				console.log(`每页 ${val} 条`);
				this.pageSize = val
				this.__init()
			},
			handleCurrentChange(val) {
				console.log(`当前页: ${val}`);
				this.currentPage = val
				this.__init()
			}
		},
	}
</script>

<style>
</style>
