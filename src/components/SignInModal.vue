<template>
  <div class="fixed inset-0 flex items-center justify-center">
    <div class="bg-white w-full max-w-sm p-6 rounded-lg shadow-lg">
      <form @submit.prevent="signIn">
        <!-- Close button -->
        <button @click="closeModal" class="close-button">&times;</button>

        <!-- Email input -->
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="email">
            Email:
          </label>
          <input
            v-model="email"
            type="email"
            id="email"
            name="email"
            class="appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            required
          />
        </div>
        <!-- Password input -->
        <div class="mb-6">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="password">
            Password:
          </label>
          <input
            v-model="password"
            type="password"
            id="password"
            name="password"
            class="appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            required
          />
        </div>
        <!-- Error message -->
        <div v-if="errorMessage" class="text-red-500 text-sm mb-4">{{ errorMessage }}</div>
        <!-- Sign In button -->
        <div class="flex justify-between items-center">
          <button
            type="submit"
            class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-full focus:outline-none focus:shadow-outline"
          >
            Sign In
          </button>
          <!-- Close button -->
          <button @click="closeModal" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-full focus:outline-none focus:shadow-outline">
            Close
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
  setup() {
    const { signInWithEmailAndPassword } = useAuth();
    const email = ref('');
    const password = ref('');
    const errorMessage = ref('');

    const signIn = async () => {
      try {
        await signInWithEmailAndPassword(email.value, password.value);
        // If authentication is successful, you can navigate to the chat or home page here.
        // For example:
        // router.push('/chat');
      } catch (error) {
        // Handle authentication error and display an error message.
        errorMessage.value = 'Incorrect username/password';
      }
    };

    const closeModal = () => {
      // Emit an event to inform the parent component to close the modal
      // You can name the event anything you want, for example: 'close'
      // In the parent component, you should listen to this event and handle the closing of the modal
      // For example: <SignInModal @close="isModalOpen = false" />
      // `emit` function should be imported from 'vue'
      emit('close');
    };

    return { email, password, errorMessage, signIn, closeModal };
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
