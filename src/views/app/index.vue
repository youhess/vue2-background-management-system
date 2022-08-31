<template>
	<div>
		<div style="margin-top: 20px">
		    <el-radio-group v-model="app_index_category_id" size="small" @change="getData">
		      <el-radio-button v-for="(item,index) in appIndexCategory" :key="index" :label="item.id">{{ item.name }}</el-radio-button>
		    </el-radio-group>
		</div>
		
		<div class="row">
			<div class="col-5">
				<div class="card" v-loading="cardLoading">
					<div class="card-header bg-white">
						<el-dropdown style="float: right; padding: 3px 0">
						  <el-button type="text">添加模块</el-button>
						  <el-dropdown-menu slot="dropdown">
						    <el-dropdown-item v-for="(item,index) in moduleList" :key="index" @click.native="addData(item)">{{ item.name }}</el-dropdown-item>
						  </el-dropdown-menu>
						</el-dropdown>
					</div>
					<div class="card-body p-3">
						<div class="text-center py-5 text-muted" v-if="!indexList.length">
							暂无数据
						</div>
						<div class="p-3 border mb-2 app-index-item" v-for="(l,li) in indexList" :key="li" :class="activeId === l.id ? 'app-index-item-active' : ''" @click="choose(l.id)" style="position: relative;">
							<!-- 轮播图组件 -->
							<el-carousel height="150px" v-if="l.type === 'swiper'">
								<div class="w-100 text-muted text-center py-5" v-if="!l.data[0]">
									点击添加轮播图
								</div>
								<el-carousel-item v-for="(item,index) in l.data" :key="index">
									<img :src="item.src" style="width: 100%;">
								</el-carousel-item>
							</el-carousel>
							
							<!-- 图标分类组件 -->
							<div v-else-if="l.type === 'indexnavs'" class="d-flex flex-wrap">
								<div class="w-100 text-muted text-center py-5" v-if="!l.data[0]">
									点击添加图标分类
								</div>
								<div v-for="(item,index) in l.data" :key="index" style="width: 20%;" class="d-flex flex-column align-items-center justify-content-center">
									<img :src="item.src" style="width: 35px;height: 35px;"/>
									<span class="my-2" style="font-size: 10px;">{{item.text}}</span>
								</div>
							</div>
							<!-- 三图广告位 threeAdv-->
							<div v-else-if="l.type === 'threeAdv'" class="d-flex flex-wrap">
								<div class="w-100 text-muted text-center py-5" v-if="!l.data[0]">
									点击添加图片广告
								</div>
								<img v-if="l.data[0]" :src="l.data[0].src" style="width: 50%;">
								<div style="width: 50%;" class="d-flex flex-column">
									<img v-if="l.data[1]" :src="l.data[1].src" style="flex: 1;width: 100%;">
									<img v-if="l.data[2]" :src="l.data[2].src" style="flex: 1;width: 100%;">
								</div>
							</div>
							<!-- 大图广告位 -->
							<el-card class="box-card" v-else-if="l.type === 'oneAdv'">
								<div slot="header" class="clearfix">
									<span class="small">{{l.data.title}}</span>
								</div>
								<img :src="l.data.cover" style="width: 100%;">
							</el-card>
							
							<!-- 商品列表 -->
							<div v-else-if="l.type === 'list'" class="row">
								<div class="w-100 text-muted text-center py-5" v-if="!l.data[0]">
									点击添加商品
								</div>
								<div class="col-6 px-1" v-for="(item,index) in l.data" :key="index">
									<div class="border mb-2">
										<img :src="item.cover" style="width: 100%;height: 190px;" class="bg-light">
										<div class="p-1">
											<h6 class="mb-1">{{item.title}}</h6>
											<p class="small mb-1 text-secondary">{{item.desc}}</p>
											<span class="text-danger">￥{{item.pprice}}</span>
											<span class="text-muted ml-2 small">￥{{item.oprice}}</span>
											
										</div>
									</div>
								</div>
							</div>
							
							<!-- 按钮 -->
							<div v-if="activeId === l.id" class="app-index-icon-box">
								<span class="border app-index-icon-btn" :class="li === 0 ? 'app-index-icon-btn-disabled' : ''" @click="moveUp(li)">
									<i class="el-icon-arrow-up"></i>
								</span>
								<span class="border app-index-icon-btn" :class="(li === (indexList.length - 1)) || indexList.length === 0 ? 'app-index-icon-btn-disabled' : ''" @click="moveDown(li)">
									<i class="el-icon-arrow-down"></i>
								</span>
								<span class="border app-index-icon-btn" @click="deleteItem(li)">
									<i class="el-icon-delete"></i>
								</span>
							</div>
							
						</div>
						
						
						
					</div>
				</div>

			</div>
			<div class="col-7 pl-5">
				<div class="card" v-if="editItem">
					<div class="card-header bg-white">
						<span>配置</span>
						<el-button style="float: right;" type="primary" size="mini" @click="submit">保存</el-button>
					</div>
					<div class="card-body p-3">
						<!-- 配置轮播图 -->
						<form v-if="editItem.type === 'swiper'">
						  <div class="form-group"
						  v-for="(item,index) in swiper.data"
						  :key="index">
								<div class="card">
									<div class="card-header">
										<el-button style="float: right; padding: 3px 0" type="text" @click="deleteFormItem(index)">删除</el-button>
									</div>
									<div class="card-body p-0">
										<div style="width: 100%;height: 150px;cursor: pointer;position: relative;" class="d-flex align-items-center justify-content-center mr-3 mb-3" @click="chooseImage(index)">
											<img v-if="item.src" :src="item.src" style="height: 100%;"/>
											<i v-else class="el-icon-plus text-muted" style="font-size: 30px;"></i>
										</div>
									</div>
								</div>
							
						  </div>
						</form>
						
						<!-- 配置图标分类 -->
						<form v-if="editItem.type === 'indexnavs'">
						  <div class="form-group"
						  v-for="(item,index) in indexnavs.data"
						  :key="index">
								<div class="card">
									<div class="card-header">
										<el-button style="float: right; padding: 3px 0" type="text" @click="deleteFormItem(index)">删除</el-button>
									</div>
									<div class="card-body p-3 d-flex align-items-center">
										<div style="width: 35px;height: 35px;cursor: pointer;position: relative;" class="d-flex align-items-center justify-content-center mr-3" @click="chooseImage(index)">
											<img v-if="item.src" :src="item.src" style="width: 35px;height: 35px;"/>
											<i v-else class="el-icon-plus text-muted" style="font-size: 30px;"></i>
										</div>
										<input type="text" class="form-control" v-model="item.text" placeholder="填写分类名称">
									</div>
								</div>
							
						  </div>
						</form>
						
						<!-- 三图广告 -->
						<form v-if="editItem.type === 'threeAdv'">
						  <div class="form-group"
						  v-for="(item,index) in threeAdv.data"
						  :key="index">
								<div class="card">
									<div class="card-body p-0">
										<div style="width: 100%;height: 150px;cursor: pointer;position: relative;" class="d-flex align-items-center justify-content-center mr-3 mb-3" @click="chooseImage(index)">
											<img v-if="item.src" :src="item.src" style="height: 100%;"/>
											<i v-else class="el-icon-plus text-muted" style="font-size: 30px;"></i>
										</div>
									</div>
								</div>
							
						  </div>
						</form>
						<!-- 大图广告 -->
						<form v-if="editItem.type === 'oneAdv'">
						  <div class="form-group">
								<div class="card">
									<div class="card-body p-0">
										<div style="width: 100%;height: 150px;cursor: pointer;position: relative;" class="d-flex align-items-center justify-content-center mr-3 mb-3" @click="chooseImage()">
											<img v-if="oneAdv.cover" :src="oneAdv.cover" style="height: 100%;"/>
											<i v-else class="el-icon-plus text-muted" style="font-size: 30px;"></i>
										</div>
										<input type="text" class="form-control" placeholder="填写标题" v-model="oneAdv.title">
									</div>
								</div>
							
						  </div>
						</form>
						<!-- 商品列表 -->
						<form v-if="editItem.type === 'list'">
						  <div class="form-group">
							<div class="row">
								<div class="col-6 px-1" v-for="(item,index) in list.data" :key="index">
									<div class="border mb-2 position-relative">
										<span class="border bg-white px-2 py-1" style="position: absolute;right: 0;top: 0;cursor: pointer;" @click="deleteFormItem(index)"><i class="el-icon-delete"></i></span>
										<img :src="item.cover" style="width: 100%;height: 280px;" class="bg-light">
										<div class="p-1">
											<h6 class="mb-1">{{item.title}}</h6>
											<p class="small mb-1 text-secondary">{{item.desc}}</p>
											<span class="text-danger">￥{{item.pprice}}</span>
											<span class="text-muted ml-2 small">￥{{item.oprice}}</span>
											
										</div>
									</div>
								</div>
							</div>
							
						  </div>
						</form>
						
						<button v-if="editItem.type !== 'oneAdv'" type="submit" class="btn btn-outline-secondary w-100" @click="addFormItem" :disabled="buttonItem.data.length === buttonItem.limit">+ 添加 {{buttonItem.data.length}}/{{buttonItem.limit}}</button>
					</div>
				</div>
			</div>
			
			
		</div>
		
		<goods-dialog ref="goodsDialog"></goods-dialog>

	</div>
