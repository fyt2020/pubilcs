<template>
  <div class="user-wrapper">
    <m-madd :con="addData" ref="addUser" />
    <div class="user-content">
      <m-crums :con="crumsData" />
      <div class="user__box">
        <div class="user__tree">
          <div class="userTree__box">
            <span class="userTree__til">组织机构</span>
          </div>
          <el-tree
            :data="data1"
            :props="defaultProps"
            node-key="expanded"
            :default-expanded-keys="['1.']"
            :highlight-current="true"
            :expand-on-click-node="false"
            ref="tree1"
            @node-click="choseOne"
          ></el-tree>
        </div>
        <div class="user__Rcon">
          <div class="user__title">用户列表</div>
          <div class="shoot-inp">
            <div class="inp-con" v-if="power[5035]">
              <label for="five">
                <label for="five">
                  <span class="must">用户名</span>
                  <input type="text" class="inp" v-model="username" placeholder="用户名" />
                </label>
              </label>
            </div>
            <div class="inp-con" v-if="power[5035]">
              <label for="five">
                <span class="must">角色</span>
                <el-select v-model="roleVal" placeholder="请选择">
                  <el-option
                    @click.native="selectRole(item)"
                    v-for="item in roleArr"
                    :key="item.rolename"
                    :label="item.rolename"
                    :value="item.rolename"
                  ></el-option>
                </el-select>
              </label>
            </div>
            <div class="inp-con found" v-if="power[5035]">
              <button class="addDec1" @click="foundUser">查询</button>
            </div>
            <div class="inp-con add">
              <button class="addDec" @click="addUser" v-if="power[5023]">新增用户</button>
            </div>
          </div>

          <div class="dec-list">
            <el-table :data="tableData" stripe style="width: 100%" highlight-current-row height="540">
              <el-table-column prop="index" label="序号" width="80" height="40" align="center"></el-table-column>
              <el-table-column
                prop="name"
                label="用户名"
                width="220"
                height="40"
                align="center"
                :show-overflow-tooltip="true"
              ></el-table-column>
              <el-table-column
                prop="orgName"
                label="所属机构"
                width="220"
                height="40"
                align="center"
                :show-overflow-tooltip="true"
              ></el-table-column>
              <el-table-column
                prop="accountName"
                label="登录账号"
                width="220"
                height="40"
                align="center"
                :show-overflow-tooltip="true"
              ></el-table-column>
              <el-table-column
                prop="telephone"
                label="手机号码"
                width="220"
                height="40"
                align="center"
                :show-overflow-tooltip="true"
              ></el-table-column>

              <el-table-column
                prop="rolename"
                label="角色"
                height="40"
                width="190"
                align="center"
                :show-overflow-tooltip="true"
              ></el-table-column>
              <el-table-column fixed="right" label="操作" width="160" height="40" align="center">
                <template slot-scope="scope">
                  <el-button @click="editRole(scope.row)" type="text" size="small" v-if="power[5025]">编辑</el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>
             <div class="slot-box">
              <div class="slot-con">
                <el-pagination
                  class="slot"
                  background
                  @size-change="handleSizeChange"
                  @current-change="handleCurrentChange"
                  :current-page.sync="currentPage"
                  :page-sizes="[10,20,50]"
                  :page-size="pagesize"
                  layout="total, sizes, prev, pager, next, jumper"
                  :total="total"
                ></el-pagination>
              </div>
            </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MCrums from "@/components/crums/crums.vue";
