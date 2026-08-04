<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import {Notyf} from "notyf";
import "notyf/notyf.min.css";

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


onMounted(() => {
  window.grecaptcha.ready(() => {
    try {
      recaptchaWidgetId.value = window.grecaptcha.render(
        recaptchaContainer.value,
        {
          sitekey: RECAPTCHA_SITE_KEY,
          callback: (token) => {
            captchaToken.value = token;
          },
          "expired-callback": () => {
            captchaToken.value = "";
          }
        }
      );

      console.log("Rendered widget:", recaptchaWidgetId.value);

    } catch (error) {
      console.error("RECAPTCHA ERROR:", error);
    }
  });
});

onBeforeUnmount(() => {
	delete window.__onRecaptchaLoad;
})

function resetCaptcha() {
	captchaToken.value = "";

	if(window.grecaptcha && recaptchaWidgetId.value !== null) {
		window.grecaptcha.reset(recaptchaWidgetId.value);
	}
}

const submitForm = async () => {
  isLoading.value = true;

  try {
    const response = await fetch("https://api.web3forms.com/submit", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Accept: "application/json",
      },
      body: JSON.stringify({
        access_key: WEB3FORMS_ACCESS_KEY,
        name: name.value,
        email: email.value,
        message: message.value,
        subject: subject,
        // "g-recaptcha-response": captchaToken.value,
      }),
    });

    const result = await response.json();

    if (result.success) {
      notyf.success("Message Sent!");

      name.value = "";
      email.value = "";
      message.value = "";

      resetCaptcha();
    } else {
      notyf.error(result.message || "Failed to send message");
      console.error(result);
    }

  } catch (error) {
    console.error(error);
    notyf.error("Failed to send message");
  } finally {
    isLoading.value = false;
  }
};

</script>





<template>


<section class="container-fluid">
		<div class="row justify-content-stretch align-items-stretch" id="contact">
	
			<div class="col py-5 px-5" id="contact_form">
				<form class="mt-5 p-5" action="https://api.web3forms.com/submit" @submit.prevent="submitForm" method="POST">
					<h2 class="mt-5 mb-5">CONTACT US</h2>
					<p>Feel free to message me and let's get a website developed</p>
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

					<div class="mb-4" ref="recaptchaContainer"></div>


					<button type="submit" class="float-end mt-3 btn rounded-pill p-2" data-bs-toggle="" data-bs-target="">
				        {{ isLoading ? 'Sending...' : 'Submit'}}
				    </button>
				</form>
			</div>
		</div>
	</section>

</template>