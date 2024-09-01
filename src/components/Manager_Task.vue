<script setup>
import { ref,onMounted } from 'vue'; 
import axios from 'axios';
const todoTasks = ref([]);
const doingTasks = ref([]);
const finishTasks = ref([]);
const isShowPopupCreate = ref(false);
const showPopupCreate=()=>{
    // Gán ngày giờ hiện tại cho biến date khi mở popup tạo mới
    dataNew.value.date = getCurrentDateTime();
    isShowPopupCreate.value = !isShowPopupCreate.value;
}
const closePopupCreate=()=>{
  isShowPopupCreate.value = false;
}
const isShowPopupMess= ref(false);
const currentId = ref(null)
const showPopupMess=(id)=>{
  currentId.value= id;
  isShowPopupMess.value = true;
}
const closePopupMess=()=>{
  isShowPopupMess.value = false;
}

const isShowPopupEdit = ref(false);
const showPopupEdit=(item)=>{
  createData.value.newID = item.id;
  createData.value.categories = item.categories;
  createData.value.title = item.title;
  createData.value.content = item.content;
  // createData.value.date = item.date;
  createData.value.date = getCurrentDateTime(); // Gán ngày giờ hiện tại khi chỉnh sửa
  createData.value.status = item.status;
  isShowPopupEdit.value = !isShowPopupEdit.value;
}
const closePopupEdit=()=>{
  isShowPopupEdit.value = false;
}
// const data = ref([]);
const getData= async()=>{
  try {
    const res = await axios.get('https://66403f25a7500fcf1a9d5ef0.mockapi.io/project_1/api/List_User');
    console.log("getData: ",res.data);
    const task = res.data;
    // data.value = res.data;
    // Reset các mảng trước khi phân loại lại
    todoTasks.value = [];
    doingTasks.value = [];
    finishTasks.value = [];
    task.forEach(task => {
      if(task.status === 'todo'){
        todoTasks.value.push(task);
      }
      else if(task.status === 'doing'){
        doingTasks.value.push(task);
      }
      else if(task.status === 'finish'){
        finishTasks.value.push(task);
      }
    });

  } catch (error) {
    console.error("Error fetching data:", error);
  }
}
onMounted(()=>{
  getData();
})
const dataNew = ref({
  categories:'',
  title:'',
  content:'',
  date:'',
  status:'todo',
})
const addItem= async()=>{
    const newItem ={
        categories: dataNew.value.categories,
        title: dataNew.value.title,
        content: dataNew.value.content,
        date: dataNew.value.date, // Đã có giá trị ngày giờ hiện tại
        status: dataNew.value.status,
    }
    try {
        const res = await axios.post(`https://66403f25a7500fcf1a9d5ef0.mockapi.io/project_1/api/List_User`,newItem);
        console.log('data duoc post thanh cong',res.data);
        getData();
        closePopupCreate();
    } catch (error) {
        console.log('post that bai!',error);
        
    }
}
const deleteItem=async()=>{
  try {
    const res = await axios.delete(`https://66403f25a7500fcf1a9d5ef0.mockapi.io/project_1/api/List_User/${currentId.value}`);
    console.log("id da xoa :",res.data);
    isShowPopupMess.value = false;
    getData();
  } catch (error) {
    
  }
}
const createData = ref({
  newID: '',
  categories: '',
  title:'',
  content:'',
  date:'',
  status:'todo',
}
)

const updateItem =async()=>{
  const updateItem = {
    categories: createData.value.categories,
    title:createData.value.title,
    content:createData.value.content,
    date:createData.value.date,
    status:createData.value.status,
  }
  try {
    const res = await axios.put(`https://66403f25a7500fcf1a9d5ef0.mockapi.io/project_1/api/List_User/${createData.value.newID}`,updateItem);
    console.log("cap nhat thanh cong :",res.data);
    // Di chuyển task giữa các mảng dựa trên status mới
    if(createData.value.status === 'todo'){
      todoTasks.value.push(res.data);
    }
    else if(createData.value.status === 'doing'){
      doingTasks.value.push(res.data);
    }
    else if(createData.value.status === 'finish'){
      finishTasks.value.push(res.data);
    }

    // Xóa task cũ khỏi mảng ban đầu
    const originalArray = [todoTasks, doingTasks, finishTasks].find(array => array.value.some(task => task.id === createData.value.newID));
    const index = originalArray.value.findIndex(task => task.id === createData.value.newID);
    if (index !== -1) {
      originalArray.value.splice(index, 1);
    }

    closePopupEdit();
    getData();
  } catch (error) {
    console.log('update du lieu that bai !',error);
  }
}