import MMadd from "@/components/ms/addUser.vue";
import { mapState, mapMutations } from "vuex";
export default {
  props: {},
  data() {
    return {
      data1: [
        {
          id: 1,
          label: "一级 1",
          children: [
            {
              id: 4,
              label: "二级 1-1",
              children: [
                {
                  id: 9,
                  label: "三级 1-1-1"
                },
                {
                  id: 10,
                  label: "三级 1-1-2"
                }
              ]
            }
          ]
        },
        {
          id: 2,
          label: "一级 2",
          children: [
            {
              id: 5,
              label: "二级 2-1"
            },
            {
              id: 6,
              label: "二级 2-2"
            }
          ]
        },
        {
          id: 3,
          label: "一级 3",
          children: [
            {
              id: 7,
              label: "二级 3-1"
            },
            {
              id: 8,
              label: "二级 3-2"
            }
          ]
        }
      ],
      defaultProps: {
        children: "children",
        label: "orgName"
      },
      tableData: [],
      addData: {
        key: false,
        type: "",
        orgId:"",
        OrgId:"",
      },

      username: "",
      dialogKey1: false,
      centerDialogVisible: false,
      currentPage: 1,
      pagesize: 10,
      total: 0,

      roleVal: "全部",

      roleId: "",
      roleArr: [],

      orgId: "", //机构id
      orgName: "", //机构名称
      parentId:"",

      idNo: "",
      collecOrg: "",
      collecOrgArr: [],

      crumsData: {
        one: "用户管理",
        two: "用户列表"
      },
      statusVal: "全部",
      statusArr: [
        {
          name: "全部",
          value: ""
        },
        {
          name: "启用",
          value: 1
        },
        {
          name: "禁用",
          value: 0
        }
      ],
      OrgId:"",
    };
  },
  computed: {
     ...mapState(["power"])
  },
  created() {
    this.getOrg();
    this.getPower();
    this.OrgId = JSON.parse(window.sessionStorage.getItem("token")).orgId;
  },

  methods: {

        ...mapMutations(["set_power"]),

     /**
     * @Author fyt
     * @Description 获取权限
     * @Date 2020-03-17 10:18:25 星期二
     */
      getPower() {
      let VM = this;
      let resourceList = new Object();
      let roleId = JSON.parse(window.sessionStorage.getItem("token")).roleId;

      if (roleId) {
        this.$http.api
          .get(
            window.BASEURL.roleUrl +
              `/yyqxzx/api/v1/permissions?id=${roleId}&type=2`
          )
          .then(function(res) {
            if (res.data.code === 0) {
              let data = res.data.result.data;
              if (data.resourceList && data.resourceList.length > 0) {
                data.resourceList.forEach(item => {
                  let key = item.checked * 1 == 1;
                  resourceList[item.id] = key;
                });
              }
              VM.set_power({ val: resourceList });
            }
          })
          .catch(function(error) {
            console.log(error);
          });
      } else {
        throw new Error("roleId");
      }
    },

    /**
     * @Author fyt
     * @Description 新增用户
     * @Date 2020-02-27 10:29:17 星期四
     */
    addUser() {
      this.addData.key = true;
      this.addData.type = "add";
      this.addData.orgId = this.orgId;
      this.addData.OrgId = this.OrgId;
      if(this.addData.orgId == this.OrgId){
         this.$refs.addUser.getOrg()
         this.$refs.addUser.getRoleArr(this.orgId)
      }else{
         this.$refs.addUser.getRoleArr(this.orgId)
      }
      this.$refs.addUser.deviceData.orgId = this.orgId;
      this.$refs.addUser.deviceData.orgName= this.orgName;
    },

    /**
     * @Author fyt
     * @Description 选择组织机构
     * @Date 2020-02-27 10:28:33 星期四
     */
    choseOne(data) {
    
      this.username = "";
      this.roleVal = "全部";
      this.statusVal = "全部";

      this.orgId = data.orgId;
      this.orgName = data.orgName;
      this.parentId = data.parentId;

      this.$refs.addUser.orgData = data;
      // this.$refs.addUser.deviceData.orgId = data.orgId;
      // this.$refs.addUser.deviceData.orgName = data.orgName;
      if(this.OrgId == data.orgId){
         this.getUserList("");
      }else{
         this.getUserList(this.orgId);
      }
      this.getRoleArr(this.orgId)
    },

    /**
     * @Author fyt
     * @Description 查询
     * @Date 2020-02-27 10:30:12 星期四
     */
    foundUser() {
      let VM = this,id;
      this.currentPage = 1;
      let status;
      if (this.statusVal == "启用") {
        status = 1;
      } else if (this.statusVal == "禁用") {
        status = 0;
      }
      if(this.parentId == ""){
         id = ""
      }else{
         id = this.orgId
      }
      this.$http.api
        .get(
          window.BASEURL.testUrl1 +
            `parent-police/farsee2/api/v1/accounts?&pageNum=${
              this.currentPage
            }&pageSize=${this.pagesize}&names=${VM.username}&orgId=${
              id
            }&status=${VM.statusVal == "全部" ? "" : status}&roleId=${
              VM.roleId
            }`
        )
        .then(function(res) {
          if (res.data.code === 0) {
            let data = res.data.result.datas;
            let len = data.length;
            if (len <= 0) {
              VM.$Message.error("没有该用户!");
              VM.tableData = [];
              VM.total = 0;
              return;
            } else {
              data.forEach((item, index) => {
                item["index"] = index + 1;
              });
              VM.tableData = data;
              VM.total = res.data.result.total;
              VM.$Message.success("查询成功!");
            }
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    /**
     * @Author fyt
     * @Description 选择角色查询
     * @Date 2020-02-27 10:30:27 星期四
     */
    selectRole(data) {
      // console.log(data)
      this.roleId = data.id;
    },
    /**
     * @Author fyt
     * @Description 选择状态查询
     * @Date 2020-02-27 10:30:40 星期四
     */
    selectStatus(data) {
      // console.log(data)
    },
    /**
     * @Author fyt
     * @Description  初始化获取数据
     * @Date 2020-02-27 10:31:11 星期四
     */
    getUserList(id) {
      //获取用户lie'biao
      if (id == undefined || id == null) id = "";
      let VM = this;
      this.$http.api
        .get(
          window.BASEURL.testUrl1 +
            `parent-police/farsee2/api/v1/accounts?pageNum=${this.currentPage}&names=${VM.username}&orgId=${id}&pageSize=${this.pagesize}`
        )
        .then(function(res) {
          if (res.data.code === 0) {
            let data = res.data.result.datas;
            VM.total = res.data.result.total;
            data.forEach((item, index) => {
              item["index"] = index + 1;
            });
            VM.tableData = data;
            // VM.total = res.data.result.total;
            // console.log(res);
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    /**
     * @Author fyt
     * @Description 获取机构
     * @Date 2020-02-27 11:35:57 星期四
     */
    getOrg() {
      let VM = this;
      let newArr = [];
      this.$http.api
        .get(
          window.BASEURL.testUrl1 +
            `/parent-police/farsee2/api/v1/organizations?flag=1`
        )
        .then(function(res) {
          if (res.data.code === 0) {
            let data = res.data.result;
            VM.orgId = data.orgId;
            VM.orgName = data.orgName;
            VM.parentId = data.parentId;
            VM.$refs.addUser.orgData = data;
            newArr.push(data);
            VM.data1 = VM.dellData(newArr);
            console.log(newArr)
            VM.getRoleArr(VM.orgId);
            VM.getUserList();

          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    /**
     * @Author fyt
     * @Description 处理树状数据 data1
     * @Date 2020-05-04 15:10:34 星期一
     */
    dellData(data){
        if(data&&data.length > 0){
             data.forEach((item)=>{ 
                    item.expanded = '1.';
                    if(item.children && item.children.length >0){
                          item.children.forEach((i)=>{
                               i.expanded = '1.';
                          })
                    }
             })
        }
        return data
    },
    /**
     * @Author fyt
     * @Description 获取角色
     * @Date 2020-02-27 11:36:19 星期四
     */
    getRoleArr(data) {
      let VM = this;
      this.$http.api
        .get(window.BASEURL.roleUrl + `/yyqxzx/api/v1/roles?type=1&filter=identify=${data}&extend=true`)
        .then(function(res) {
          if (res.data.code === 0) {
            let data = res.data.result.datas;
            let obj = {
              cKey: "",
              cValue: "",
              description: "",
              rolename: "全部",
              id: "",
              value: ""
            };
            VM.roleArr = data;
            VM.roleArr.unshift(obj);
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    /**
     * @Author fyt
     * @Description 改变每页显示条数
     * @Date 2020-02-27 14:39:14 星期四
     */

    handleSizeChange(size) {
      this.pagesize = size;
        if(this.parentId == ""){
         this.getUserList("");
      }else{
         this.getUserList(this.orgId);
         
      }
    },
    /**
     * @Author fyt
     * @Description 跳到某一页
     * @Date 2020-02-27 14:39:27 星期四
     */
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage;
      if(this.parentId == ""){
         this.getUserList("");
      }else{
         this.getUserList(this.orgId);  
      }
    },
    /**
     * @Author fyt
     * @Description 编辑角色
     * @Date 2020-02-27 14:21:35 星期四
     */
    editRole(data) {
      this.addData.key = true;
      this.addData.type = "edit";
      this.$refs.addUser.getUserInfo(data.accountId);
      this.$refs.addUser.getRoleArr(data.orgId);
    },
    /**
     * @Author fyt
     * @Description 确认重置密码
     * @Date 2020-02-27 14:21:04 星期四
     */
    handleClick(data) {
      let sure = window.confirm("确认重置密码？");
      if (!sure) {
        return;
      }
      let accountId = data.accountId;
      let VM = this;
      this.$http.api
        .put(
          window.BASEURL.testUrl1 +
            `parent-police/farsee2/api/v1/accounts/password/${accountId}`
        )
        .then(function(res) {
          // console.log(res)
          if (res.data.code === 0 && res.status == 200) {
            VM.$refs.addUser.deviceData.accountPasswd = "123456";
            VM.$refs.addUser.resetPassw = "重置成功";
            setTimeout(() => {
              VM.$refs.addUser.resetPassw = "重置密码";
            }, 4000);
            // VM.centerDialogVisible = true;
          } else {
            VM.$Message.success(`${res.data.msg}`);
          }
        })
        .catch(function(error) {
          console.log(error);
        });
      console.log(accountId);
      //  this.dialogKey1 = true;
    },

    sure() {
      this.centerDialogVisible = false;
    }
  },
  components: {
    MCrums,
    MMadd
    // MMrest
  }
};
</script>

<style scoped lang="scss">
@import "@/assets/css/userMange/userList.scss";
</style>
