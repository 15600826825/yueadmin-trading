<template>
	<section>
	  <!--  工具栏  -->
		<el-row class="toolbar">
			<el-button type="primary" @click="uploadVisible = true">本地上传</el-button>
			<el-button type="danger"  @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
		</el-row>
		<!-- 图片列表 -->
		<el-table 
			border
			:data="mediaList" 
			highlight-current-row
			style="width: 100%" 
			@selection-change="selsChange"> 
	    <el-table-column type="selection" width="55"></el-table-column>
	    <el-table-column type="index" width="60"></el-table-column>
	    <el-table-column label="图片缩略图" width="120">
	    	<template scope="scope">
		    	<img :src="scope.row.filePath" height="50px" class="cell-img" @click="viewImage(scope.row.filePath)">
	    	</template>
	    </el-table-column>
	    <el-table-column label="更新者" prop="createName" width="150"></el-table-column>
	    <el-table-column label="更新时间" prop="createTime" width="200" :formatter="formatCreateTime" sortable></el-table-column>
	    <el-table-column label="是否为主图" prop="isMainPic" width="120" :formatter="formatIsMainPic"></el-table-column>
	    <el-table-column label="是否显示" width="120">
	    	<template scope="scope">
	    		<el-switch
					  v-model="scope.row.status"
					  on-color="#13ce66"
					  off-color="#ff4949"
					  on-text="是"
					  off-text="否"
					  :on-value="1"
					  :off-value="0"
					  @change="handleStatus(scope.row)">
					</el-switch>
	    	</template>
	    </el-table-column>
		  <el-table-column label="操作">
	        <template scope="scope">
	        	<el-button v-if="scope.row.isMainPic === 0" size="small" type="primary" @click="handleSetMainImg(scope.$index, scope.row)">设为主图</el-button>
	        	<el-button size="small" type="danger" @click="handleSingleDelete(scope.$index, scope.row)">删除</el-button>
	        </template>
    	</el-table-column>
		</el-table>
		<el-row class="text-center m-t">
			<el-button size="large" type="primary">下一步</el-button>
		</el-row>
		<!-- 本地上传 -->
		<el-dialog title="本地上传" :visible.sync="uploadVisible" size="small">
			<el-row>
				<el-col :span="20" :class="{ 'text-danger' : mediaLength === 0 }">
					建议上传大小为1024*512图片，单张不超过2M，最多上传{{maxLength}}张，还可以上传{{mediaLength}}张。
				</el-col>
			</el-row>
			<el-row class="m-t">
				<el-col :span="20">	
					<el-upload
						name="fileName"
						:data="uploadData"
						ref="uploadMedia"
						class="multiple-uploader"
			  		action="/imgUploadUrl"
			  		list-type="picture-card"
			  		accept="image/jpeg, image/png"
			  		:on-preview="handleImgPreview"
			  		:on-remove="handleRemove"
			  		:before-upload="beforeUpload"
			  		:on-success="handleSuccess"
			  		:on-error="handleError"
			  		:multiple="true"
			  		:disabled="mediaLength === 0">
			  		<i class="el-icon-plus"></i>
					</el-upload>
				</el-col>
			</el-row>
		  	<div slot="footer">
			    <el-button @click="uploadVisible = false">取 消</el-button>
			    <el-button type="primary" @click="onSubmit">确 定</el-button>
	  		</div>  		
		</el-dialog>
		<!-- 图片预览 -->
		<el-dialog v-model="previewVisible" size="tiny">
			<img width="100%" :src="previewImgUrl">
		</el-dialog>
	</section>
