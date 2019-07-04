<template>
	<div class="">
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item>
					<i class="el-icon-lx-copy"></i>
					tab选项卡
				</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="container">
			<el-tabs v-model="message">
				<el-tab-pane :label="`未读消息(${unread.length})`" name="first">
					<el-table :data="unread" :show-header="false" style="width: 100%">
						<el-table-column>
							<template slot-scope="scope">
								<span class="message-title">{{ scope.row.title }}</span>
							</template>
						</el-table-column>
						<el-table-column prop="date" width="180"></el-table-column>
						<el-table-column width="120">
							<template slot-scope="scope">
								<el-button size="small" @click="handleRead(scope.$index,1)">标为已读</el-button>
							</template>
						</el-table-column>
					</el-table>
					<div class="handle-row">
						<el-button type="primary" @click="handleAllRead(scope)">全部标为已读</el-button>
					</div>
				</el-tab-pane>
				<el-tab-pane :label="`已读消息(${read.length})`" name="second">
					<template v-if="message === 'second'">
						<el-table :data="read" :show-header="false" style="width: 100%">
							<el-table-column>
								<template slot-scope="scope">
									<span class="message-title">{{ scope.row.title }}</span>
								</template>
							</el-table-column>
							<el-table-column prop="date" width="150"></el-table-column>
							<el-table-column width="120">
								<template slot-scope="scope">
									<el-button type="danger" @click="handleDel(scope.$index,2)">放入回收站</el-button>
								</template>
							</el-table-column>
						</el-table>
						<div class="handle-row">
							<el-button type="danger">删除全部</el-button>
						</div>
					</template>
				</el-tab-pane>
				<el-tab-pane :label="`回收站(${recycle.length})`" name="third">
					<template v-if="message === 'third'">
						<el-table :data="recycle" :show-header="false" style="width: 100%">
							<el-table-column>
								<template slot-scope="scope">
									<span class="message-title">{{ scope.row.title }}</span>
								</template>
							</el-table-column>
							<el-table-column prop="date" width="150"></el-table-column>
							<el-table-column width="200">
								<template slot-scope="scope">
									<el-button @click="handleRestore(scope.$index,3)">还原</el-button>
									<el-button type="danger" @click="handleRestore(scope.$index,4)">删除</el-button>
								</template>
							</el-table-column>
						</el-table>
						<div class="handle-row">
							<el-button type="danger">清空回收站</el-button>
						</div>
					</template>
				</el-tab-pane>
			</el-tabs>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'tabs',
		data() {
			return {
				message: 'first',
				showHeader: false,
				unread: [],
				read: [],
				recycle: []
			};
		},
		created() {
			this.getData();
		},
		methods: {
			getData() {
				const data = this.$qs.stringify({});
				const url = 'http://localhost:8086/tabs/queryAll';
				this.$axios
					.post(url, data)
					.then(res => {
						this.message = res.data.message;
						this.showHeader = res.data.showHeader;
						this.unread = res.data.unread;
						this.read = res.data.read;
						this.recycle = res.data.recycle;
					})
					.catch(res => {
						this.$message.error('系统异常，请联系管理员');
					});
			},
			handleRead(index, category) {
				//splice  向数组中添加数据  将第index数组元素拿出来 
				const item = this.unread.splice(index, 1);
				console.log(item);
				this.read = item.concat(this.read);
				this.handUpdate(item, 1);
			},
			handleAllRead(index) {
				//const item = this.unread.splice(index, 1);
				console.log(index);
				//this.read = item.concat(this.read);
			},
			handleDel(index, category) {
				const item = this.read.splice(index, 1);
				this.recycle = item.concat(this.recycle);
				this.handUpdate(item, 2);
				
			},
			handleRestore(index, category) {
				const item = this.recycle.splice(index, 1);
				if (category == 4) {
					this.handUpdate(item, 4);
				} else {
					this.handUpdate(item, 3);
					this.read = item.concat(this.read);
				}
			},
			handUpdate(item, category) {
				let xStatus = -1;
				switch (category) {
					case 1:
						xStatus = 2;
						break;
					case 2:
						xStatus = 3;
						break;
					case 3:
						xStatus = 2;
						break;
					default:
						break;
				}
				const url = "http://localhost:8086/tabs/updateMsg";
				const data = this.$qs.stringify({
					id: item[0].id,
					xStatus: xStatus
				});
				this.$axios.post(url, data)
					.then(res => {
						this.$message.warning('成功')
					})
					.catch(res => {
						this.$message.error('更新消息异常')
					});

			}
		},
		computed: {
			unreadNum() {
				return this.unread.length;
			}
		}
	};
</script>

<style>
	.message-title {
		cursor: pointer;
	}

	.handle-row {
		margin-top: 30px;
	}
</style>
