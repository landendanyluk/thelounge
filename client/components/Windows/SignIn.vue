<template>
	<div id="sign-in" class="window" role="tabpanel" aria-label="Sign-in">
		<form class="container" method="post" action="" @submit="onSubmit">
			<img
				src="img/logo-vertical-transparent-bg.svg"
				class="logo"
				alt="Utherverse Support"
				width="256"
				height="170"
			/>
			<img
				src="img/logo-vertical-transparent-bg-inverted.svg"
				class="logo-inverted"
				alt="Utherverse Support"
				width="256"
				height="170"
			/>

			<label for="signin-username">Username</label>
			<input
				id="signin-username"
				ref="username"
				class="input"
				type="text"
				name="username"
				autocapitalize="none"
				autocorrect="off"
				autocomplete="username"
				:value="getStoredUser()"
				required
				autofocus
			/>

			<div class="password-container">
				<label for="signin-password">Password</label>
				<RevealPassword v-slot:default="slotProps">
					<input
						id="signin-password"
						ref="password"
						:type="slotProps.isVisible ? 'text' : 'password'"
						name="password"
						class="input"
						autocapitalize="none"
						autocorrect="off"
						autocomplete="current-password"
						required
					/>
				</RevealPassword>
			</div>

			<div v-if="errorShown" class="error">Authentication failed.</div>

			<button :disabled="inFlight" type="submit" class="btn">Sign in</button>
		</form>
	</div>
</template>

<script>
import storage from "../../js/localStorage";
import socket from "../../js/socket";
import RevealPassword from "../RevealPassword.vue";

export default {
	name: "SignIn",
	components: {
		RevealPassword,
	},
	data() {
		return {
			inFlight: false,
			errorShown: false,
		};
	},
	mounted() {
		socket.on("auth:failed", this.onAuthFailed);
	},
	beforeDestroy() {
		socket.off("auth:failed", this.onAuthFailed);
	},
	methods: {
		onAuthFailed() {
			this.inFlight = false;
			this.errorShown = true;
		},
		onSubmit(event) {
			event.preventDefault();

			this.inFlight = true;
			this.errorShown = false;

			const values = {
				user: this.$refs.username.value,
				password: this.$refs.password.value,
			};

			storage.set("user", values.user);

			socket.emit("auth:perform", values);
		},
		getStoredUser() {
			return storage.get("user");
		},
	},
};
</script>
