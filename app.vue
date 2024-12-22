<template>
  <div class="wrapper">
    <h2>Submit a Complaint</h2>

    <!-- Form to add a new complaint -->
    <div class="complain-form">
      <label for="complain-title">Title</label>
      <input v-model.lazy="title" id="complain-title" type="text" placeholder="Title" aria-label="Title" />

      <label for="complain-body">Complaint</label>
      <textarea v-model.lazy="body" id="complain-body" placeholder="Enter your complaint" aria-label="Complaint"></textarea>
      <button @click="saveComplain" :disabled="isSaving" class="btn-submit">
        {{ isSaving ? "Saving..." : "Submit Complaint" }}
      </button>
      <!-- Place Simple text loader when saving -->
   

      <!-- Show Error message if any error is occured-->
    
      <p v-if="errorMessage" class="error-message" aria-live="assertive">{{ errorMessage }}</p>

    </div>

    <h2>Complaints List</h2>

    <!-- Simple text loader when loading -->
    <div v-if="isLoading">Loading...</div>
    <!-- List of complaints -->
    <template v-else-if="complains.length">
      <div
        v-for="complain in complains"
        :key="complain.Id"
        class="complain-item"
      >
        <h3>{{ complain.Title }}</h3>
        <p>{{ complain.Body }}</p>
        
        <span class="item-span">{{ new Date(complain.CreatedAt).toLocaleString() }}</span>
      </div>
    </template>
    <div v-else>
      <p>No complaints available.</p>
    </div>
  </div>
</template>

<script setup>
// import {debounce} from 'lodash'
const baseUrl = "https://sugarytestapi.azurewebsites.net/";
const listPath = "TestApi/GetComplains";
const savePath = "TestApi/SaveComplain";

const complains = ref([]);
const title = ref("");
const body = ref("");
const isSaving = ref(false);
const errorMessage = ref(""); 

const isLoading = ref(false);



// Utility function to fetch data with a timeout mechanism.
// This prevents the application from hanging indefinitely if the server is unresponsive.
const fetchWithTimeout = async (url, options, timeout = 5000) => {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, { ...options, signal: controller.signal });
    clearTimeout(id); // Clear timeout if fetch succeeds
    return response;
  } catch (error) {
    if (error.name === "AbortError") {
      throw new Error("Request timed out");
    }
    throw error; // Re-throw other errors
  }
};
// Fetch complaints from the API
const fetchComplains = async () => {
  try {
    isLoading.value = true;
    const response = await fetchWithTimeout(`${baseUrl}${listPath}`, {}, 5000); // 5-second timeout
    if (!response.ok) throw new Error("Failed to load complaints.");
    console.log(response, "RESPONSE")
    const data = await response.json();
    console.log(data[0].CreatedAt, "get");
    complains.value = Array.isArray(data) ? data : [];
  } catch (error) {
    console.error(error);
    errorMessage.value =
      error.message === "Request timed out"
        ? "The request timed out. Please try again later."
        : "Failed to load complaints. Please try again.";
    errorMessage.value = "Failed to load complaints. Please try again.";
  } finally {
    isLoading.value = false;
  }
};

// Save a new complaint
const saveComplain = async () => {
  if (!title.value.trim() || !body.value.trim()) {
    errorMessage.value = "Title and Body are required!";
    return;
  }
  try {
    isSaving.value = true;
    errorMessage.value = ""; // Clear previous error
    const response = await fetchWithTimeout(`${baseUrl}${savePath}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        Title: title.value,
        Body: body.value,
      }),
    },15000); // Set a 15-sec timeout
    const data = await response.json();
    console.log(data ,'dataaa');
    if (!data.Success) throw new Error("Failed to save complaint.");
   
    // update list of complaints on success
    // if (data && data.Success) {
      const newComplaint = {
        Id: data.Id || Date.now(), // Temporary unique ID
      Title: title.value,
      Body: body.value,
    };

    complains.value.push(newComplaint )
   
   
    // }
     // Reset the form
 title.value = "";
 body.value = "";
 //  nextTick(() => {
 //   document.querySelector("input[aria-label='Title']").focus();
 // })
 document.querySelector("input[aria-label='Title']").focus();
  
  } catch (e) {
    console.log(e);
    errorMessage.value = e.message || "An error occurred while saving the complaint.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  // call fetchComplains when component is mounted
  fetchComplains()
});


</script>

<style>
/* body {background-color: #e7e7e7;} */
.wrapper {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
 box-shadow:  rgba(0, 0, 0, 0.05) 0px 6px 24px 0px, rgba(0, 0, 0, 0.08) 0px 0px 0px 1px;
  /* background-color: red; */
}
h2{
  border: 2px;
  border-bottom: 1px;
  border-color: grey;
  font-size: 30px;
}
.complain-form {
  margin-bottom: 20px;
  /* background-color: red; */
  /* padding: 20px; */
  
}
label{
  font-size: 20px;
  margin-bottom: 10px;
}
.complain-form input,
.complain-form textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 15px;
  box-sizing: border-box;
}

.complain-form button {
  /* padding: 8px; */
  background-color: rgb(196, 240, 0);
  border-radius: 20px;
  padding: 12px;
}
/* } */

.error-message {
  color: red;
  margin-top: 10px;
}

/* input {
  border-top: none;
  border-right: none;
  border-left: none;
} */


.complain-item {
  border: 1px solid #e7e7e7;
  margin-bottom: 10px;
  font : 400 1rem Quicksand, sans-serif;
  padding: 10px;
  border-radius: 10px;
}
/* .item-span {
  position: relative;s
  top: -71px;
  left: 40px;
} */
</style>
