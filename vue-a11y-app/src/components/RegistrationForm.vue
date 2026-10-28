<script setup>
import { ref, reactive } from 'vue';

defineProps({
  isAccessible: {
    type: Boolean,
    default: false,
  },
});

const formData = reactive({ name: '', email: '', password: '' });
const errors = reactive({ name: '', email: '', password: '' });
const submitted = ref(false);

const validate = () => {
  errors.name = !formData.name.trim() ? 'Full Name is required.' : '';
  errors.email = !formData.email.includes('@') ? 'Please provide a valid email address.' : '';
  errors.password = formData.password.length < 8 ? 'Password must be at least 8 characters.' : '';

  return !errors.name && !errors.email && !errors.password;
};

const handleSubmit = () => {
  if (validate()) {
    submitted.value = true;
  }
};
</script>

<template>
  <div class="component-demo">
    <p class="demo-description">
      Tests form control labeling, error association (`aria-describedby`), `aria-invalid`, required field announcements, and autocomplete semantics.
    </p>

    <div v-if="submitted" class="alert alert-success" :role="isAccessible ? 'status' : undefined">
      Registration successful! Form passed all checks.
    </div>

    <form v-else class="form-stack" noValidate @submit.prevent="handleSubmit">
      <!-- 1. Name Field -->
      <div class="form-group">
        <template v-if="isAccessible">
          <label for="vue-user-name" class="form-label">
            Full Name <span aria-hidden="true" class="required-star">*</span>
          </label>
          <input
            id="vue-user-name"
            v-model="formData.name"
            type="text"
            name="name"
            autocomplete="name"
            class="form-input"
            :class="{ 'input-error': errors.name }"
            aria-required="true"
            :aria-invalid="errors.name ? 'true' : 'false'"
            :aria-describedby="errors.name ? 'vue-name-error' : undefined"
          />
          <span v-if="errors.name" id="vue-name-error" class="form-error-text" role="alert">
            {{ errors.name }}
          </span>
        </template>

        <template v-else>
          <div class="form-label-fake">Full Name *</div>
          <input
            v-model="formData.name"
            type="text"
            placeholder="Enter full name"
            class="form-input"
          />
          <div v-if="errors.name" style="color: red; font-size: 13px; margin-top: 4px;">
            {{ errors.name }}
          </div>
        </template>
      </div>

      <!-- 2. Email Field -->
      <div class="form-group">
        <template v-if="isAccessible">
          <label for="vue-user-email" class="form-label">
            Email Address <span aria-hidden="true" class="required-star">*</span>
          </label>
          <input
            id="vue-user-email"
            v-model="formData.email"
            type="email"
            name="email"
            autocomplete="email"
            class="form-input"
            :class="{ 'input-error': errors.email }"
            aria-required="true"
            :aria-invalid="errors.email ? 'true' : 'false'"
            :aria-describedby="errors.email ? 'vue-email-error vue-email-hint' : 'vue-email-hint'"
          />
          <span id="vue-email-hint" class="form-hint-text">We will never share your email address.</span>
          <span v-if="errors.email" id="vue-email-error" class="form-error-text" role="alert">
            {{ errors.email }}
          </span>
        </template>

        <template v-else>
          <input
            v-model="formData.email"
            type="text"
            placeholder="Email Address (required)"
            class="form-input"
          />
          <div style="color: #888; font-size: 12px; margin-top: 4px;">We will never share your email address.</div>
          <div v-if="errors.email" style="color: red; font-size: 13px; margin-top: 4px;">
            {{ errors.email }}
          </div>
        </template>
      </div>

      <!-- 3. Password Field -->
      <div class="form-group">
        <template v-if="isAccessible">
          <label for="vue-user-password" class="form-label">
            Create Password <span aria-hidden="true" class="required-star">*</span>
          </label>
          <input
            id="vue-user-password"
            v-model="formData.password"
            type="password"
            name="password"
            autocomplete="new-password"
            class="form-input"
            :class="{ 'input-error': errors.password }"
            aria-required="true"
            :aria-invalid="errors.password ? 'true' : 'false'"
            :aria-describedby="errors.password ? 'vue-pwd-error vue-pwd-hint' : 'vue-pwd-hint'"
          />
          <span id="vue-pwd-hint" class="form-hint-text">Must be at least 8 characters long.</span>
          <span v-if="errors.password" id="vue-pwd-error" class="form-error-text" role="alert">
            {{ errors.password }}
          </span>
        </template>

        <template v-else>
          <div class="form-label-fake">Create Password *</div>
          <input
            v-model="formData.password"
            type="password"
            placeholder="••••••••"
            class="form-input"
          />
          <div v-if="errors.password" style="color: red; font-size: 13px; margin-top: 4px;">
            {{ errors.password }}
          </div>
        </template>
      </div>

      <div style="margin-top: 12px;">
        <button type="submit" class="btn btn-primary">Register Account</button>
      </div>
    </form>
  </div>
</template>
