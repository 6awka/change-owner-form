<script setup>
import { ref, computed } from 'vue'
import { vMaska } from 'maska/vue'

const responseType = ref('email')

const form = ref({
  lastName: '',
  firstName: '',
  middleName: '',
  email: '',
  phone: '',
  address: '',
  accountNumber: '',
  question: '',
  terms: false,
})

const files = ref([])
const fileError = ref('')
const touched = ref({})

const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function validateField(name) {
  const value = form.value[name]

  switch (name) {
    case 'lastName':
    case 'firstName':
    case 'address':
    case 'question':
      return value.trim().length > 0

    case 'email':
      return emailRegex.test(value.trim())

    case 'phone':
      return value.replace(/\D/g, '').length === 11

    default:
      return true
  }
}

function handleBlur(name) {
  touched.value[name] = true
}

const isFormValid = computed(() => {
  const requiredFields = ['lastName', 'firstName', 'email', 'phone', 'address', 'question']

  const fieldsValid = requiredFields.every(validateField)
  const termsValid = form.value.terms
  const filesValid = files.value.length > 0 && !fileError.value

  return fieldsValid && termsValid && filesValid
})

function handleFiles(event) {
  const selectedFiles = Array.from(event.target.files)

  const allowedExtensions = ['pdf', 'jpg', 'jpeg', 'png']
  const maxSize = 20 * 1024 * 1024

  fileError.value = ''

  if (selectedFiles.length > 5) {
    fileError.value = 'Можно прикрепить не более 5 файлов.'
  } else if (
    selectedFiles.some((file) => {
      const extension = file.name.split('.').pop().toLowerCase()
      return !allowedExtensions.includes(extension)
    })
  ) {
    fileError.value = 'Разрешены только файлы PDF, JPG, JPEG и PNG.'
  } else if (selectedFiles.some((file) => file.size > maxSize)) {
    fileError.value = 'Размер каждого файла не должен превышать 20 МБ.'
  }

  if (fileError.value) {
    alert(fileError.value)
    files.value = []
    event.target.value = ''
    return
  }

   files.value = selectedFiles.map(file => ({
    id: crypto.randomUUID(),
    file,
  }))

}

function submitForm() {
  if (!isFormValid.value) return

  alert('Успешно отправлено')
}
</script>

<template>
  <main class="page">
    <form class="owner-form" @submit.prevent="submitForm">
      <div class="hat">
        <h1>СМЕНА СОБСТВЕННИКА</h1>
        <button class="ESIA-enter-button" type="button">Войти через ЕСИА</button>
      </div>

      <p>Для отправки обращения пользователю необходимо пройти авторизацию через ЕСИА</p>
      <label class="terms">
        <input
          v-model="form.terms"
          type="checkbox"
          :class="{ invalid: touched.terms && !form.terms }"
          @change="touched.terms = true"
          required
        />
        Я ознакомлен(а) и принимаю
        <a href="https://example.com">политику обработки персональных данных</a>
      </label>

      <section class="form-section">
        <h2>Персональные данные</h2>
        <div class="inputs">
          <input
            v-model="form.lastName"
            type="text"
            :class="{ invalid: touched.lastName && !validateField('lastName') }"
            @blur="handleBlur('lastName')"
            placeholder="Фамилия*"
            required
          />

          <input
            v-model="form.firstName"
            type="text"
            :class="{ invalid: touched.firstName && !validateField('firstName') }"
            @blur="handleBlur('firstName')"
            placeholder="Имя*"
            required
          />

          <input v-model="form.middleName" type="text" placeholder="Отчество" />

          <input
            v-model="form.email"
            type="email"
            :class="{ invalid: touched.email && !validateField('email') }"
            @blur="handleBlur('email')"
            placeholder="you@example.com*"
            required
          />

          <input
            v-model="form.phone"
            v-maska
            data-maska="+7 (###) ###-##-##"
            type="tel"
            placeholder="+7 (___) ___-__-__*"
            :class="{ invalid: touched.phone && !validateField('phone') }"
            @blur="handleBlur('phone')"
            required
          />
        </div>
      </section>

      <section class="form-section">
        <h2>Данные лицевого счёта</h2>

        <div class="inputs">
          <input
            v-model="form.address"
            type="text"
            :class="{ invalid: touched.address && !validateField('address') }"
            @blur="handleBlur('address')"
            placeholder="Адрес помещения*"
            required
          />

          <input v-model="form.accountNumber" type="text" placeholder="Номер лицевого счета" />
        </div>
      </section>

      <div class="question-files-field">
        <div class="question-field">
          <section class="question-section">
            <section class="form-section">
              <h2>Ваш вопрос</h2>

              <textarea
                v-model="form.question"
                rows="4"
                :class="{ invalid: touched.question && !validateField('question') }"
                @blur="handleBlur('question')"
                placeholder="Опишите проблему*"
                required
              ></textarea>
            </section>

            <section class="answer-field">
              <label class="radio-option">
                <input v-model="responseType" type="radio" value="none" />
                Ответ не требуется
              </label>

              <label class="radio-option">
                <input v-model="responseType" type="radio" value="email" />
                Ответ в соответствии с 59-ФЗ в 30-дневный срок на адрес электронной почты
              </label>
            </section>
          </section>
        </div>

        <section class="form-section" name="files">
          <label class="upload-area">
            <h2>Копия документа, подтверждающего право собственности*</h2>
            <input type="file" accept=".pdf,.jpg,.jpeg,.png" multiple @change="handleFiles" />
            <span class="upload-icon">Добавить файл</span>
            <span class="upload-hint"
              >Прикрепить можно 5 файлов формата PDF, JPEG, PNG размером до 20 МБ</span
            >
          </label>

          <ul v-if="files.length" class="file-list">
            <li v-for="item in files" :key="item.id">
              {{ item.file.name }}
            </li>
          </ul>
        </section>
      </div>

      <button class="submit-button" type="submit" :disabled="!isFormValid">
        Отправить обращение
      </button>

      <section class="footer-field">
        <footer>Поля, отмеченные *, обязательны для заполнения</footer>

        <footer>
          Ваше обращение будет направлено на рассмотрение ответсвенному специалисту ФКР Липецкой
          Области. Изменения, согласно Вашему обращению, будут внесены в срок не более 30 дней с
          даты регистрации. На удовлетворительное обращение, результат рассмотрения которого будет
          отражен в платежном документе, ответ не направляется.
        </footer>
      </section>
    </form>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Rubik:wght@400;500;700&display=swap');

