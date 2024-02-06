<template>
  <div class="container">
    <div class="sidebar">
      <h2>Informacije o ispitu</h2>

      <!-- Display the form only if the user is filipslavic45@gmail.com -->
      <form v-if="isFilip" @submit.prevent="postExamInfo">
        <input v-model="nazivKolegija" placeholder="Naziv kolegija" required />
        <input v-model="vrijemeIspita" placeholder="Vrijeme ispita" required />
        <input v-model="prostorija" placeholder="Prostorija" required />
        <button type="submit">Post</button>
      </form>

      <!-- Display the posted exam info -->
      <div v-if="examInfo">{{ examInfo }}</div>
    </div>

    <div class="sidebar uploaded-files">
      <h2>Uploadani dokumenti</h2>
      <div class="uploaded-file-link" v-for="(file, index) in uploadedFiles" :key="index">
        <a :href="file.url" target="_blank" rel="noopener noreferrer">{{ file.name }}</a>
      </div>
    </div>

    <div class="chat">
      <div class="container-sm mt-20">
        <div class="mx-5">
          <Message
            v-for="{ id, text, userPhotoURL, userName, userId, fileURL, fileName } in messages"
            :key="id"
            :name="userName"
            :photo-url="userPhotoURL"
            :sender="userId === user?.uid"
          >
            <div v-if="text">{{ text }}</div>
            <div v-if="fileURL" class="download-link">
              <a :href="fileURL" target="_blank" rel="noopener noreferrer">{{ fileName }}</a>
            </div>
          </Message>
        </div>
      </div>

      <div ref="bottom" class="mt-20" />

      <div class="bottom">
        <div class="container-sm">
          <form v-if="isLogin" @submit.prevent="send">
            <input v-model="message" placeholder="Message" required />
            <input type="file" @change="handleFileUpload" accept="application/pdf,image/*" />
            <button type="submit">
              <SendIcon />
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch, nextTick, computed } from 'vue';
import { useAuth, useChat, storage } from '@/firebase';
import SendIcon from './SendIcon.vue';
import Message from './Message.vue';
import firebase from 'firebase/app';
import 'firebase/firestore';

export default {
  components: { Message, SendIcon },
  setup() {
    const { user, isLogin } = useAuth();
    const { messages, sendMessage } = useChat();
    const examInfo = ref('');
    const nazivKolegija = ref('');
    const vrijemeIspita = ref('');
    const prostorija = ref('');

    const examInfoRef = firebase.firestore().collection('examInfo');

    const postExamInfo = async () => {
      examInfo.value = `Naziv kolegija: ${nazivKolegija.value}, Vrijeme ispita: ${vrijemeIspita.value}, Prostorija: ${prostorija.value}`;

      const docData = {
        nazivKolegija: nazivKolegija.value,
        vrijemeIspita: vrijemeIspita.value,
        prostorija: prostorija.value,
      };

      try {
        await examInfoRef.add(docData);
      } catch (error) {
        console.error('Error posting exam info:', error);
      }
    };

    const isFilip = computed(() => user.value?.email === 'filipslavic45@gmail.com');

    watch(
      () => examInfoRef,
      async () => {
        try {
          const snapshot = await examInfoRef.get();
          examInfo.value = snapshot.docs.map(doc => {
            const data = doc.data();
            return `Naziv kolegija: ${data.nazivKolegija}, Vrijeme ispita: ${data.vrijemeIspita}, Prostorija: ${data.prostorija}`;
          }).join('\n');
        } catch (error) {
          console.error('Error fetching exam info:', error);
        }
      },
      { immediate: true }
    );

    const bottom = ref(null);
    watch(
      messages,
      () => {
        nextTick(() => {
          bottom.value?.scrollIntoView({ behavior: 'smooth' });
        });
      },
      { deep: true }
    );

    const message = ref('');
    const file = ref(null);
    const uploadedFiles = ref([]);

    const loadUploadedFiles = () => {
      const storedFiles = localStorage.getItem('uploadedFiles');
      if (storedFiles) {
        uploadedFiles.value = JSON.parse(storedFiles);
      }
    };

    loadUploadedFiles();

    const saveUploadedFiles = () => {
      localStorage.setItem('uploadedFiles', JSON.stringify(uploadedFiles.value));
    };

    const handleFileUpload = (event) => {
      const uploadedFile = event.target.files[0];
      if (uploadedFile) {
        file.value = uploadedFile;
      }
    };

    const send = async () => {
      if (!isLogin.value) return;

      if (file.value) {
        const storageRef = storage.ref();
        const fileRef = storageRef.child(`uploads/${file.value.name}`);

        try {
          await fileRef.put(file.value);
          const fileURL = await fileRef.getDownloadURL();
          const fileName = file.value.name;
          sendMessage(fileName);
          file.value = null;

          uploadedFiles.value.push({ name: fileName, url: fileURL });
          saveUploadedFiles();
        } catch (error) {
          console.error('Error uploading file:', error);
        }
      } else if (message.value.trim() !== '') {
        sendMessage(message.value);
      }

      message.value = '';
    };

    return { isFilip, user, isLogin, messages, bottom, message, send, handleFileUpload, uploadedFiles, examInfo, postExamInfo, nazivKolegija, vrijemeIspita, prostorija };
  },
};
</script>

<style scoped>
/* Global styles */
body {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

.container {
  display: flex;
}

.sidebar {
  width: 30%;
  min-height: 300px;
  position: fixed;
  top: 70px;
  left: 0;
  background-color: rgba(173, 168, 168, 0.8);
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  z-index: 5;
}

.uploaded-files {
  margin-top: 330px;
  background-color: rgba(63, 60, 60, 0.8);
  border-radius: 10px;
  z-index: 5;
}

.uploaded-file-link {
  padding: 10px;
}

.chat {
  margin-left: 30%;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.container-sm {
  max-width: 600px;
  margin: 0 auto;
}

.download-link a {
  color: blue;
  text-decoration: underline;
}
</style>
