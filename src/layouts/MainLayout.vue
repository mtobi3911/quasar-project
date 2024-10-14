<template>
  <div id="q-app" style="min-height: 100vh;">
    <div class="q-pa-md q-gutter-sm">
      <!-- <q-btn label="Написать письмо" color="secondary" @click="openNewEmail"></q-btn> -->
      

      <q-dialog v-model="alert" persistent>
        <q-card class="q-pa-md dialog-card" style="max-width: 500px;">
          <q-card-section>
            <div class="text-h6 text-center q-mb-md">{{ isEditingDraft ? 'Редактировать черновик' : 'Написать письмо' }}</div>
          </q-card-section>
          <q-card-section class="q-gutter-sm">
            
            <q-input
              class="input_email"
              v-model="email_address"
              label="Введите адрес"
              :error="emailError"
              :error-message="errorMessage"
              dense
              outlined
              @change="validateEmail"
            ></q-input>
            <q-input 
              v-model="text" 
              filled 
              type="textarea" 
              label="Введите текст письма"
              dense
              outlined
              class="q-mt-md"
            ></q-input>
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="Отмена" color="secondary"  @click="alert = false"></q-btn>
            <q-space></q-space>
            <q-btn flat label="Сохранить в черновик" color="orange" @click="saveToDrafts" :disabled="!isValidEmail"></q-btn>
            <q-btn flat label="Отправить" color="primary" @click="handleSubmit" :disabled="!isValidEmail"></q-btn>
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
    <!-- Перенесенная кнопка с header -->
    <q-btn class="btn_click_bar" label="Написать письмо" color="secondary" @click="openNewEmail"></q-btn>
    <div>
      <q-splitter v-model="splitterModel" class="box">
        <template v-slot:before>
          <q-tabs v-model="tab" vertical class="text-teal">
            <q-tab name="mails" icon="mail" label="Входящие"></q-tab>
            <q-tab name="alarms" icon="inbox" label="Отправленные"></q-tab>
            <q-tab name="movies" icon="label" label="Черновики"></q-tab>

            <q-btn
              class="night_theme"
              flat
              icon="brightness_6"
              :label="Dark.isActive ? 'Светлая тема' : 'Темная тема'"
              @click="toggleTheme"
              color="secondary"
            />
          </q-tabs>
        </template>

        <template v-slot:after>
          <q-tab-panels v-model="tab" animated swipeable vertical transition-prev="jump-up" transition-next="jump-up">
            
            <!-- Входящие -->
            <q-tab-panel class="mail_box" name="mails">
              <div class="text-h4 q-mb-md">Входящие</div>
              <div v-for="(mail, index) in incomingMails" :key="index" class="q-pa-sm q-mb-sm">
                <q-card class="mail-card">
                  <q-card-section>
                    <div><strong>От:</strong> {{ mail.sender }}</div>
                    <div><strong>Тема:</strong> {{ mail.subject }}</div>
                    <div><strong>Дата:</strong> {{ mail.date }}</div>
                    <p>{{ mail.body }}</p>
                  </q-card-section>
                  <q-btn flat color="negative" label="Удалить" @click="deleteMail(index, 'incoming')"></q-btn>
                </q-card>
              </div>
            </q-tab-panel>

            <!-- Отправленные -->
            <q-tab-panel name="alarms">
              <div class="text-h4 q-mb-md">Отправленные</div>
              <div v-for="(mail, index) in outboxMails" :key="index" class="mail-card">
                <p><strong>Адрес:</strong> {{ mail.email }}</p>
                <p><strong>Текст:</strong> {{ mail.text }}</p>
                <q-btn flat color="negative" label="Удалить" @click="deleteMail(index, 'outbox')"></q-btn>
              </div>
            </q-tab-panel>

            <!-- Черновики -->
            <q-tab-panel name="movies">
              <div class="text-h4 q-mb-md">Черновики</div>
              <div v-for="(draft, index) in draftMails" :key="index" class="mail-card" @click="editDraft(index)">
                <p><strong>Адрес:</strong> {{ draft.email }}</p>
                <p><strong>Текст:</strong> {{ draft.text }}</p>
                <q-btn flat color="primary" label="Редактировать" @click="editDraft(index)"></q-btn>
                <q-btn flat color="negative" label="Удалить" @click="deleteMail(index, 'draft')"></q-btn>
              </div>
            </q-tab-panel>
          </q-tab-panels>
        </template>
      </q-splitter>
    </div>
  </div>
</template>

<script setup>
import { Dark } from 'quasar'
import {onMounted, ref} from 'vue'

const alert = ref(false)
const tab = ref('mails')
const splitterModel = ref(20)

const email_address = ref('')
const text = ref('')

const emailError = ref(false)
const errorMessage = ref('')

const isValidEmail = ref(false)

const outboxMails = ref([]) // Массив для исходящих писем
const draftMails = ref([])   // Массив для черновиков

