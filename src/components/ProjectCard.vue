<template>
   <div class="col-12 col-md-6 col-lg-4 mb-4">
	   <div class="card h-100 p-3 border project-card">
			  <img :src="project.image" class="img-fluid" alt="...">
		  <div class="card-body text-center">
			  <h5 class="card-title mt-5">{{project.title}}</h5>
			  <p class="card-text">{{project.description}}</p>
			  <a :href="project.link" class="btn btn-secondary">More info...</a>
		  </div>
		</div>
   </div>
</template>

<script setup>
	defineProps({
		project: {
			type: Object,
			required:true
		}
	});
</script>