const getCurrentDateTime = () => {
  const now = new Date();
  // Định dạng ngày và giờ theo dạng 'YYYY-MM-DD HH:mm:ss'
  const year = now.getFullYear();
  const month = String(now.getMonth() + 1).padStart(2, '0');
  const day = String(now.getDate()).padStart(2, '0');
  const hours = String(now.getHours()).padStart(2, '0');
  const minutes = String(now.getMinutes()).padStart(2, '0');
  const seconds = String(now.getSeconds()).padStart(2, '0');
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};
</script>

<template>
    <div id="manager-container">
    <header>
      <div class="logo"><img src="@/assets/images/logo.png" alt=""></div>
      <button @click="showPopupCreate" class="btn-create"><Icon class="icon-add" icon="material-symbols-light:add-box-outline-rounded"></Icon>Create</button>
    </header>
    <main>
      <div class="box">
        <div class="todo-line">
          <div class="todo">
            <p>Todo</p>
            <span>{{ todoTasks.length }}</span>
          </div>
          <div v-for="(item, index) in todoTasks" :key="item.id" class="box-content">
            <div class="title">
              <h4>{{ item.categories }}</h4>
              <div class="edit">
                <button @click="showPopupEdit(item)" class="edit"><Icon icon="material-symbols:edit-document-outline-rounded"></Icon></button>
                <button @click="showPopupMess(item.id)" class="delete"><Icon icon="material-symbols:delete-outline"></Icon></button>
              </div>
            </div>
            <div class="body">
              <div class="content-top">
                <p>{{ item.title }}</p>
              </div>
              <div class="content-bottom">
                <h5>{{ item.content }}</h5>
              </div>
            </div>
            <div class="date">
              <Icon class="icon-watch" icon="material-symbols-light:alarm-smart-wake-outline"></Icon>
              <span class="hour">{{ item.date }}</span>
            </div>
          </div>
        </div>
        <div class="doing-line">
          <div class="doing">
            <p>Doing</p>
            <span>{{ doingTasks.length }}</span>
          </div>
          <div v-for="(item, index) in doingTasks" :key="item.id">
            <div class="title">
              <h4>{{ item.categories }}</h4>
              <div class="edit">
                <button @click="showPopupEdit(item)" class="edit"><Icon icon="material-symbols:edit-document-outline-rounded"></Icon></button>
                <button @click="showPopupMess(item.id)" class="delete"><Icon icon="material-symbols:delete-outline"></Icon></button>
              </div>
            </div>
            <div class="body">
              <div class="content-top">
                <p>{{ item.title }}</p>
              </div>
              <div class="content-bottom">
                <h5>{{ item.content }}</h5>
              </div>
            </div>
            <div class="date">
              <Icon class="icon-watch" icon="material-symbols-light:alarm-smart-wake-outline"></Icon>
              <span class="hour">{{ item.date }}</span>
            </div>
          </div>
        </div>
        <div class="finish-line">
          <div class="finish">
            <p>Finish</p>
            <span>{{ finishTasks.length }}</span>
          </div>
          <div v-for="(item, index) in finishTasks" :key="item.id">
            <div class="title">
              <h4>{{ item.categories }}</h4>
              <div class="edit">
                <button @click="showPopupEdit(item)" class="edit"><Icon icon="material-symbols:edit-document-outline-rounded"></Icon></button>
                <button @click="showPopupMess(item.id)" class="delete"><Icon icon="material-symbols:delete-outline"></Icon></button>
              </div>
            </div>
            <div class="body">
              <div class="content-top">
                <p>{{ item.title }}</p>
              </div>
              <div class="content-bottom">
                <h5>{{ item.content }}</h5>
              </div>
            </div>
            <div class="date">
              <Icon class="icon-watch" icon="material-symbols-light:alarm-smart-wake-outline"></Icon>
              <span class="hour">{{ item.date }}</span>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>

  <!-- popup create -->
    <div id="popup-create-container" v-if="isShowPopupCreate">
    <div class="popup">
      <Icon @click="closePopupCreate" class="icon-cancel" icon="material-symbols-light:cancel-outline-rounded"></Icon>
      <h3>Add new todo</h3>
      <form @submit.prevent="addItem" action="">
        <input v-model="dataNew.categories" type="text" name="" id="" placeholder="cotegories">
        <input v-model="dataNew.title" type="text" name="" id="" placeholder="title">
        <input v-model="dataNew.content" type="text" name="" id="" placeholder="content">
        <button type="submit" class="btn-submit">Submit</button>
      </form>
    </div>
    </div>

  <!-- popup mess -->
  <div v-if="isShowPopupMess" class="popup-overlay">
    <div class="popup-content">
      <h2>Xác nhận xóa</h2>
      <p>Bạn có chắc chắn muốn xóa dữ liệu này không?</p>
      <div class="popup-actions">
        <button @click="closePopupMess">Hủy</button>
        <button @click="deleteItem">Xóa</button>
      </div>
    </div>
  </div>


  <!-- edit -->
  <div id="popup-edit-container" v-if="isShowPopupEdit">
    <div class="popup">
      <Icon @click="closePopupEdit" class="icon-cancel" icon="material-symbols-light:cancel-outline-rounded"></Icon>
      <h3>Edit todo</h3>
      <p>Current status: {{ createData.status }}</p>
      <form @submit.prevent="updateItem" action="">
        <input v-model="createData.categories" type="text" name="" id="" placeholder="cotegories">
        <input v-model="createData.title" type="text" name="" id="" placeholder="title">
        <input v-model="createData.content" type="text" name="" id="" placeholder="content">
        <div class="checkbox">
        <div class="item">
          <input v-model="createData.status" type="radio" value="todo"  id="todo-edit">
          <label for="todo-edit">Todo</label>
        </div>
        <div class="item">
          <input v-model="createData.status" type="radio" value="doing"  id="doing-edit">
          <label for="doing-edit">Doing</label>
        </div>
        <div class="item">
          <input v-model="createData.status" type="radio" value="finish"  id="finish-edit">
          <label for="finish-edit">Finish</label>
        </div>
      </div>
        <button class="btn-submit" type="submit">Submit</button>
      </form>
    </div>
  </div>