const isEditingDraft = ref(false)  // Флаг для определения редактирования черновика
let editingDraftIndex = ref(null)  // Индекс редактируемого черновика

// Массив с входящими письмами (заполненный вручную)
const incomingMails = ref([
  {
    sender: 'noreply-maps-timeline@google.com',
    subject: 'Тоби, вспомните, где Вы побывали в сентябре.',
    date: 'сб, 5 окт., 14:52',
    body: 'Тоби, вот ваша хронология за месяц Вы получили это письмо, поскольку в вашем аккаунте Google включена история местоположений. При этом создается хронология – ваша личная карта, на которой отмечаются посещенные места и маршруты поездок. Посмотреть, изменить или удалить информацию о местах, где вы побывали, можно в хронологии.'
  },
  {
    sender: 'help@accts.epicgames.com',
    subject: 'Ваш код двухфакторной аутентификации.',
    date: 'пн, 30 сент., 23:35',
    body: 'Здравствуйте, Тоби. Недавно вы попытались зайти в учётную запись с нового устройства, из нового браузера или местоположения. Используйте данный код, чтобы завершить операцию. Узнать подробности.Если это были не вы, то смените пароль. С благодарностью, Команда Epic Games'
  },
  {
    sender: 'messages@avito.ru',
    subject: 'Вам пришло новое сообщение.',
    date: 'пн, 30 сент., 17:02',
    body: 'Я пока уехал 5 октября буду дома'
  }
])

const toggleTheme = () => {
  Dark.set(!Dark.isActive)
  localStorage.setItem('isDarkTheme', String(Dark.isActive))
}

// Открыть окно для нового письма
const openNewEmail = () => {
  isEditingDraft.value = false
  email_adsress.value = ''
  text.value = ''
  alert.value = true
}

// Открыть и редактировать черновик
const editDraft = (index) => {
  isEditingDraft.value = true
  editingDraftIndex.value = index
  email_address.value = draftMails.value[index].email
  text.value = draftMails.value[index].text
  alert.value = true
}


// Функция для валидации email
const validateEmail = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (emailRegex.test(email_adsress.value)) {
    console.log('Valid email:', email_address.value)
    emailError.value = false
    errorMessage.value = ''
    isValidEmail.value = true
  } else {
    console.log('Invalid email:', email_address.value)
    emailError.value = true
    errorMessage.value = 'Введите корректный email'
    isValidEmail.value = false
  }
  console.log('isValidEmail:', isValidEmail.value)
}

// Функция для отправки письма
const handleSubmit = () => {
    if (!isValidEmail.value) {
    emailError.value = true
    errorMessage.value = 'Введите корректный email'
  } else {
    emailError.value = false
    errorMessage.value = ''

    // Если редактируется черновик, удаляем его
    if (isEditingDraft.value) {
      draftMails.value.splice(editingDraftIndex.value, 1)
    }

    outboxMails.value.push({
      email: email_address.value,
      text: text.value
    })

    email_address.value = ''
    text.value = ''
    alert.value = false
  }
}

// Функция для сохранения черновика
const saveToDrafts = () => {
  if (!isValidEmail.value) {
    emailError.value = true
    errorMessage.value = 'Введите корректный email для сохранения черновика'
  } else {
    emailError.value = false
    errorMessage.value = ''

      // Если редактируется черновик, обновляем его
      if (isEditingDraft.value) {
       draftMails.value[editingDraftIndex.value] = {
          email: email_address.value,
          text: text.value
        }
     } else {
        draftMails.value.push({
         email: email_address.value,
         text: text.value
       })
      }

    email_address.value = ''
    text.value = ''
    alert.value = false
  }
}

const deleteMail = (index, type) => {
  if (type === 'incoming') {
    incomingMails.value.splice(index, 1); // Удалить письмо из входящих
  } else if (type === 'outbox') {
    outboxMails.value.splice(index, 1); // Удалить письмо из отправленных
  } else if (type === 'draft') {
    draftMails.value.splice(index, 1); // Удалить черновик
  }
};

onMounted(() => {
  const savedTheme = localStorage.getItem('isDarkTheme')
  const booleanSavedTheme = savedTheme === 'true' 

  Dark.set(booleanSavedTheme)
})
</script>

<style scoped>
.dialog-card {
  max-width: 500px;
  width: 100%;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
.input_email {
  margin-bottom: 16px;

}
.btn_click_bar {
  margin-left: 110px;
  margin-bottom: 10px;
}
.night_theme {
  margin-top: 10px;
  margin-left: 100px;
}
.q-card-actions .q-btn {
  margin-left: 10px;
}
.text_mail {
  padding: 10px;
}
.mail-card {
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  margin-bottom: 16px;
  padding: 10px;
  cursor: pointer;
}
.q-btn {
  min-width: 100px;
}
</style>