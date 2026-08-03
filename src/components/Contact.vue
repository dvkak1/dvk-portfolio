<script setup>
import { ref, onMounted, onBeforeMount } from "vue";
import {Notyf} from "notyf";

const notyf = new Notyf();

const name = ref("");
const email = ref("");
const message = ref("");

const isLoading = ref(false);

const captchaToken = ref("");
const recaptchaWidgetId = ref(null);
const recaptchaContainer = ref(null);

const WEB3FORMS_ACCESS_KEY = import.meta.env.VITE_WEB3FORMS_ACCESS_KEY
const WEB3FORMS_ENDPOINT = 'https://api.web3forms.com/submit'
const RECAPTCHA_SITE_KEY = import.meta.env.VITE_RECAPTCHA_SITE_KEY
const subject = "A user sent a message from your WebPortfolio"; 


function loadRecaptchaScript() {
	return new Promise((resolve) => {
		if(window.grecaptcha && window.grecaptcha.render) {
			resolve();
			return;
		}
		window.__onRecaptchaLoad = () => resolve();
		const script = document.createElement("script");
		script.src = "https://www.google.com/recaptcha/api.js?onload=__onRecaptchaLoad&render=explicit";
		script.async = true;
		script.defer = true;
		document.head.appendChild(script);
	})
}

onMounted(async() => {
	if(!RECAPTCHA_SITE_KEY) return;
	await loadRecaptchaScript();
	recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
		sitekey: RECAPTCHA_SITE_KEY,
		callback: (token) => {captchaToken.value = token; },
		"expired callback": () => {captchaToken.value = ""; },
	});
});

onBeforeMount(() => {
	delete window.__onRecaptchaLoad;
})

function resetCaptcha() {
	captchaToken.value = "";
	if(window.grecaptcha && recaptchaWidgetId.value !== null) {
		window.grecaptcha.reset(recaptcha.WidgetId.value);
	}
}

const submitForm = async () => {
		try{
			const response = await fetch(WEB3FORMS_ENDPOINT, {
				method: "POST",
				headers: {
					"Content-Type" : "application/json",
					Accept: "application/json"
				},
				body: {
					access_key: WEB3FORMS_ACCESS_KEY,
					subject: subject,
					name: name.value,
					email: email.value,
					message: message.value
				}
			})

			const result = await response.json();

			if(result.success){
				isLoading.value = false
				notyf.success("Message Sent! ")
			}
		}
		catch(error){
			console.log(error);
			isLoading.value = false;
			notyf.error("Failed to send message");
		}
}

</script>





<template>


<section class="container-fluid">
		<div class="row justify-content-stretch align-items-stretch" id="contact">
	
			<div class="col py-5 px-5" id="contact_form">
				<form class="mt-5 p-5" action="https://api.web3forms.com/submit" method="POST">
					<h2 class="mt-5 mb-5">CONTACT US</h2>
					<p>Feel free to message me and let's get a website developed</p>
					<input type="hidden" name="access_key" value="b2cbca3e-34f9-4dd-8f50-921abbf53db9">
					<div class="mt-5 mb-4">
					    <label for="name" class="form-label">Name</label>
					    <input type="text" class="form-control" id="name" v-model="name" required>
				  	</div>
				  	<div class="mb-4">
					    <label for="email" class="form-label">Email</label>
					    <input type="email" class="form-control" id="email" v-model="email" required>
					</div>
					<div class="mb-4">
					    <label for="message" class="form-label">Message</label>
					    <textarea class="form-control" id="message" rows="7" v-model="message" required></textarea>
					</div>

					<div class="mb-4" ref="recaptchaContainer" v-show="RECAPTCHA_SITE_KEY"></div>


					<button type="button" class="float-end mt-3 btn rounded-pill p-2" data-bs-toggle="" data-bs-target="">
				        {{ isLoading ? 'Sending...' : 'Submit'}}
				    </button>
				</form>
			</div>
		</div>
	</section>

</template>