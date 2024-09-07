<script setup lang="ts">
import { ref, computed } from "vue"
import axios from "axios"

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

const members = ref<Member[]>([])
const books = ref<Book[]>([])
const borrowBooks = ref<BorrowBook[]>([])
const search = ref("")

async function getMembers() {
  try { 
    const res = await axios.get<Member[]>("http://localhost:3001/members")
    members.value = res.data
  } catch (error) {
    console.error("Error fetching members:", error)
  }
}

async function getBooks() {
  try { 
    const res = await axios.get<Book[]>("http://localhost:3001/books")
    books.value = res.data
  } catch (error) {
    console.error("Error fetching books:", error)
  }
}

async function getBorrowBooks() {
  try { 
    const res = await axios.get<BorrowBook[]>("http://localhost:3001/borrows")
    borrowBooks.value = res.data
  } catch (error) {
    console.error("Error fetching borrow books:", error)
  }
}

const filteredBorrowBooks = computed(() => {
  return borrowBooks.value.filter(borrow => {
    const book = books.value.find(b => b.b_id === borrow.b_id)
    const member = members.value.find(m => m.m_user === borrow.m_user)
    const searchLower = search.value.toLowerCase()

    return book && member && (
      book.b_name.toLowerCase().includes(searchLower) ||
      member.m_name.toLowerCase().includes(searchLower)
    )

  })
})

await getMembers()
await getBooks()
await getBorrowBooks()
</script>

<template>
  <div class="container ">
    <div class="text-center text-3xl font-bold">
      การจัดการข้อมูลการยืม-คืนหนังสือ
    </div>

    <div class="items-center justify-center text-center mt-6">
      <input
        v-model="search"
        type="text"
        placeholder="ค้นหา (ชื่อหนังสือ และผู้ยืม-คืน)"
        class="input input-bordered w-full max-w-xs"
      />
    </div>

    <div class="space-x-2 mt-4 justify-end text-end">
      <span><nuxt-link to="/borrow" class="btn btn-primary w-30">ยืม-คืนหนังสือ</nuxt-link></span>
      <span><nuxt-link to="/datapage" class="btn btn-primary w-30">ข้อมูลสถิติ</nuxt-link></span>
    </div>

    <div class="overflow-x-auto mt-6">
      <table class="table">
        <thead>
          <tr>
            <th>รหัสหนังสือ</th>
            <th>ชื่อหนังสือ</th>
            <th>ผู้ยืม-คืน</th>
            <th>วันที่ยืม</th>
            <th>วันที่คืน</th>
            <th>ค่าปรับ</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="borrow in filteredBorrowBooks" :key="borrow.b_id + borrow.m_user">
            <td>{{ borrow.b_id }}</td>
            <td>{{ books.find(b => b.b_id === borrow.b_id)?.b_name }}</td>
            <td>{{ members.find(m => m.m_user === borrow.m_user)?.m_name }}</td>
            <td>{{ borrow.br_date_br }}</td>
            <td>{{ borrow.br_date_rt }}</td>
            <td>{{ borrow.br_fine }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped></style>