</template>
<script>
	import { readWareFileList, createWareFile, updatetWareFileState, updateWareFileIsMainPic, deleteWareFile } from '@/api'
	export default{
		data () {
			return {
				maxLength: 10,
    		previewImgUrl: '',
				uploadVisible: false,
    		previewVisible: false,
    		uploadData: {
	        fileUrlType: 1
	      },
    		mediaForm: {
    			wareId: '',
    			fileList: []
    		},
				mediaList: [{
					fileId: 1,
					filePath: 'https://avatars0.githubusercontent.com/u/26806103?v=3&s=460',
					status: 1,
					isMainPic: 0,
					createName: '上传者'
				},{
					fileId: 2,
					filePath: 'https://avatars0.githubusercontent.com/u/26806103?v=3&s=460',
					status: 1,
					isMainPic: 0,
					createName: '上传者'
				}],
    		sels: [],
    		fileIdList: [],
    		value2: true
			}
		},
		methods: {
			formatIsMainPic (row, column) {
				return row.isMainPic === 1 ? '是' : '否'
			},
			formatCreateTime (row, column) {
				return this.$moment(row.createTime).format('YYYY-MM-DD HH:mm:ss')
			},
			// 获取图片列表
			getWareFileList () {
				readWareFileList({wareId: this.mediaForm.wareId}).then(res => {
					// console.log(res)
					if (res.data.code === '0001') {
						this.mediaList = res.data.result.fileList;
						this.mediaList.forEach(function(media, index) {
							media.filePath = 'http://192.168.199.211:8080/' + media.filePath;
						})
					} else {
						this.$message.error(res.data.message)
					}
				}).catch((error) => {
					this.catchError(error.response)
				})
			},
			// 上传校验
      beforeUpload (file) {
        const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error('图片只能是 JPG 或 PNG格式!');
        }
        if (!isLt2M) {
          this.$message.error('图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
	    },
    	// 上传成功处理
    	handleSuccess (res, file) {
    		if (res.code === '0001') {
    			let resFile = res.result.file;
			    file.path = resFile.filePath;
	    		this.mediaForm.fileList.push(resFile)
    		} else {
    			this.$message.error(res.message)
    		}
    	},
    	// 上传失败处理
    	handleError (err, file) {
    		this.$message.error('图片上传失败，请重试')
    	},
    	// 上传图片删除
			handleRemove (file) {
        if (!file) return;
        let fileList = this.mediaForm.fileList;
        let filePath = file.path || '';
        fileList = fileList.filter(file => file.filePath !== filePath)
        this.mediaForm.fileList = fileList
  		},
			// 图片预览
			handleImgPreview (file, fileList) {
				this.previewImgUrl = file.url;
        this.previewVisible = true;
    	},
 	    // 图片列表提交
	    onSubmit () {
	    	let data = JSON.stringify(Object.assign({}, this.mediaForm))
	    	console.log(data)
	    	createWareFile(data).then(res => {
	    		if(res.data.code === '0001') {
	    			this.$message.success('上传成功')
	    			this.getWareFileList()
	    		} else {
	    			this.$message.error(res.data.message)
	    		}
	    		this.uploadVisible = false
	    		this.mediaForm.fileList = []
	    		this.$refs.uploadMedia.clearFiles()
	    	})
	    	.catch(error => {
	    		this.catchError(error.response)
	    		this.uploadVisible = false
	    		this.mediaForm.fileList = []
	    		this.$refs.uploadMedia.clearFiles()
	    	})
	    },
	    // 图片选中
			selsChange (sels) {
				this.sels = sels;
			},
			// 图片查看
			viewImage (filePath) {
				this.previewImgUrl = filePath;
        this.previewVisible = true;
			},
			// 是否显示
			handleStatus (row) {
				let data = {
					fileId: row.fileId
				}
				updatetWareFileState(data).then(res => {
					if (res.data.code === '0001') {
						this.$message.success(res.data.message)
					} else {
						row.status = row.status === 1 ? 0 : 1
						this.$message.error(res.data.message)
					}
				}).catch(error => {
					row.status = row.status === 1 ? 0 : 1
					this.catchError(error.response)
				})
			},
			// 单张图片删除
			handleSingleDelete (index, row) {
				let _fileIdList = [];
				_fileIdList[0] = { fileId: row.fileId }
				this.$confirm('确认删除该图片？', '提示', {type: 'warning'})
				.then(() => {
					let data = {
						fileIdList: _fileIdList
					}
					console.log(data)
					deleteWareFile(JSON.stringify(data)).then(res => {
						console.log(res)
						if (res.data.code === '0001') {
							this.$message.success(res.data.message)
							this.getWareFileList()
						} else {
							this.$message.error(res.data.message)
						}
					}).catch(error => {
						this.catchError(error.response)
					})
				})
				.catch((err) => {
					console.log(err)
					this.$message('已取消操作')
				})
			},
			// 批量删除
			batchRemove () { 				
				this.$confirm('确认删除选中记录吗？', '提示', {type: 'warning'})
				.then(() => {
					let fileIds = this.sels.map(sel => sel.fileId);
					let _fileIdList = [];
					fileIds.forEach((value, key) => {
						_fileIdList.push({ fileId: value })
					})
					let data = {
						fileIdList: _fileIdList
					}
					deleteWareFile(JSON.stringify(data))
					.then(res => {
						if(res.data.code === '0001') {
							this.$message.success(res.data.message)
							this.getWareFileList()
						} else {
							this.$message.error(res.data.message)
						}
					})
					.catch((error) => {
						console.log(error)
						this.catchError(error.response)
					})
				})
				.catch((err) => {
					console.log(err)
					this.$message('已取消操作')
				})
			},
			// 设为主图
			handleSetMainImg (index, row) {	
				let data = {
					fileId: row.fileId,
					isMainPic: 1,
					wareId: 10001
				}	
				updateWareFileIsMainPic(data).then(res => {
					if(res.data.code === '0001') {
						this.$message.success(res.data.message)
						this.getWareFileList()
					} else {
						this.$message.error(res.data.message)
					}
				})
				.catch(error => {
					this.catchError(error.response)
				})
			},
		},
		computed:{
			mediaLength () {
				return  this.maxLength - this.mediaList.length - this.mediaForm.fileList.length;
			}
		},
		mounted () {
			this.$store.dispatch('setStepActive', 2)
			let wareId = this.$route.query.wareId;
			if(wareId) {
				this.mediaForm.wareId = parseInt(wareId);
				this.getWareFileList()
			}
		}
	}
</script>
