<template>
  <div class="fixed inset-0 flex items-center justify-center">
    <div class="bg-white w-full max-w-sm p-6 rounded-lg shadow-lg">
      <form @submit.prevent="registerUser">
        <!-- Close button -->
        <button @click="closeModal" class="close-button">&times;</button>

        <!-- Name input -->
        <div class="mb-4">
          <label for="name" class="block text-gray-700 text-sm font-bold mb-2">Name:</label>
          <input v-model="name" type="text" id="name" placeholder="Your Name" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required />
        </div>
        <!-- Family Name input -->
        <div class="mb-4">
          <label for="familyName" class="block text-gray-700 text-sm font-bold mb-2">Family Name:</label>
          <input v-model="familyName" type="text" id="familyName" placeholder="Your Family Name" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required />
        </div>
        <!-- Email input -->
        <div class="mb-4">
          <label for="email" class="block text-gray-700 text-sm font-bold mb-2">Email:</label>
          <input v-model="email" type="email" id="email" placeholder="Your Email" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required />
        </div>
        <!-- Username input -->
        <div class="mb-4">
          <label for="username" class="block text-gray-700 text-sm font-bold mb-2">Username:</label>
          <input v-model="username" type="text" id="username" placeholder="Your Username" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required />
        </div>
        <!-- Password input -->
        <div class="mb-4">
          <label for="password" class="block text-gray-700 text-sm font-bold mb-2">Password:</label>
          <input v-model="password" type="password" id="password" placeholder="Your Password" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required />
        </div>
        <!-- Registration button -->
        <div class="flex items-center justify-between">
          <button type="submit" class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
            Register
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { ref, defineComponent, emit } from 'vue'; // Import emit function from 'vue'
import { useAuth } from '@/firebase';

export default defineComponent({
  data() {
    return {
      name: '',
      familyName: '',
      email: '',
      username: '',
      password: '',
    };
  },
  methods: {
    async registerUser() {
      const { name, familyName, email, username, password } = this;

      try {
        // Call the register function from useAuth to register the user
        await useAuth().registerUser(name, familyName, email, username, password);

        // Registration successful, emit events to inform the parent component
        // to close the modal and handle registration success
        emit('close-registration-form');
        emit('registration-success');
        
      } catch (error) {
        console.error('Error registering user:', error);
        // Handle registration error (display error message, etc.)
      }
    },
    closeModal() {
      // Emit an event to inform the parent component to close the modal
      // You can name the event anything you want, for example: 'close-registration-form'
      // In the parent component, you should listen to this event and handle the closing of the modal
      emit('close-registration-form');
    },
  },
});
</script>

<style scoped>
.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
}
</style>
