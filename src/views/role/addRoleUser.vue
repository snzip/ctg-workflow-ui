<template>
    <div class="addRoleUser">
        <Modal v-model="showFlag" title="添加成员" @on-cancel="cancel" @on-ok="ok" :mask-closable="false"
         width="410px" :loading="loading">
            <div class="modal-body">
                <Row>
                    <Col span="6">角色名称</Col>
                    <Col span="18">{{selectedRole}}</Col>
                </Row>
                <Row style="margin-top: 20px;">
                    <Col span="6">选择成员</Col>
                    <Col span="18">
                        <div class='addroleusers-userselect' id='AddRoleUserControlId' 
                         data-ismultiple='true' data-uservisible='true' data-orgunitvisible='true' data-width='100%'></div>
                    </Col>
                </Row>
            </div>
            <div slot="footer" class="bottom_btns">
                <a class="confirm" @click="ok">确定</a>
                <a class="cancel" @click="cancel">取消</a>
            </div>
        </Modal>
    </div>
</template>

<script>
import { GetRoleNameAndUsers, UpdateRoleUsers } from '../../api/role.js';   //引入添加角色成员接口
import './BaseControl.js';
import './ControlManager.js';
import './SmartForm.js';
import './FormControls.js';
import './FormMultiUser.js';

export default {
    props: [
        "showAddUserRoleFlag",
        "RoleId",
        "selectedRole",
        "UnitTypeUser",
        "roleusersList"
    ],
    data() {
        return {
            loading: true,
            showFlag: false,
            AddRoleUserControl: '',  
        }
    },
    watch: {
        showAddUserRoleFlag: function() {
            this.showFlag = this.showAddUserRoleFlag
            if(this.showAddUserRoleFlag) {
                this._GetRoleNameAndUsers()
            }
        }
    },
    mounted() {
        // 创建页面时，获取部门信息数据
        let _self = this
        this.$nextTick(() => {
            _self._initFormUser()
        })  
    },
    methods: {
        // 远程请求时，确定按钮出现loading圆圈
         _setLoading() {
            var _self = this
            setTimeout(() => {
                _self.loading = false;
                _self.$nextTick(() => {
                    _self.loading = true;
                })
            }, 1000)
        },
        // 初始化FormUser控件
        _initFormUser() {
            if (!this.AddRoleUserControl) {
                this.AddRoleUserControl = $("#AddRoleUserControlId").RoleFormMultiUser();
            }
        },
        // 获取角色里面的成员信息
        async _GetRoleNameAndUsers() {
            // let res = await GetRoleNameAndUsers("GetRoleNameAndUsers", this.RoleId);
            // if (res.Successful) {
            //    if (res.ReturnData["NameTable"]) {
            //         this.SetUserControlValue(this.AddRoleUserControl, res.ReturnData["NameTable"], this.UnitTypeUser);
            //     }
            // } else {
            //     // 获取数据失败
            //     $.IShowError(res.ErrorMessage);
            // }
            console.log(this.roleusersList);
            console.log(this.getNameTable(this.roleusersList))
            this.SetUserControlValue(this.AddRoleUserControl,this.getNameTable(this.roleusersList),this.UnitTypeUser);
        },
        // 更新角色成员
        async _UpdateRoleUsers(UserIds) {
            UserIds = JSON.parse(UserIds);
            let res = await UpdateRoleUsers(UserIds, this.RoleId);
            if (res.code === '0') {
               this.$emit('getUserRoles', true);
               // 将点击确定之后的showModal 派发给父组件
                this.showFlag = false;
                this.$emit('addRoleUserShowFlag');
            } else {
                // 获取数据失败
                $.IShowError(res.ErrorMessage);
                this.showFlag = true;
            }
        },
        SetUserControlValue(controlItem, value, type) {
            var ArrayUnit = [];
            for (var key in value) {
                var displayname = value[key];
                let Unit = { "id": key, "Code": null, "userName": displayname, "ParentId": null, "Type": type, "Icon": null, "DepartmentName": null, "Birthday": null, "_Gender": null, "Gender": null, "email": null, "Mobile": null };
                ArrayUnit.push(Unit);
            }
            controlItem.SetValue(ArrayUnit);
        },
        ok() {
            let UserIds = JSON.stringify(this.AddRoleUserControl.GetUnitIDs());
            this._UpdateRoleUsers(UserIds);     
        },
        cancel() {
            this.AddRoleUserControl.ClearChoices();
            // 将点击取消之后的showModal 派发给父组件
            this.$emit('addRoleUserShowFlag')
        },
        getNameTable (roleusersList){
            let nameTable = {};
            for (let i = 0; i < roleusersList.length; i++){
                nameTable[roleusersList[i].id] = roleusersList[i].userName;
            }
            return nameTable;
        }
    }
}
</script>





// WEBPACK FOOTER //
// src/components/role/addRoleUser.vue