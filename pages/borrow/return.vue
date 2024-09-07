<script setup lang="ts">
import { ref, reactive } from 'vue';
import axios from 'axios';

interface Book {
  b_id: string;
  b_name: string;
}

interface Member {
  m_user: string;
  m_name: string;
}

interface BorrowInfo {
  b_id: string;
  b_name: string;
  m_user: string;
  m_name: string;
  br_date_rt: string;
  br_fine: number;
}

const bookId = ref('');
const borrowInfo = reactive<BorrowInfo>({
  b_id: '',
  b_name: '',
  m_user: '',
  m_name: '',
  br_date_rt: '',
  br_fine: 0,
});

const errorMessage = ref('');
const successMessage = ref('');

async function searchBook() {
  try {
    // Fetch book information
    const bookResponse = await axios.get<Book>(`http://localhost:3001/books/${bookId.value}`);
    borrowInfo.b_id = bookResponse.data.b_id;
    borrowInfo.b_name = bookResponse.data.b_name;

    // Fetch borrow information
    const borrowResponse = await axios.get<any>(`http://localhost:3001/borrows?b_id=${bookId.value}&_sort=br_date_br&_order=desc&_limit=1`);
    if (borrowResponse.data.length === 0) {
      throw new Error('ไม่พบข้อมูลการยืมหนังสือ');
    }

    const borrowData = borrowResponse.data[0];
    borrowInfo.m_user = borrowData.m_user;
    borrowInfo.br_date_rt = new Date().toISOString().split('T')[0]; // Set return date to today
    borrowInfo.br_fine = calculateFine(borrowData.br_date_br, borrowInfo.br_date_rt);

    // Fetch member information
    const memberResponse = await axios.get<Member>(`http://localhost:3001/members/${borrowInfo.m_user}`);
    borrowInfo.m_name = memberResponse.data.m_name;

    errorMessage.value = '';
  } catch (error) {
    errorMessage.value = 'ไม่พบข้อมูลหนังสือหรือการยืม กรุณาตรวจสอบรหัสหนังสือ';
    resetBorrowInfo();
  }
}

function calculateFine(borrowDate: string, returnDate: string): number {
  const millisecondsPerDay = 24 * 60 * 60 * 1000;
  const borrowTime = new Date(borrowDate).getTime();
  const returnTime = new Date(returnDate).getTime();
  const daysDifference = Math.floor((returnTime - borrowTime) / millisecondsPerDay);

  // Assume fine is 5 baht per day after 7 days
  return daysDifference > 7 ? (daysDifference - 7) * 5 : 0;
}

async function returnBook() {
  if (!borrowInfo.b_id) {
    errorMessage.value = 'กรุณาค้นหาหนังสือก่อนทำการคืน';
    return;
  }
  try {
    await axios.patch(`http://localhost:3001/borrows/${borrowInfo.b_id}`, {
      br_date_rt: borrowInfo.br_date_rt,
      br_fine: borrowInfo.br_fine,
    });
    successMessage.value = 'คืนหนังสือสำเร็จ';
    resetForm();
  } catch (error) {
    errorMessage.value = 'เกิดข้อผิดพลาดในการคืนหนังสือ';
  }
}

function resetBorrowInfo() {
  borrowInfo.b_id = '';
  borrowInfo.b_name = '';
  borrowInfo.m_user = '';
  borrowInfo.m_name = '';
  borrowInfo.br_date_rt = '';
  borrowInfo.br_fine = 0;
}

function resetForm() {
  bookId.value = '';
  resetBorrowInfo();
  errorMessage.value = '';
  successMessage.value = '';
}

function cancelReturn() {
  resetForm();
  // นำทางกลับไปยังหน้าหลัก (ต้องกำหนด route ที่ถูกต้อง)
  // router.push('/');
}
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <div class="flex justify-center space-x-4 mb-8">
      <nuxt-link to="/borrow" class="btn btn-secondary text-xl">ยืมหนังสือ</nuxt-link>
      <nuxt-link to="/borrow/return" class="btn btn-primary text-xl">คืนหนังสือ</nuxt-link>
    </div>

    <h1 class="text-3xl font-bold text-center mb-8">คืนหนังสือ</h1>

    <div class="max-w-2xl mx-auto space-y-6">
      <div class="flex items-center space-x-4">
        <label class="w-48 text-right">รหัสหนังสือที่ต้องการคืน:</label>
        <input v-model="bookId" placeholder="กรอกรหัสหนังสือ" class="input input-bordered flex-grow" type="text">
        <button @click="searchBook" class="btn btn-primary">ค้นหา</button>
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
            <td class="font-bold">รหัสหนังสือ</td>
            <td>{{ borrowInfo.b_id }}</td>
          </tr>
          <tr>
            <td class="font-bold">ชื่อหนังสือ</td>
            <td>{{ borrowInfo.b_name }}</td>
          </tr>
          <tr>
            <td class="font-bold">ผู้ยืม-คืน</td>
            <td>{{ borrowInfo.m_name }}</td>
          </tr>
          <tr>
            <td class="font-bold">วันที่คืน</td>
            <td>{{ borrowInfo.br_date_rt }}</td>
          </tr>
          <tr>
            <td class="font-bold">ค่าปรับ</td>
            <td>{{ borrowInfo.br_fine }} บาท</td>
          </tr>
        </tbody>
      </table>

      <div v-if="borrowInfo.b_id" class="flex justify-center space-x-4 mt-8">
        <button @click="returnBook" class="btn btn-success text-xl text-white">คืนหนังสือ</button>
        <button @click="cancelReturn" class="btn btn-error text-xl text-white">ยกเลิก</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>