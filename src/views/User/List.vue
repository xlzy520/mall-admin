<template>
	<div>
		<el-card class="box-card">
			<div slot="header" class="clearfix">
				<span>用户列表</span>
			</div>
			<el-table :data="tableData" style="width: 100%">
				<el-table-column prop="id" label="#">
				</el-table-column>
				<el-table-column sortable label="头像">
					<template slot-scope="scope">
						<el-avatar :src="scope.row.avatar" :size="50"></el-avatar>
					</template>
				</el-table-column>
				<el-table-column prop="username" sortable label="账号">
				</el-table-column>
				<el-table-column prop="fullname" sortable label="姓名">
				</el-table-column>
				<el-table-column prop="sex" sortable label="性别">
				</el-table-column>
				<el-table-column prop="tel" sortable label="手机">
				</el-table-column>
				<el-table-column prop="role_name" sortable label="角色">
					<template slot-scope="scope">
						<el-tag :type="scope.row.role===1?'danger':''">{{scope.row.role_name}}</el-tag>
					</template>
				</el-table-column>
				<el-table-column prop="login_time" width="160" sortable label="上次登录">
				</el-table-column>
				<el-table-column prop="login_count" sortable label="登录次数">
				</el-table-column>
				<el-table-column label="操作">
					<template slot-scope="scope">
						<el-button @click="showEditModal(scope.row)" plain icon="el-icon-edit" size="small" type="primary"></el-button>
						<el-button @click="showDeleteModal(scope.row.id)" :disabled="scope.row.id==1" icon="el-icon-delete" plain size="small"
						 type="danger"></el-button>
					</template>
				</el-table-column>
			</el-table>
		</el-card>
		<!--编辑数据-->
		<el-dialog title="修改信息" :visible.sync="editModalShow" @closed="handleCloseDialog('form')">
			<el-form ref="form" :model="form" :rules="rules" label-position="left" label-width="80px">
				<el-form-item label="姓名" prop="fullname">
					<el-input v-model="form.fullname" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别" prop="sex">
					<el-radio-group v-model="form.sex">
						<el-radio label="男">男</el-radio>
						<el-radio label="女">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="手机" prop="tel">
					<el-input v-model="form.tel" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="权限" prop="role">
					<el-select v-model="form.role" placeholder="请选择账户">
						<el-option v-for="item in roles" :key="item.id" :label="item.name" :value="item.id"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="头像" prop="avatar">
					<single-upload default-image="/images/avatar/default.jpg" :data="{type:'avatar'}" action="/api/upload/common/" :url.sync="form.avatar" />
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="editModalShow = false">取 消</el-button>
				<el-button type="primary" @click="updateInfo">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>

<script>
	//引入service模块
	import { Admin, Role } from '@/api/index'
	import SingleUpload from '@/components/SingleUpload.vue';

	export default {
		name: "List",
		components: {
			SingleUpload,
		},
		data() {

			return {
				tableData: [],
				roles: [
          { id: 1, name: '超级管理员' },
          { id: 2, name: '管理员' },
          { id: 3, name: '运营人员' },
        ],
				editModalShow: false,
				form: {
					id: "",
					fullname: "",
					sex: "男",
					tel: "",
					role: "",
					avatar: "",
				},
				rules: {
					fullname: [
						{ required: true, type: 'string', message: '请输入真实姓名！', trigger: 'blur' },
						{ pattern: /^[\u4E00-\u9FA5A-Za-z\s]+(·[\u4E00-\u9FA5A-Za-z]+)*$/, message: '请输入有效的姓名', trigger: 'blur' }
					],
					sex: [
						{ required: true, message: '请选择性别！', trigger: 'blur' },
					],
					tel: [
						{ required: true, message: '请输入手机号码！', trigger: 'blur' },
						{ pattern: /^1(3|4|5|6|7|8|9)\d{9}$/, message: '请输入有效的手机号码', trigger: 'blur' }
					],
					role: [
						{ required: true, message: '请选择账户角色！', trigger: 'blur' },
					],
					avatar: [
						{ required: true, message: '请上传一张头像！', trigger: 'click' },
					],
				}
			};
		},
		created() {
			this.loadList();
			document.title = "用户列表";
		},
		methods: {
			async loadList() {
				let { status, data } = await Admin.list();
				if (status) {
					this.tableData = data;
				}
			},
			// 显示编辑框
			showEditModal(row) {
				this.editModalShow = true;
				this.form = { ...row };
			},
			// 删除账户
			showDeleteModal(id) {
				this.$confirm('此操作将永久删除该账户, 是否继续?', { type: 'warning' })
					.then(async () => {
						let { status, msg } = await Admin.remove({ id });
						if (status) {
							this.$message.success('删除成功！');
							this.loadList();
						}
					})
					.catch(() => {
						this.$message.info('已取消删除');
					});
			},
			// 修改账户信息
			updateInfo() {
				this.$refs.form.validate(async (valid) => {
					if (!valid) { return false };
					let { status, msg } = await Admin.update({ ...this.form });
					if (status) {
						this.editModalShow = false;
						this.$message.success(msg);
						this.loadList();
					}
				})
			},
			// 关闭dialog弹窗，清除之前的验证提示
			handleCloseDialog(formName) {
				this.$refs[formName].clearValidate();
			},
		}
	};
</script>

<style scoped>

</style>
