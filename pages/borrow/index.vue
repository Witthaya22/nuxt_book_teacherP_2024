<script setup lang="ts">
import { ref, reactive } from 'vue';
import axios from 'axios';

interface Member {
  m_user: string;
  m_name: string;
}

interface Book {
  b_id: string;
  b_name: string;
}

const memberName = ref('');
const bookId = ref('');
const borrowInfo = reactive({
  memberUsername: '',
  memberName: '',
  bookId: '',
  bookName: '',
});

const errorMessage = ref('');
const successMessage = ref('');

async function searchMember() {
  try {
    const response = await axios.get<Member[]>(`http://localhost:3001/members?name=${memberName.value}`);
    if (response.data.length > 0) {
      borrowInfo.memberUsername = response.data[0].m_user;
      borrowInfo.memberName = response.data[0].m_name;
    } else {
      // If member not found, create a new one
      const newMember = await axios.post<Member>('http://localhost:3001/members', {
        m_name: memberName.value
      });
      borrowInfo.memberUsername = newMember.data.m_user;
      borrowInfo.memberName = newMember.data.m_name;
    }
    errorMessage.value = '';
  } catch (error) {
    errorMessage.value = 'เกิดข้อผิดพลาดในการค้นหาหรือสร้างสมาชิก';
    borrowInfo.memberUsername = '';
    borrowInfo.memberName = '';
  }
}

async function searchBook() {
  try {
    const response = await axios.get<Book>(`http://localhost:3001/books/${bookId.value}`);
    borrowInfo.bookId = response.data.b_id;
    borrowInfo.bookName = response.data.b_name;
    errorMessage.value = '';
  } catch (error) {
    errorMessage.value = 'ไม่พบข้อมูลหนังสือ กรุณาตรวจสอบรหัสหนังสือ';
    borrowInfo.bookId = '';
    borrowInfo.bookName = '';
  }
}

async function borrowBook() {
  if (!borrowInfo.memberUsername || !borrowInfo.bookId) {
    errorMessage.value = 'กรุณากรอกข้อมูลให้ครบถ้วน';
    return;
  }
  try {
    await axios.post('http://localhost:3001/borrows', {
      m_user: borrowInfo.memberUsername,
      b_id: borrowInfo.bookId,
    });
    successMessage.value = 'ยืมหนังสือสำเร็จ';
    resetForm();
  } catch (error) {
    errorMessage.value = 'เกิดข้อผิดพลาดในการยืมหนังสือ';
  }
}

function resetForm() {
  memberName.value = '';
  bookId.value = '';
  borrowInfo.memberUsername = '';
  borrowInfo.memberName = '';
  borrowInfo.bookId = '';
  borrowInfo.bookName = '';
  errorMessage.value = '';
  successMessage.value = '';
}

function cancelBorrow() {
  resetForm();
  // นำทางกลับไปยังหน้าหลัก (ต้องกำหนด route ที่ถูกต้อง)
  // router.push('/');
}
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <div class="flex justify-center space-x-4 mb-8">
      <nuxt-link to="/borrow" class="btn btn-primary text-xl">ยืมหนังสือ</nuxt-link>
      <nuxt-link to="/borrow/return" class="btn btn-secondary text-xl">คืนหนังสือ</nuxt-link>
    </div>

    <h1 class="text-3xl font-bold text-center mb-8">ยืมหนังสือ</h1>

    <div class="max-w-2xl mx-auto space-y-6">
      <div class="flex items-center space-x-4">
        <label class="w-48 text-right">ผู้ที่ต้องการยืมหนังสือ:</label>
        <input v-model="memberName" placeholder="กรอกชื่อ ID ผู้ใช้" class="input input-bordered flex-grow" type="text">
        <button @click="searchMember" class="btn btn-primary">ตกลง</button>
      </div>

      <div class="flex items-center space-x-4">
        <label class="w-48 text-right">รหัสหนังสือ:</label>
        <input v-model="bookId" placeholder="กรอกรหัสหนังสือ" class="input input-bordered flex-grow" type="text">
        <button @click="searchBook" class="btn btn-primary">ตกลง</button>
      </div>

      <div v-if="errorMessage" class="alert alert-error shadow-lg">
        <div>
          <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current flex-shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
          <span>{{ errorMessage }}</span>
        </div>
      </div>

      <div v-if="successMessage" class="alert alert-success shadow-lg">
        <div>
          <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current flex-shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
          <span>{{ successMessage }}</span>
        </div>
      </div>

      <table class="table w-full">
        <tbody>
          <tr>
            <td class="font-bold">ชื่อ-สกุลผู้ยืม</td>
            <td>{{ borrowInfo.memberName }}</td>
          </tr>
          <tr>
            <td class="font-bold">รหัสหนังสือ</td>
            <td>{{ borrowInfo.bookId }}</td>
          </tr>
          <tr>
            <td class="font-bold">ชื่อหนังสือ</td>
            <td>{{ borrowInfo.bookName }}</td>
          </tr>
        </tbody>
      </table>

      <div class="flex justify-center space-x-4 mt-8">
        <button @click="borrowBook" class="btn btn-success text-xl text-white">ยืมหนังสือ</button>
        <button @click="cancelBorrow" class="btn btn-error text-xl text-white">ยกเลิก</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>