<template>
  <div class="container">
    <!-- Left sidebar for Exam Info -->
    <div class="sidebar" v-if="isLogin">
      <h2>Informacije o ispitu</h2>

      <!-- Display the form only if the user is filipslavic45@gmail.com || msvarcmajer@gmail.com -->
      <form v-if="isFilipsExamInfo && !examInfo" @submit.prevent="postExamInfo">
        <!-- Add form fields for Naziv kolegija, Vrijeme ispita, Prostorija here -->
        <!-- For example: -->
        <input v-model="nazivKolegija" placeholder="Naziv kolegija" required />
        <input v-model="vrijemeIspita" placeholder="Vrijeme ispita" required />
        <input v-model="prostorija" placeholder="Prostorija" required />
        <button type="submit">Post</button>
      </form>

      <!-- Display the posted exam info -->
      <div v-if="examInfo">{{ examInfo }}</div>

      <!-- Display edit form for exam info -->
      <form v-if="isFilipsExamInfo && examInfo" @submit.prevent="editExamInfo">
        <!-- Add form fields for Naziv kolegija, Vrijeme ispita, Prostorija here -->
        <!-- For example: -->
        <input v-model="editedNazivKolegija" placeholder="Naziv kolegija" required />
        <input v-model="editedVrijemeIspita" placeholder="Vrijeme ispita" required />
        <input v-model="editedProstorija" placeholder="Prostorija" required />
        <button type="submit">Edit</button>
      </form>
    </div>

    <!-- Left sidebar for Uploaded Files -->
    <div class="sidebar uploaded-files" v-if="isLogin">
      <h2>Uploadani dokumenti</h2>
      <!-- Add your Uploaded Files content here -->
      <div class="uploaded-file-link" v-for="(file, index) in uploadedFiles" :key="index">
        <a :href="file.url" target="_blank" rel="noopener noreferrer">{{ file.name }}</a>
      </div>
    </div>

    <!-- Chat area -->
    <div class="chat">
      <div class="container-sm mt-20">
        <div class="mx-5">
          <!-- Display text messages and uploaded files as download links -->
          <Message
            v-for="{ id, text, userPhotoURL, userName, userId, fileURL, fileName } in messages"
            :key="id"
            :name="userName"
            :photo-url="userPhotoURL"
            :sender="userId === user?.uid"
          >
            <!-- Display text messages -->
            <div v-if="text">{{ text }}</div>

            <!-- Display uploaded files as download links -->
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
import { ref, watch, nextTick } from 'vue';
import { useAuth, useChat, storage } from '@/firebase';

import SendIcon from './SendIcon.vue';
import Message from './Message.vue';
import firebase from 'firebase/app'; // Import firebase at the top
import 'firebase/firestore'; // Import Firestore if not already imported

export default {
  components: { Message, SendIcon },
  setup() {
    const { user, isLogin } = useAuth();
    const { messages, sendMessage } = useChat();
    const examInfo = ref('');
    const nazivKolegija = ref('');
    const vrijemeIspita = ref('');
    const prostorija = ref('');
    const editedNazivKolegija = ref('');
    const editedVrijemeIspita = ref('');
    const editedProstorija = ref('');
    const isFilipsExamInfo = ref(false);

    // Reference to your Firebase collection or reference
    const examInfoRef = firebase.firestore().collection('examInfo');

    const postExamInfo = async () => {
      examInfo.value = `Naziv kolegija: ${nazivKolegija.value}, Vrijeme ispita: ${vrijemeIspita.value}, Prostorija: ${prostorija.value}`;

      // Create a document with the exam info data
      const docData = {
        nazivKolegija: nazivKolegija.value,
        vrijemeIspita: vrijemeIspita.value,
        prostorija: prostorija.value,
      };

      try {
        // Add the exam info document to Firestore
        await examInfoRef.add(docData);
        isFilipsExamInfo.value = true;
      } catch (error) {
        console.error('Error posting exam info:', error);
      }
    };

    const editExamInfo = async () => {
      examInfo.value = `Naziv kolegija: ${editedNazivKolegija.value}, Vrijeme ispita: ${editedVrijemeIspita.value}, Prostorija: ${editedProstorija.value}`;

      // Update the exam info document in Firestore
      const docData = {
        nazivKolegija: editedNazivKolegija.value,
        vrijemeIspita: editedVrijemeIspita.value,
        prostorija: editedProstorija.value,
      };

      try {
        await examInfoRef.doc('examInfo').update(docData);
      } catch (error) {
        console.error('Error updating exam info:', error);
      }
    };

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

    // Load uploaded files from localStorage on component mount
    const loadUploadedFiles = () => {
      const storedFiles = localStorage.getItem('uploadedFiles');
      if (storedFiles) {
        uploadedFiles.value = JSON.parse(storedFiles);
      }
    };

    loadUploadedFiles(); // Call the function when the component mounts

    const saveUploadedFiles = () => {
      // Save uploaded files to localStorage whenever it changes
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
          sendMessage(fileName); // Just send the file name without the `[File](...)` part
          file.value = null;

          // Add the uploaded file to the list
          uploadedFiles.value.push({ name: fileName, url: fileURL });

          // Save the updated list to localStorage
          saveUploadedFiles();
        } catch (error) {
          console.error('Error uploading file:', error);
        }
      } else if (message.value.trim() !== '') {
        sendMessage(message.value);
      }

      message.value = '';
    };

    return {
      user,
      isLogin,
      messages,
      bottom,
      message,
      send,
      handleFileUpload,
      uploadedFiles,
      examInfo,
      postExamInfo,
      nazivKolegija,
      vrijemeIspita,
      prostorija,
      editedNazivKolegija,
      editedVrijemeIspita,
      editedProstorija,
      isFilipsExamInfo,
      editExamInfo,
    };
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

/* Left sidebar styles */
.sidebar {
  width: 30%;
  min-height: 300px;
  position: fixed;
  top: 70px;
  left: 0;
  background-color: rgba(173, 168, 168, 0.8);
  padding: 20px;
  border-radius: 10px; /* Rounded corners */
  margin-bottom: 20px; /* Space between Exam Info and Uploaded Files */
  z-index: 5;
}

.uploaded-files {
  margin-top: 330px; /* Space between Exam Info and Uploaded Files */
  background-color: rgba(63, 60, 60, 0.8);
  border-radius: 10px; /* Rounded corners */
  z-index: 5;
}

.uploaded-file-link {
  padding: 10px;
}

/* Chat area styles */
.chat {
  margin-left: 30%; /* Adjust for sidebar width */
  padding: 20px;
  display: flex; /* Use flexbox for layout */
  flex-direction: column; /* Stack elements vertically */
}

/* Style the message input and send button container */
.message-input-container {
  display: flex;
  align-items: center; /* Align items vertically in the container */
  gap: 10px; /* Add some spacing between elements */
}

/* Style the file input */
.file-input {
  flex: 1; /* Allow the input to grow and fill available space */
}

/* Style the send button */
.send-button {
  /* Add your button styles here */
}

/* Additional styles for the chat container */
.container-sm {
  max-width: 600px;
  margin: 0 auto;
}

/* Style the file download link */
.download-link a {
  color: blue; /* Blue color */
  text-decoration: underline; /* Underline the link text */
}
</style>