* {
  box-sizing: border-box;
}

body {
  display: flex;
  justify-content: center;
  font-family: 'Rubik', Arial, sans-serif;
  font-weight: 400;
  color: #252a34;
}

.page {
  width: 1124px;
}

.hat {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

h1 {
  font-size: 32px;
  font-weight: 700;
}

h2 {
  font-size: 24px;
}

.form-section {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  padding: 20px;
  border: 1px solid #b1b9bf;
  border-radius: 8px;
  margin-bottom: 20px;
}

.form-section > h2,
.upload-area > h2 {
  margin-top: 0;
}

input:not([type='radio']) {
  width: 342px;
  height: 39px;
  border-radius: 8px;
  padding: 12px 8px;
  border: none;
  background-color: #f3f7f9;
  font-family: 'Rubik';
  font-weight: 400;
}

 a {
  color: #ffcc00;
 }

.inputs {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 28px;
  width: 100%;
}

.question-files-field {
  display: flex;
  gap: 20px;
}

.question-field {
  display: flex;
  flex-direction: row;
  gap: 20px;
}

.form-section[name='files'] {
  max-height: 241px;
}

input[type='file'] {
  display: none;
}

input[type='checkbox'] {
  accent-color: #000;
  width: 18px;
  height: 18px;
  margin-bottom: 48px;
}

.invalid {
  border: 1px solid #e53935 !important;
}

.submit-button {
  width: 220px;
  border-radius: 20px;
  padding: 16px 8px;
  gap: 8px;
  background-color: #bdbdbd;
  border: none;
  font-size: 15px;
  color: #000;
  margin: 16px 0px;
}

.submit-button:disabled {
  cursor: not-allowed;
}

.terms input.invalid {
  outline: 1px solid #e53935;
}

textarea {
  width: 701px;
  height: 101px;
  border-radius: 8px;
  padding: 12px 8px;
  background-color: #f3f7f9;
  border: none;
  font-family: 'Rubik';
  font-weight: 400;
}

.ESIA-enter-button {
  width: 220px;
  height: 39px;
  padding: 12px 8px;
  background-color: #ffffff;
  color: #000000;
  border: 1px solid #ffcc00;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-family: 'Rubik';
  font-weight: 400;
}

.upload-area {
  max-width: 100%;
  display: flex;
  flex-direction: column;
}

.upload-icon {
  font-size: 15px;
  border-radius: 8px;
  padding: 12px 8px;
  background-color: #f3f7f9;
  gap: 8px;
  text-align: center;
}

.upload-hint {
  color: #4c5e6b;
  font-size: 15px;
  padding-top: 24px;
}

.footer-field {
  padding-bottom: 130px;
}

footer {
  width: 741px;
  font-size: 15px;
  line-height: 130%;
  color: #4c5e6b;
  padding: 12px 0px;
}

.radio-option {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.radio-option input[type='radio'] {
  -webkit-appearance: none;
  appearance: none;
  flex-shrink: 0;
  width: 18px;
  height: 18px;
  background-color: #ffffff;
  border: 2px solid #000;
  border-radius: 50%;
  display: inline-grid;
  place-content: center;
  margin-right: 8px;
}

input[type='radio']:checked {
  border-color: #ffcc00;
}

.radio-option input[type='radio']:checked::before {
  content: '';
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: #ffcc00;
}

.answer-field {
  width: 571px;
  display: flex;
  align-items: center;
  gap: 20px;
}

.file-list {
  padding-left: 20px;
  overflow-wrap: anywhere;
  font-size: 14px;
}
</style>
