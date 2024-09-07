<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios'; // Import Axios

interface StatisticItem {
  title: string;
  value: number;
  icon: string;
}

interface Member {
  m_user: string
  m_pass: string
  m_name: string
  m_phone: string
}

interface Book {
  b_id: string
  b_name: string
  b_writer: string
  b_category: string
  b_price: string
}

interface BorrowBook {
  br_date_br: string
  br_date_rt: string
  b_id: string
  m_user: string
  br_fine: number
}

const statistics = ref<StatisticItem[]>([
  { title: 'หนังสือทั้งหมด (เล่ม)', value: 0, icon: 'book' },
  { title: 'สมาชิกทั้งหมด (คน)', value: 0, icon: 'users' },
  { title: 'การใช้บริการยืม-คืนหนังสือ (ครั้ง)', value: 0, icon: 'repeat' },
  { title: 'หนังสือค้างส่ง (เล่ม)', value: 0, icon: 'alert-triangle' },
]);

// Replace your database calls with Axios API requests
onMounted(async () => {
  try {
    const [booksRes, membersRes, borrowBooksRes] = await Promise.all([
      axios.get('http://localhost:3001/books'),       // Replace with your API endpoint
      axios.get('http://localhost:3001/members'),     // Replace with your API endpoint
      axios.get('http://localhost:3001/borrows')  // Replace with your API endpoint
    ]);

    const books: Book[] = booksRes.data;
    const members: Member[] = membersRes.data;
    const borrowBooks: BorrowBook[] = borrowBooksRes.data;

    // Populate statistics
    statistics.value[0].value = books.length;
    statistics.value[1].value = members.length;
    statistics.value[2].value = borrowBooks.length;
    statistics.value[3].value = borrowBooks.filter(book => book.br_date_rt === '0000-00-00').length;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
});
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <h1 class="text-3xl font-bold text-center mb-8">ข้อมูลสถิติของห้องสมุด</h1>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
      <div v-for="stat in statistics" :key="stat.title" class="card bg-base-100 shadow-xl">
        <div class="card-body items-center text-center">
          <div class="stat-figure text-secondary">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="inline-block w-8 h-8 stroke-current">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" :d="getIconPath(stat.icon)"></path>
            </svg>
          </div>
          <h2 class="card-title">{{ stat.title }}</h2>
          <p class="text-4xl font-bold">{{ stat.value }}</p>
        </div>
      </div>
    </div>

    <div class="mt-8 text-center">
      <div @click="$router.go(-1)" class="btn btn-primary">กลับไปหน้าเดิม</div>
    </div>
  </div>
</template>

<script lang="ts">
function getIconPath(icon: string): string {
  switch (icon) {
    case 'book':
      return "M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253";
    case 'users':
      return "M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z";
    case 'repeat':
      return "M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15";
    case 'alert-triangle':
      return "M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z";
    default:
      return "";
  }
}
</script>
