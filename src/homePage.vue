<template>
    <!-- <button @click="refresh()">refresh</button> -->
    <!-- 添加联系人 -->
    
    <div id="container" class="center-justify">
        <el-button plain @click="dialogFormVisible = true" type="primary" class="add-contact-btn">添加联系人</el-button>
        <div class="sea-container">
            <form action="">
                <input v-model="searchMsg" type="text" name="search" class="blue-input" @input="handleSearch(searchMsg)" autocomplete="off">
                <input type="submit" value="搜索" class="blue-button" autocomplete="off" @click.prevent="handleSearch(searchMsg),searchedTable()">
            </form>
            <ul v-if="searchMsg">
                <li v-for="item in filteredData" :key="item.id" @click="searchedTable(),searchMsg = ''">{{ item.contactName }}</li>
            </ul>
        </div>
    </div>
    <el-dialog v-model="dialogFormVisible" title="Add Contact" width="500">
        <el-form :model="form" :rules="rules" ref="formRef">
          <el-form-item label="contact name" :label-width="formLabelWidth" prop="name">
            <el-input v-model="form.name" autocomplete="off" />
          </el-form-item>
          <el-form-item label="phone number" :label-width="formLabelWidth" prop="number">
                <el-input v-model="form.number" autocomplete="off" />
          </el-form-item>
        </el-form>
        <template #footer>
            <div class="dialog-footer">
              <el-button @click="dialogFormVisible = false, form.name = '', form.number = '', formRef.clearValidate()">Cancel</el-button>
              <!-- 添加操作 -->
              <el-button type="primary" @click="handAddContact()">Confirm</el-button>
            </div>
        </template>
    </el-dialog>
    <!-- 表单 -->
            <el-table :data="tableData" style="width: 100vw; height: 80vh;">
              <el-table-column prop="contactName" label="Name" align="center" width="180" />
              <el-table-column prop="phoneNumber" label="Phone Number" align="center"/>
              <el-table-column label="操作"  align="center">
                <template v-slot="scope">
                    <el-button type="parimary" @click="openEditForm(scope.row, scope.$index)">修改</el-button>
                    <el-button type="warning" @click="deleteContact(scope.row.id)">删除</el-button>
                </template>
            </el-table-column>
            </el-table>
            <el-dialog v-model="dialogEditFormVisible" title="Edit Contact" width="500" >
                <el-form :model="form1" :rules="rules" ref="formRef">
                  <el-form-item label="contact name" :label-width="formLabelWidth" prop="name">
                    <el-input v-model="form1.name" autocomplete="off" />
                  </el-form-item>
                  <el-form-item label="phone number" :label-width="formLabelWidth" prop="number">
                        <el-input v-model="form1.number" autocomplete="off" />
                  </el-form-item>
                </el-form>
                <template #footer>
                    <div class="dialogEdit-footer">
                      <el-button @click="dialogEditFormVisible = false, form1.name = '', form1.number = '', formRef.clearValidate()">Cancel</el-button>
                      <!-- 添加操作 -->
                      <el-button type="primary" @click="handleEdit()">Confirm</el-button>
                    </div>
                </template>
            </el-dialog>
    <!-- </div> -->
</template>

<script lang="ts" setup>
    // import {getData} from './connect.js'
import {pageJump} from './pageJumps.js'
import { reactive, ref,computed,watchEffect} from 'vue'
import { onMounted } from 'vue'
// import {addContact,
//     fetchAllContacts} from './function.js'
import { inject } from 'vue';
const tableData = ref([]); //列表联系人数据
const dialogFormVisible = ref(false) //弹窗是否显示
const dialogEditFormVisible = ref(false) //弹窗是否显示
const allData = [];
        // 获取数据
        const url = 'http://127.0.0.1:8081/Contacts/findAll'
          async function fetchHotSearch() {
            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Access-Control-Allow-Origin': '*',
                        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
                        'Access-Control-Allow-Headers': 'Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With'
                    }
                });
                const data = await response.json();
                // 输出返回的数据以供调试
                console.log('API Response:', data);
                // 确认data字段存在且为数组
                if (data.data && Array.isArray(data.data)) {
                    tableData.value= data.data;
                } else {
                    console.error('Expected an array, but got:', data);
                }
            } catch (error) {
                console.error('Error fetching data:', error);
            }
        }
        fetchHotSearch() //加载页面时获取数据
        watchEffect(() => {
            allData.value = tableData.value;
            console.log("allData:", allData.value); // 这里可以观察 allData 的变化
        });