</template>

<script>
	import $U from '@/common/util.js';
	import goodsDialog from '@/components/shop/goods-dialog.vue';
	export default {
		components: {
			goodsDialog
		},
		inject:['app','layout'],
		data() {
			return {
				activeId: 0,
				appIndexCategory:[],
				app_index_category_id:1,
				swiper:{
					limit:5,
					data:[]
				},
				indexnavs:{
					limit:10,
					data:[]
				},
				threeAdv:{
					limit:3,
					data:[]
				},
				oneAdv:{
					title: "",
					cover: ""
				},
				list:{
					limit:8,
					data:[]
				},
				moduleList:[{
					name:"轮播图",
					type:"swiper",
					data:[]
				},{
					name:"图标分类",
					type:"indexnavs",
					data:[]
				},{
					name:"三图广告位",
					type:"threeAdv",
					data:[]
				},{
					name:"单图广告位",
					type:"oneAdv",
					data:{
						title: "",
						cover: ""
					}
				},{
					name:"商品列表",
					type:"list",
					data:[]
				}],
				indexList: [],
				cardLoading:false
			}
		},
		computed: {
			buttonItem(){
				return this[this.editItem.type]
			},
			editIndex(){
				return this.indexList.findIndex((item)=>item.id === this.activeId)
			},
			editItem(){
				return this.editIndex === -1 ? false : this.indexList[this.editIndex]
			}
		},
		created() {
			this.__init()
		},
		methods: {
			__init(){
				this.layout.showLoading()
				this.axios.get('/admin/app_index_category/list',{
					token:true
				}).then(res=>{
					this.appIndexCategory = res.data.data
					this.getData()
				}).finally(()=>{
					this.layout.hideLoading()
				})
			},
			getData(){
				this.cardLoading = true
				this.axios.get('/admin/app_index_data/list?app_index_category_id='+this.app_index_category_id,{
					token:true
				}).then(res=>{
					this.indexList = res.data.data
				}).finally(()=>{
					this.cardLoading = false
				})
			},
			addData(item){
				this.cardLoading = true
				this.axios.post('/admin/app_index_data',{
					type:item.type,
					order:50,
					data:JSON.stringify(item.data),
					app_index_category_id:this.app_index_category_id
				},{
					token:true
				}).then(res=>{
					let d = res.data.data
					d.data = JSON.parse(d.data)
					this.indexList.push(d)
					this.$message.success('添加成功')
				}).catch(err=>{
					this.$message.error('添加失败')
				}).finally(()=>{
					this.cardLoading = false
				})
			},
			choose(id) {
				this.activeId = id
				let item = { ...this.editItem }
				if(item && this[item.type]){
					if(Array.isArray(typeof item.data)){
						this[item.type].data = [...item.data]
					} else if(typeof item.data === 'object') {
						if(item.type === 'oneAdv'){
							this[item.type].title = item.data.title
							this[item.type].cover = item.data.cover
							return
						} 
						this[item.type].data = []
						for (let k in item.data) {
							let obj = {}
							switch (item.type){
								case 'indexnavs':
								obj = {
									src:item.data[k].src,
									text:item.data[k].text,
									id:item.data[k].id,
								}
									break;
								case 'list':
								obj = {
									id: item.data[k].id,
									cover: item.data[k].cover,
									title:item.data[k].title,
									desc: item.data[k].desc,
									oprice: item.data[k].oprice,
									pprice:item.data[k].pprice,
								}
									break;	
								default:
								obj = {
									src:item.data[k].src,
									id:item.data[k].id,
								}
									break;
							}
							this[item.type].data.push(obj)
						}
					}
				}
				console.log(this.list);
			},
			submit(){
				let item = this.buttonItem
				if(item){
					let o = this.indexList[this.editIndex]
					let newData = {}
					if(this.editItem.type === 'oneAdv'){
						newData.title = item.title
						newData.cover = item.cover
					} else {
						item.data.forEach((s,si)=>{
							newData[si.toString()] = { ...s }
						})
					}
					this.axios.post('/admin/app_index_data/'+o.id,{
						data:JSON.stringify(newData)
					},{
						token:true
					}).then(res=>{
						o.data = newData
						this.$message.success('保存成功');
					}).catch(err=>{
						this.$message.error('保存失败');
					})
				}
			},
			chooseImage(index){
				this.app.chooseImage((res)=>{
					if(this.editItem.type === 'oneAdv'){
						return this.oneAdv.cover = res[0].url
					}
					this[this.editItem.type].data[index].src = res[0].url
				},1)
			},
			deleteFormItem(index){
				this.$confirm('是否要删除?', '提示', {
					confirmButtonText: '删除',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					this[this.editItem.type].data.splice(index,1)
				})
			},
			addFormItem(){
				let obj = {}
				if(this.editItem.type === 'list'){
					// 弹出选择商品
					this.$refs.goodsDialog.chooseGoods((res)=>{
						res.forEach(item=>{
							this[this.editItem.type].data.push({
								cover:item.cover,
								desc: item.desc,
								id: item.id,
								oprice: item.min_oprice,
								pprice:item.min_price,
								title: item.title,
							})
						})
					},this.buttonItem.limit-this.buttonItem.data.length)
				} else if(this.editItem.type === 'indexnavs'){
					this[this.editItem.type].data.push({
						src:"",
						text:"",
						id:"",
					})
				} else {
					this[this.editItem.type].data.push({
						src:"",
						id:""
					})
				}
			},
			deleteItem(li){
				this.$confirm('是否要删除?', '提示', {
					confirmButtonText: '删除',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					this.indexList.splice(li,1)
				})
			},
			moveUp(index){
				if(index === 0){
					return
				}
				$U.moveUp(this.indexList,index)
				this.sort()
			},
			moveDown(index){
				if(this.indexList.length === 0 || index === (this.indexList.length - 1)){
					return
				}
				$U.moveDown(this.indexList,index)
				this.sort()
			},
			sort(){
				let sortdata = this.indexList.map((o,i)=>{
					return {
						id:o.id,
						order:(i+1)
					}
				})
				this.cardLoading = true
				this.axios.post('/admin/app_index_data/sort',{
					sortdata:JSON.stringify(sortdata)
				},{
					token:true
				}).then(res=>{
					this.$message.success('排序成功')
				}).catch(err=>{
					this.$message.error('排序失败')
				}).finally(()=>{
					this.cardLoading = false
				})
			}
		},
	}
</script>

<style>
	.el-carousel__item h3 {
		color: #475669;
		font-size: 14px;
		opacity: 0.75;
		line-height: 150px;
		margin: 0;
	}

	.el-carousel__item:nth-child(2n) {
		background-color: #99a9bf;
	}

	.el-carousel__item:nth-child(2n+1) {
		background-color: #d3dce6;
	}

	.app-index-item {
		cursor: pointer;
		border-width: 2px !important;
	}

	.app-index-item-active {
		border-color: #00a5a5 !important;
	}
	.app-index-icon-box{
		position: absolute;width: 60px;height: 60px;
		z-index: 100;
		right: -70px;
		top: 0;display: flex;flex-wrap: wrap;background-color: #FFFFFF;
	}
	.app-index-icon-btn{
		width: 30px;height: 30px;display: flex;align-items: center;justify-content: center;
		cursor: pointer!important;
	}
	.app-index-icon-btn:hover{
		background-color: #eeeeee!important;
	}
	.app-index-icon-btn-disabled{
		cursor: not-allowed!important;
		background-color: #EEEEEE!important;
	}
</style>
