<script setup lang="ts">
import { ref } from "vue";
import { z } from "zod";
import { useRecoveryPasswordMutation } from "@/services/auth.service";
import MyButton from "@/components/UI/MyButton.vue";
// import { useRouter } from "vue-router";

// const router = useRouter();

const validationSchema = z.object({
	email: z.string().email(),
});

const validationErrors = ref<{
	email?: string;
}>({});

const { mutate: recovery_password, isPending, error } = useRecoveryPasswordMutation();

const submitForm = async (event: Event) => {
	const rawData = Object.fromEntries(
		new FormData(event.target as HTMLFormElement)
	);

	const result = validationSchema.safeParse(rawData);
	if (!result.success) {
		const error = result.error;
		validationErrors.value.email = error.issues.find(
			(issue) => issue.path[0] === "email"
		)?.message;
		return;
	}
	validationErrors.value = {};

    // recovery_password(result.data);
	
	recovery_password(result.data, {
		onSuccess: () => {
			alert("A password recovery link has been sent to your e-mail address");
		}
	});
};
</script>

<template>
	<form @submit.prevent="submitForm">
		<fieldset>
			<label for="email">Email:</label>
			<input type="email" id="email" name="email" />
			<span v-if="validationErrors.email">{{ validationErrors.email }}</span>
		</fieldset>

		<span v-if="error">{{ error }}</span>

		<my-button type="submit" :disabled="isPending">
			{{ isPending ? "Fetching..." : "Reset password" }}
		</my-button>
	</form>
</template>

<style scoped>
form {
	max-width: 300px;
	margin: auto;
	padding: 20px;
	border: 1px solid #ccc;
	border-radius: 5px;
}

label {
	display: block;
	margin-bottom: 5px;
}

input {
	width: 100%;
	padding: 8px;
	margin-bottom: 10px;
	box-sizing: border-box;
}

fieldset {
	border: none;
}
</style>
