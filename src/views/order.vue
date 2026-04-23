<template>
  <div class="main-bg py-5">
    <div class="container">

      <!-- Header -->
      <div class="d-flex justify-content-between align-items-center mb-4 text-white">
        <div>
          <h4 class="fw-bold">ยินดีต้อนรับ, user</h4>
          <small>จัดการสั่งสินค้าของคุณได้ที่นี่</small>
        </div>
        <div class="avatar">US</div>
      </div>

      <!-- TOP PANEL -->
      <div class="row g-4 mb-4">
        
        <!-- LEFT SUMMARY -->
        <div class="col-md-4">
          <div class="card custom-card p-3">
            <h6 class="fw-bold text-primary mb-3">📊 สรุปข้อมูลการซื้อของคุณ</h6>

            <div class="summary-box orange mb-3">
              <small>กำลังซื้อ</small>
              <h4>{{ totalQty }}</h4>
            </div>

           
            
          </div>
        </div>

        <!-- RIGHT CART -->
        <div class="col-md-8">
          <div class="card custom-card p-3">
            <div class="d-flex justify-content-between mb-3">
              <h6 class="fw-bold text-primary">🛒 รายการที่เลือก</h6>
              <span class="dot"></span>
            </div>

            <!-- ✅ กัน error ตรงนี้ -->
            <div v-if="selectedItems.length === 0" class="text-center text-muted py-5">
              ยังไม่มีรายการที่เลือก
            </div>

            <div v-else>
              <div 
                v-for="item in selectedItems" 
                :key="item.id" 
                class="cart-item d-flex justify-content-between"
              >
                <span>{{ item.name }}</span>
                <span>x {{ item.orderQty }}</span>
              </div>

              <div class="text-end mt-3">
            <button @click="openEmailModal">
              ยืนยันการทำรายการ
            </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- PRODUCT LIST -->
      <div class="card custom-card p-4">
        <h5 class="fw-bold text-primary mb-3">🛠 อุปกรณ์ที่สามารถซื้อได้</h5>

        <!-- loading -->
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-light"></div>
        </div>

        <div v-else class="row g-4">
          <div v-for="item in products" :key="item.id" class="col-md-4">
            <div class="product-card">

              <img src="https://picsum.photos/400/200" class="img-fluid rounded mb-2"/>

              <span class="badge bg-light text-primary mb-2">Electronics</span>

              <h6 class="fw-bold">{{ item.name }}</h6>
              <p class="text-muted">฿ {{ item.price }}</p>

             <input 
  type="number"
  v-model.number="item.inputQty"
  min="1"
  :max="item.pcs"
  class="form-control mb-2 text-center"
  placeholder="จำนวน"
/>

<button 
  class="btn btn-primary w-100 rounded-pill"
  @click="addToCart(item)"
>
  เพิ่มสินค้า
</button>
            </div>
          </div>
        </div>

      </div>

    </div>
  </div>
  <!-- EMAIL MODAL -->
<div v-if="showEmailModal" class="modal-overlay">
  <div class="modal-box shadow-lg">
    <h5 class="fw-bold mb-3 text-primary">📧 ระบุอีเมลเพื่อรับใบสั่งซื้อ</h5>

    <input 
      type="email"
      v-model="customerEmail"
      class="form-control mb-3 rounded-pill"
      placeholder="example@email.com"
      :disabled="ordering"
    />

    <div class="text-end">
      <button class="btn btn-light rounded-pill me-2" @click="showEmailModal = false" :disabled="ordering">
        ยกเลิก
      </button>

      <button class="btn btn-primary rounded-pill px-4" @click="confirmOrder" :disabled="ordering">
        <span v-if="ordering" class="spinner-border spinner-border-sm me-2"></span>
        {{ ordering ? 'กำลังส่ง...' : 'ส่งข้อมูล' }}
      </button>
    </div>
  </div>
</div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

/* ================= STATE ================= */
const products = ref([])
const loading = ref(false)
const ordering = ref(false)

const showEmailModal = ref(false)
const customerEmail = ref('')