// 在适当的时候调用 loadData，比如数据加载完成后

        // 删除联系人
        async function deleteContact(id) {
        try {
            const response = await fetch(`http://127.0.0.1:8081/Contacts/${id}`, {
                method: 'DELETE',
            });
            this.tableData.splice(this.tableData.findIndex(item => item.id === id), 1);
            if (!response.ok) {
                throw new Error('删除联系人失败');
            }
            console.log('联系人删除成功');
        } catch (error) {
            console.error('删除联系人失败:', error);
        }
        }
    
    //添加联系人
    async function addContact() {
        var raw = JSON.stringify({
            "contactName": form.name,
            "phoneNumber": form.number
        });
    try {
        const response = await fetch('http://127.0.0.1:8081/Contacts', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: raw,
            
        });
        if (!response.ok) {
            throw new Error('网络错误');
        }
        const newContact = await response.json();
        console.log('添加联系人成功:', newContact);
        tableData.value.push(newContact)
        return newContact;
    } catch (error) {
        console.error('添加联系人失败: ', error);
    }
    }
    const formRef = ref(null); //表单引用
    const submitForm = () => {
        formRef.value.validate((valid) => {
            if (valid) {
            // 提交逻辑
            console.log('提交成功', form);
            } else {
            console.log('验证失败');
            }
        });
        };
    function handAddContact() {
        if (!form.name || !form.number) {
            return;
        }
        dialogFormVisible.value = false;
        addContact()
        form.name = '';
        form.number = '';
        formRef.value.clearValidate();
    }
    // updateContact 
    async function updateContact(id, updatedContact) {
    try {
        const response = await fetch(`http://127.0.0.1:8081/Contacts/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(updatedContact),
        });
        if (!response.ok) {
            throw new Error('更新联系人失败');
        }
        const contact = await response.json();
        console.log('更新后的联系人:', contact);
        tableData.value[currentIndex.value] = contact;
        return contact;
    } catch (error) {
        console.error('更新联系人失败:', error);
    }
    }
    const currentIndex = ref(null); // 用于存储当前行的索引
    function openEditForm(row, index ) {
        currentIndex.value = index;
        //填充表单打开弹窗
        dialogEditFormVisible.value = true;
    }
    // 编辑联系人
    function handleEdit() {
        
        // 打开弹窗
        if (!form1.name || !form1.number) {
            return;
        }
        dialogEditFormVisible.value = false;
        // 填充表单
        updateContact( tableData.value[currentIndex.value].id , {
            contactName: form1.name,
            phoneNumber: form1.number,
        });
        form1.name = '';
        form1.number = '';
        formRef.value.clearValidate();
    }
    function refresh() {
        location.reload()
    }
        
        
        const formLabelWidth = '140px'
        const form = reactive({
            name: '',
            number: '',
            delivery: false,
            type: [],
            resource: '',
        })
        const form1 = reactive({
            name: '',
            number: '',
            delivery: false,
            type: [],
            resource: '',
        })
        const rules = {
            name: [
                { required: true, message: '名称不能为空', trigger: 'blur' },
            ],
            number: [
                { required: true, message: '电话号码不能为空', trigger: 'blur' },
            ],
        }
        //search 方法
        var searchMsg = ref('');
        var filteredData =ref(tableData.value);

        
    function    handleSearch(queryString) {
      filteredData.value = [];
        if(queryString === '') {
            return;
        }
      tableData.value.forEach(item => {
        if (item.contactName.indexOf(queryString) !== -1) {
            // if(queryString!== '') {
            this.filteredData.push(item);
            // }
          console.log(this.filteredData)
        }
      });
    }
    
    function searchedTable() {
        if(searchMsg.value === ''){
            location.reload()
            return;
        }
        tableData.value = filteredData.value;

    }
</script>

<style>
    ul{
        list-style: none;
        border: #317EF3;
        position: absolute;
        width: 350px;
        padding-left: 0;
        z-index: 10;
        padding-top: -10px;
    }
    li{
        text-align: left; 
        border: 1px rgb(194, 171, 168) solid;
    }
    .add-contact-btn {
        float: left;
        margin-left: 50px;
    }
.center-align {
    display: flex;
    align-items: center;
}
.center-justify {
    display: flex;
    justify-content: center;
}
.center-spacearound {
    display: flex;
    align-items: center;
    justify-content: space-around;
}
 
/*------------------------本项目公共样式----------------------------*/
#container {
    width: 1280px;
    margin: 0 auto;
    display: flex; /* 使用 flexbox */
    justify-content: space-between; /* 使元素在左右两端 */
    align-items: center; /* 垂直居中对齐 */
    box-sizing: border-box;
    padding-bottom: 15px;
    /*background-color: yellow;*/
}
 
/*搜索栏*/
.sea-container {
    width: 450px;
    height: 30px;
    margin: 0 auto;
    /* background-color: green; */
    /* border: 1px solid #317EF3; */
    padding-right: 400px;
}
.blue-input {
    float: left;
    width: 350px;
    height: 30px;
    box-sizing: border-box;
}
.blue-button {
    width: 100px;
    height: 30px;
    background-color: #317EF3;
    color: white;
    font-size: 15px;
    
}
 
/* 搜索下拉框*/
ul#drop {
    list-style: none;
    margin: 0;
    padding: 0;
}
ul#drop li {
    margin: 0;
    padding: 10px;
}
 
ul#drop li:hover {
    background-color: darkgrey;
	width:200px;
}
 
p#selectedId {
    display: inline-block;
}
</style>