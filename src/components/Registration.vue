<template>
  <div style="z-index: 999;">
    <h2>Registration</h2>
    <!-- Conditionally display the RegistrationForm based on showRegistrationForm -->
    <RegistrationForm @registration-success="handleRegistrationSuccess" v-if="showRegistrationForm" />

    <!-- Display a success message when registration is successful -->
    <div v-if="registrationSuccessful">
      <p>Registration successful!</p>
    </div>
  </div>
</template>

<script>
import RegistrationForm from '@/components/RegistrationForm.vue';

export default {
  components: {
    RegistrationForm,
  },
  data() {
    return {
      registrationSuccessful: false,
      showRegistrationForm: true, // Add this property
    };
  },
  methods: {
    handleRegistrationSuccess() {
      // Handle the event emitted by the RegistrationForm
      // Update the data property to close the form and show success message
      this.registrationSuccessful = true;
      
      // Close the registration form
      this.showRegistrationForm = false;

      // Use Vue Router to navigate to the chat page
      this.$router.push({ name: 'Chat' });
    },
  },
};
</script>
