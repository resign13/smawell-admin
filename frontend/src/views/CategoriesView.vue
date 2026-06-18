<template>
  <AdminLayout>
    <div class="admin-page two-col">
      <section class="admin-card">
        <h1>分类管理</h1>
        <div v-if="!admin.categories.length" class="empty-state">暂无分类</div>
        <div v-for="item in admin.categories" :key="item.id" class="list-row stack-row category-list-row">
          <img class="category-thumb" :src="item.imageUrl || defaultCategoryImage" :alt="item.labels?.en || item.label || item.key" />
          <div>
            <strong>{{ item.labels?.en || item.label || item.key }}</strong>
            <p>{{ item.key }} / 排序 {{ item.sortOrder }}</p>
            <p>关联商品 {{ item.productCount }}</p>
          </div>
          <div class="inline-actions">
            <button class="admin-button ghost" type="button" @click="editItem(item)">编辑</button>
            <button class="admin-button ghost" type="button" @click="removeItem(item.id)">删除</button>
          </div>
        </div>
      </section>

      <section class="admin-card">
        <h2>{{ form.id ? '编辑分类' : '新建分类' }}</h2>
        <form class="editor-form" @submit.prevent="save">
          <input
            v-model.trim="form.key"
            class="admin-field"
            placeholder="分类编码，例如 womenswear"
          />
          <input
            v-model.number="form.sortOrder"
            class="admin-field"
            type="number"
            min="0"
            placeholder="排序值"
          />
          <textarea
            v-model.trim="form.labels.en"
            class="admin-field admin-textarea"
            placeholder="英文分类名"
          ></textarea>
          <textarea
            v-model.trim="form.labels.zh"
            class="admin-field admin-textarea"
            placeholder="中文分类名"
          ></textarea>
          <div class="category-image-editor">
            <div class="category-image-preview">
              <img :src="form.imageUrl || defaultCategoryImage" alt="Category preview" />
            </div>
            <div class="category-image-actions">
              <label class="admin-button ghost upload-button">
                {{ uploadingImage ? '上传中...' : '上传分类图片' }}
                <input type="file" accept="image/jpeg,image/png,image/webp" :disabled="uploadingImage" @change="handleImageUpload" />
              </label>
              <button v-if="form.imageUrl" class="admin-button ghost" type="button" @click="form.imageUrl = ''">
                使用默认图片
              </button>
              <p>未上传时，商城首页会自动使用默认分类图。</p>
            </div>
          </div>
          <p v-if="errorText" class="admin-error">{{ errorText }}</p>
          <button class="admin-button" type="submit">
            {{ saving ? '保存中...' : '保存分类' }}
          </button>
        </form>
      </section>
    </div>
  </AdminLayout>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'

import AdminLayout from '../components/AdminLayout.vue'
import { useAdminStore } from '../stores/admin'

const admin = useAdminStore()
const saving = ref(false)
const uploadingImage = ref(false)
const errorText = ref('')
const defaultCategoryImage = '/media/storefront/womenswear-blouse-1.jpg'

function emptyForm() {
  return {
    id: null,
    key: '',
    sortOrder: 0,
    imageUrl: '',
    labels: { zh: '', en: '' },
  }
}

const form = reactive(emptyForm())

function resetForm() {
  Object.assign(form, emptyForm())
  errorText.value = ''
}

function editItem(item) {
  form.id = item.id
  form.key = item.key
  form.sortOrder = item.sortOrder
  form.labels = {
    zh: item.labels?.zh || '',
    en: item.labels?.en || '',
  }
  form.imageUrl = item.imageUrl || ''
  errorText.value = ''
}

async function handleImageUpload(event) {
  const file = event.target.files?.[0]
  event.target.value = ''
  if (!file) return
  errorText.value = ''
  uploadingImage.value = true
  try {
    const urls = await admin.uploadFiles([file])
    form.imageUrl = urls[0] || ''
  } catch (error) {
    errorText.value = error.message || '上传分类图片失败'
  } finally {
    uploadingImage.value = false
  }
}

async function save() {
  errorText.value = ''
  saving.value = true
  try {
    await admin.saveCategory({
      id: form.id,
      key: form.key.trim().toLowerCase(),
      sortOrder: Number(form.sortOrder || 0),
      imageUrl: form.imageUrl,
      labels: { ...form.labels },
    })
    resetForm()
  } catch (error) {
    errorText.value = error.message || '保存分类失败'
  } finally {
    saving.value = false
  }
}

async function removeItem(id) {
  errorText.value = ''
  try {
    await admin.deleteCategory(id)
    if (form.id === id) {
      resetForm()
    }
  } catch (error) {
    errorText.value = error.message || '删除分类失败'
  }
}

onMounted(() => {
  admin.loadCategories()
})
</script>

<style scoped>
.category-list-row {
  display: grid;
  grid-template-columns: 88px 1fr auto;
  gap: 14px;
  align-items: center;
}

.category-thumb,
.category-image-preview img {
  width: 88px;
  height: 64px;
  object-fit: cover;
  border-radius: 12px;
  background: #f1f5f9;
}

.category-image-editor {
  display: flex;
  gap: 14px;
  align-items: center;
  padding: 12px;
  border: 1px solid #e5e7eb;
  border-radius: 14px;
  background: #f8fafc;
}

.category-image-preview img {
  width: 140px;
  height: 96px;
}

.category-image-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.category-image-actions p {
  margin: 0;
  color: #64748b;
  font-size: 12px;
}

.upload-button {
  position: relative;
  overflow: hidden;
  width: max-content;
}

.upload-button input {
  position: absolute;
  inset: 0;
  opacity: 0;
  cursor: pointer;
}
</style>