</template>

<style scoped>
body {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
}

#manager-container {
  width: 100%;
  height: 100%;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
}

.logo img {
  width: 200px;
  height: 150px;
}

.icon-add {
  font-size: 20px;
  vertical-align: middle;
  margin-bottom: 2px;
}

.btn-create {
  padding: 10px 20px;
  border: none;
  background: none;
  font-size: 17px;
  color: #484545;
  border-radius: 5px;
}

.btn-create:hover {
  box-shadow: 0 0 10px rgb(52, 153, 49);
}

main {
  padding: 0 50px;
}

.box,
.content-wrapper {
  display: flex;
  justify-content: space-between;
  padding: 0 40px;
  gap: 30px;
}

.todo-line,
.doing-line,
.finish-line {
  flex: 1;
}

.todo,
.doing,
.finish {
  flex: 1;
  border-radius: 8px;
  text-align: center;
  display: flex;
  justify-content: space-between;
  padding: 0 10px;
  color: #f6f0f0;
  margin-bottom: 10px;
}
.todo {
  background: #00a5d9;
}

.doing {
  background: #e1d174;
}

.finish {
  background: #3cc057;
}

.box-content {
  border: 1px solid rgb(191, 191, 191);
  padding: 10px 20px;
  width: 100%;
  margin-top: 10px;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.title {
  display: flex;
  justify-content: space-between;
}

h4 {
  font-size: 15px;
}

.edit {
  margin: 0 5px;
}

.body {
  margin: 10px 0;
}

.content-top {
  border-bottom: 1px solid #ccc;
}

.content-top p {
  margin-bottom: 10px;
  font-size: 18px;
}

.content-bottom h5 {
  margin-top: 5px;
  font-size: 15px;
}

.date {
  display: flex;
  justify-content: flex-start;
  align-items: center;
}

.date .icon-watch {
  font-size: 18px;
}

.date .hour {
  font-size: 13px;
  color: #a9a7a7;
}



/* create */
#popup-create-container{
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  top: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 100%;
  background: #dad8d88f;
}
.popup{
  position: relative;
  width: 400px;
  height: auto;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.icon-cancel{
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
h3{
  text-align: center;
}
form{
  padding: 10px;
  display: flex;
  flex-direction: column;
}
input[type="text"]{
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* popup mess */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.popup-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  width: 300px;
}

.popup-actions {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  gap: 10px; 
}

.popup-actions button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.popup-actions button:first-child {
  background-color: #ccc;
}

.popup-actions button:last-child {
  background-color: #ff4d4d;
  color: white;
}

/* create edit */
#popup-edit-container{
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  top: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 100%;
  background: #dad8d88f;
}
.popup{
  position: relative;
  width: 400px;
  height: auto;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.icon-cancel{
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
h3{
  text-align: center;
}
form{
  padding: 10px;
  display: flex;
  flex-direction: column;
}
input[type="text"]{
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.checkbox{
  display: flex;
  justify-content: space-between;
}
.checkbox .item{
  margin-right: 5px;
  display: flex;
  align-items: center;
}
.item input[type="radio"]{
  margin-right: 5px;
}
.btn-submit{
  padding: 7px;
  margin-top: 10px;
  border-radius: 5px;
  border: none;
  background: #02b2e7;
  color: #f6f0f0;
  font-weight: bold;
  font-size: 17px;
}


.popup form input {
  margin-bottom: 10px;
  padding: 5px;
  width: 100%;
  box-sizing: border-box;
}
.popup form .btn-submit {
  background-color: #00a5d9;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.popup form .btn-submit:hover {
  background-color: #007da1;
}
</style>