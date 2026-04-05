<template>
  <div class="container py-5">

    <div class="text-center mb-4">
      <h2 class="fw-bold text-primary">📦 ระบบสต็อกสินค้า</h2>
      <p class="text-muted">ข้อมูลสินค้าจาก n8n Webhook API</p>
    </div>

    <div class="card shadow-lg border-0 rounded-4">
      <div class="card-body">

        <div class="d-flex justify-content-between align-items-center mb-3">
          <h5 class="mb-0">รายการสินค้าปัจจุบัน</h5>
          <button class="btn btn-primary" @click="fetchData" :disabled="loading">
            <span v-if="loading" class="spinner-border spinner-border-sm me-1"></span>
            🔄 {{ loading ? 'กำลังโหลด...' : 'รีเฟรชข้อมูล' }}
          </button>
        </div>

        <div v-if="loading" class="text-center my-4">
          <div class="spinner-border text-primary"></div>
          <p class="mt-2 text-muted">กำลังดึงข้อมูลจากระบบ...</p>
        </div>

        <div class="table-responsive" v-if="!loading && products.length">
          <table class="table table-hover align-middle">
            <thead class="table-primary text-center">
              <tr>
                <th style="width: 50px">#</th>
                <th style="width: 150px">รายชื่ออุปกรณ์</th>
                <th class="text-start">ชื่อ-นามสกุล</th> <th style="width: 150px">แผนก</th>
                <th style="width: 180px">จำนวนที่เบิก (ชิ้น)</th>
              </tr>
            </thead>
            <tbody class="text-center">
              <tr v-for="(item, index) in products" :key="index">
                <td>{{ index + 1 }}</td>
                
                <td>
                  <span class="badge bg-light text-dark border">{{ item.tool }}</span>
                </td>
                
                <td class="text-start ps-3">{{ item.name }}</td>
                
                <td>{{ item.position }}</td>
                
                <td class="fw-bold text-success">{{ item.pcs }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else-if="!loading" class="text-center text-muted py-5">
          <div class="mb-2" style="font-size: 2rem;">❌</div>
          <h5>ไม่พบข้อมูลสินค้า</h5>
          <p>โปรดตรวจสอบการเชื่อมต่อกับ n8n หรือ URL ของ Webhook</p>
        </div>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const products = ref([])
const loading = ref(false)

const fetchData = async () => {
  loading.value = true
  try {
    // ดึงข้อมูลจาก n8n Webhook
    const response = await fetch('http://localhost:5678/webhook/showdata')
    if (!response.ok) throw new Error('Network response was not ok')
    
    const data = await response.json()
    
    // ตรวจสอบโครงสร้างข้อมูลที่ส่งมาจาก n8n
    if (Array.isArray(data)) {
      products.value = data
    } else if (data.data && Array.isArray(data.data)) {
      products.value = data.data
    } else {
      products.value = []
    }
  } catch (error) {
    console.error('Error fetching data:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style>
body {
  background-color: #f8f9fa;
}
.table thead th {
  font-weight: 600;
  white-space: nowrap;
}
.card {
  transition: transform 0.2s;
}
/* เพิ่มระยะห่างให้ชื่อดูไม่ติดเส้นขอบเกินไป */
.ps-3 {
  padding-left: 1rem !important;
}
</style>