/* ================= FETCH DATA ================= */
const fetchData = async () => {
  loading.value = true
  try {
    const res = await fetch('http://localhost:5678/webhook/data')
    const data = await res.json()

    products.value = data.map(item => ({
      id: item.id,
      name: item["ืname"] || item.name || "ไม่มีชื่อสินค้า",
      pcs: item.pcs || 0,
      price: item.price || 0,
      orderQty: 0,
      inputQty: 1 // ✅ เพิ่มแล้ว
    }))

  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

/* ================= COMPUTED ================= */
const selectedItems = computed(() => {
  return products.value.filter(i => i.orderQty > 0)
})

const totalQty = computed(() => {
  return products.value.reduce((sum, i) => sum + i.orderQty, 0)
})

const totalPrice = computed(() => {
  return products.value.reduce((sum, i) => sum + (i.orderQty * i.price), 0)
})

/* ================= CART ================= */
const addToCart = (item) => {
  if (!item.inputQty || item.inputQty <= 0) {
    alert('กรุณาระบุจำนวน')
    return
  }

  if (item.inputQty > item.pcs) {
    alert('สินค้าไม่พอ')
    return
  }

  item.orderQty += item.inputQty
  item.inputQty = 1
}

/* ================= MODAL ================= */
const openEmailModal = () => {
  if (selectedItems.value.length === 0) {
    alert('ยังไม่มีสินค้าในรายการ')
    return
  }
  showEmailModal.value = true
}

/* ================= ORDER (เหลือตัวเดียว!) ================= */
const confirmOrder = async () => {
  // 1. ตรวจสอบอีเมล
  if (!customerEmail.value || !customerEmail.value.includes('@')) {
    alert('กรุณากรอกอีเมลให้ถูกต้อง')
    return
  }

  ordering.value = true

  try {
    const res = await fetch('http://localhost:5678/webhook/order', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: customerEmail.value, // ตรวจสอบว่าใน n8n ใช้ตัวแปรชื่อ email
        items: selectedItems.value,
        total_amount: totalPrice.value,
        order_date: new Date().toLocaleString('th-TH')
      })
    })

    if (res.ok) {
      alert('🌟 สั่งซื้อและส่งอีเมลเรียบร้อยแล้ว!')
      
      // ล้างข้อมูลหลังส่งสำเร็จ
      products.value.forEach(i => i.orderQty = 0)
      customerEmail.value = ''
      showEmailModal.value = false
    } else {
      const errorData = await res.text();
      console.error('Server Error:', errorData);
      alert('เกิดข้อผิดพลาดจากเซิร์ฟเวอร์ n8n');
    }

  } catch (err) {
    console.error('Fetch Error:', err)
    alert('ไม่สามารถเชื่อมต่อกับ n8n ได้ (กรุณาเช็กว่าเปิด n8n ทิ้งไว้หรือไม่)')
  } finally {
    ordering.value = false
  }
}

onMounted(fetchData)
</script>

<style>
body {
  font-family: 'Sarabun', sans-serif;
}

/* BG */
.main-bg {
  background: linear-gradient(135deg, #5f6cff, #8e44ad);
  min-height: 100vh;
}

/* CARD */
.custom-card {
  border-radius: 20px;
  border: none;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

/* AVATAR */
.avatar {
  width: 40px;
  height: 40px;
  background: #a855f7;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
}

/* SUMMARY */
.summary-box {
  padding: 15px;
  border-radius: 15px;
}

.orange { background: #fce7d6; }
.blue { background: #dbeafe; }

/* DOT */
.dot {
  width: 12px;
  height: 12px;
  background: red;
  border-radius: 50%;
}

/* PRODUCT */
.product-card {
  background: #fff;
  border-radius: 15px;
  padding: 15px;
  transition: 0.2s;
}

.product-card:hover {
  transform: translateY(-5px);
}

/* CART */
.cart-item {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-box {
  background: white;
  padding: 25px;
  border-radius: 15px;
  width: 300px;
}
</style>