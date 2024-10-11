<template>
  <div id="q-app" style="min-height: 100vh;">
    <div class="q-pa-md q-gutter-sm">
      <q-btn label="Написать письмо" color="secondary" @click="alert = true"></q-btn>

      <q-dialog v-model="alert" persistent>
        <q-card class="q-pa-md dialog-card" style="max-width: 500px;">
          <q-card-section>
            <div class="text-h6 text-center q-mb-md">Написать письмо</div>
          </q-card-section>
          <q-card-section class="q-gutter-sm">
            <q-input
              class="input_email"
              v-model="email_adres"
              label="Введите адрес"
              :error="emailError"
              :error-message="errorMessage"
              dense
              outlined
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
            <q-btn flat label="Отмена" color="secondary"  @click="saveToDrafts"></q-btn>
            <q-btn flat label="Отправить" color="primary" @click="handleSubmit"></q-btn>
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>

    <div>
      <q-splitter v-model="splitterModel" class="box">
        <template v-slot:before>
          <q-tabs v-model="tab" vertical class="text-teal">
            <q-tab name="mails" icon="mail" label="Входящие"></q-tab>
            <q-tab name="alarms" icon="inbox" label="Исходящие"></q-tab>
            <q-tab name="movies" icon="label" label="Черновики"></q-tab>
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
                </q-card>
              </div>
            </q-tab-panel>

            <!-- Исходящие -->
            <q-tab-panel name="alarms">
              <div class="text-h4 q-mb-md">Исходящие</div>
              <div v-for="(mail, index) in outboxMails" :key="index" class="mail-card">
                <p><strong>Адрес:</strong> {{ mail.email }}</p>
                <p><strong>Текст:</strong> {{ mail.text }}</p>
              </div>
            </q-tab-panel>

            <!-- Черновики -->
            <q-tab-panel name="movies">
              <div class="text-h4 q-mb-md">Черновики</div>
              <div v-for="(draft, index) in draftMails" :key="index" class="mail-card">
                <p><strong>Адрес:</strong> {{ draft.email }}</p>
                <p><strong>Текст:</strong> {{ draft.text }}</p>
              </div>
            </q-tab-panel>
          </q-tab-panels>
        </template>
      </q-splitter>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const alert = ref(false)
const tab = ref('mails')
const splitterModel = ref(20)

const email_adres = ref('')
const text = ref('')

const emailError = ref(false)
const errorMessage = ref('')

const outboxMails = ref([]) // Массив для исходящих писем
const draftMails = ref([])   // Массив для черновиков

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




// Функция для отправки данных с проверкой
// const handleSubmit = () => {
//   if (email_adres.value.trim() === '') {
//     emailError.value = true
//     errorMessage.value = 'Введите значение'
//   } else {
//     emailError.value = false
//     errorMessage.value = ''

const handleSubmit = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  if (email_adres.value.trim() === '') {
    emailError.value = true;
    errorMessage.value = 'Введите значение';
  } else if (!emailRegex.test(email_adres.value)) {
    emailError.value = true;
    errorMessage.value = 'Введите корректный email';
  } else {
    emailError.value = false;
    errorMessage.value = '';
    


    outboxMails.value.push({
      email: email_adres.value,
      text: text.value
    });

  
    email_adres.value = '';
    text.value = '';

    alert.value = false;
  }
};
// Функция для сохранения письма в черновики
const saveToDrafts = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  // Если введен текст или email
  if (email_adres.value.trim() !== '' || text.value.trim() !== '') {
    // Если email есть, проверяем его корректность
    if (email_adres.value.trim() !== '' && !emailRegex.test(email_adres.value)) {
      emailError.value = true;
      errorMessage.value = 'Введите корректный email для сохранения черновика';
    } else {

       emailError.value = false;
       errorMessage.value = '';

        draftMails.value.push({
         email: email_adres.value,
          text: text.value
       });

     email_adres.value = ''
     text.value = ''

     alert.value = false;
    }
  }
};
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
.text_mail {
  padding: 10px;
}
.mail-card {
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  margin-bottom: 16px;
  padding: 10px;
}
.q-btn {
  min-width: 100px;
}
</style>