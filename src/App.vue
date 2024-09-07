<script setup>
import { GoogleGenerativeAI } from "@google/generative-ai";
import MarkdownIt from "markdown-it";
import { ref } from "vue";

// API Key
const API_KEY = "AIzaSyC9bSlfrbfdc-2cYgVpJ5_65QaF3H895c4";

// Refs
const promptInput = ref("");
const chatHistory = ref([]); // Menyimpan riwayat percakapan

// Initialize Markdown-It
const md = new MarkdownIt();

// Inisialisasi API
const genAI = new GoogleGenerativeAI(API_KEY);

// Mulai chat dengan riwayat percakapan
const chat = genAI
  .getGenerativeModel({
    model: "gemini-1.5-flash",
  })
  .startChat({
    history: [
      {
        role: "user",
        parts: [{ text: "Hello, I have 2 dogs in my house." }],
      },
      {
        role: "model",
        parts: [{ text: "Great to meet you. What would you like to know?" }],
      },
    ],
    generationConfig: {
      maxOutputTokens: 1000,
    },
  });

// Function to check for inappropriate content
const isContentInappropriate = (content) => {
  const inappropriateWords = ["ngentot", "porno", "seksual", "kriminalitas"];
  return inappropriateWords.some((word) =>
    content.toLowerCase().includes(word)
  );
};

// Submit handler untuk mengirim pesan
const submitForm = async (event) => {
  event.preventDefault();

  if (isContentInappropriate(promptInput.value)) {
    chatHistory.value.push({
      role: "user",
      text: promptInput.value,
    });

    chatHistory.value.push({
      role: "model",
      text: "Maaf, pertanyaan Anda tidak bisa saya jawab karena berbau pornografi/kriminalitas.",
    });

    promptInput.value = "";
    resetTextareaHeight(); // Reset tinggi textarea setelah pesan terkirim
    return;
  }

  // Tambahkan pesan user ke riwayat percakapan
  chatHistory.value.push({
    role: "user",
    text: promptInput.value,
  });

  // Set status generating
  chatHistory.value.push({
    role: "system",
    text: "Generating...",
  });

  try {
    const result = await chat.sendMessage(promptInput.value);
    const response = await result.response;

    // Hapus status "Generating..."
    chatHistory.value.pop();

    // Tambahkan respons AI ke riwayat percakapan
    chatHistory.value.push({
      role: "model",
      text: response.text(),
    });

    // Reset textarea setelah pesan terkirim
    promptInput.value = "";
    resetTextareaHeight(); // Reset tinggi textarea setelah pesan terkirim
  } catch (e) {
    chatHistory.value.push({
      role: "error",
      text: e.toString(),
    });
  }
};

// Fungsi untuk menyesuaikan tinggi textarea
const adjustTextareaHeight = (event) => {
  const textarea = event.target;
  textarea.style.height = "auto"; // Reset height
  textarea.style.height = `${textarea.scrollHeight}px`; // Set height sesuai scrollHeight
};

// Fungsi untuk mereset tinggi textarea ke semula setelah pesan dikirim
const resetTextareaHeight = () => {
  const textarea = document.querySelector("textarea");
  if (textarea) {
    textarea.style.height = "40px"; // Mengembalikan tinggi textarea ke semula
  }
};
</script>

<template>
  <div class="chat-container">
    <h4>Curhat ceria dengan <span class="fancy">GEMINI</span></h4>
    <p class="made">Made with love by Barito Surya</p>

    <!-- Render chat history -->
    <div class="chat-history">
      <div
        v-for="(message, index) in chatHistory"
        :key="index"
        :class="message.role"
      >
        <div class="message-box">
          <img
            v-if="message.role === 'user'"
            class="avatar"
            src="../public/img/user.png"
            alt="User Avatar"
          />
          <img
            v-if="message.role === 'model'"
            class="avatar"
            src="../public/img/robot.png"
            alt="AI Avatar"
          />
          <div class="message-content">
            <p v-html="md.render(message.text)"></p>
          </div>
        </div>
      </div>
    </div>

    <!-- Form berada di bawah chat history -->
    <form @submit="submitForm">
      <div class="prompt-box">
        <img class="avatar" src="../public/img/user.png" alt="User Avatar" />
        <textarea
          v-model="promptInput"
          name="prompt"
          placeholder="Type message"
          rows="1"
          @input="adjustTextareaHeight"
        ></textarea>
        <button type="submit">
          <img src="../public/img/send.png" alt="Send Icon" />
        </button>
      </div>
    </form>
    <p class="version">Versi Beta 1.0</p>
  </div>
</template>

<style>
h4 {
  text-transform: uppercase;
  text-align: center;
  line-height: 1;
  font-weight: 900;
  text-shadow: 3px 2px 2px #171717;
}
.made {
  text-align: center;
  color: #9e9e9e;
}
.fancy {
  color: aquamarine;
}

.chat-container {
  background-color: #2d2d2d;
  color: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 600px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  margin: auto;
}

/* Chat history */
.chat-history {
  max-height: 400px;
  overflow-y: auto;
  margin-bottom: 20px;
}

/* Message box style */
.message-box {
  display: flex;
  align-items: flex-start;
  margin-bottom: 15px;
}

.message-box.user {
  justify-content: flex-end;
  align-self: flex-end;
}

.message-box.model {
  justify-content: flex-start;
}

.message-box .avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}

.user .avatar {
  order: 1;
  margin-left: 10px;
  margin-right: 0;
}

.message-content {
  max-width: 70%;
  padding: 10px;
  border-radius: 10px;
  position: relative;
}

.user .message-content {
  background-color: #1976d2;
  color: white;
  text-align: left;
  margin-left: auto;
}

.user .message-content:after {
  content: "";
  position: absolute;
  top: 10px;
  right: -10px;
  border-width: 10px;
  border-style: solid;
  border-color: #1976d2 transparent transparent transparent;
}

/* AI message bubble */
.model .message-content {
  background-color: #424242;
  color: white;
}

.model .message-content:after {
  content: "";
  position: absolute;
  top: 10px;
  left: -10px;
  border-width: 10px;
  border-style: solid;
  border-color: #424242 transparent transparent transparent;
}

/* Input form styling */
.prompt-box {
  display: flex;
  align-items: center;
  padding: 10px;
  background-color: #333;
  border-radius: 10px;
}

.prompt-box textarea {
  flex-grow: 1;
  padding: 10px;
  margin-left: 10px;
  font-size: 14px;
  border: none;
  border-radius: 5px;
  resize: none;
  height: 40px;
  background-color: #444;
  color: white;
  overflow-y: hidden;
}

.prompt-box button {
  background-color: #4040403d;
  margin-left: 2px;
  color: white;
  border: none;
  padding: 5px;
  border-radius: 5px;
  cursor: pointer;
}
.prompt-box button:hover {
  background-color: #155a9d;
}
.version {
  color: #9e9e9e;
  font-size: 14px;
  margin-top: 10px;
  text-align: center;
}

/* Media Queries for Responsive Design */

/* Untuk tablet */
@media (max-width: 768px) {
  .chat-container {
    width: 90%;
  }

  .message-box .avatar {
    width: 35px;
    height: 35px;
  }

  .message-content {
    max-width: 80%;
    font-size: 14px;
  }

  .prompt-box textarea {
    font-size: 13px;
    height: 35px;
  }

  .prompt-box button {
    padding: 8px;
  }
}

/* Untuk HP */
@media (max-width: 480px) {
  .chat-container {
    width: 100%;
    padding: 15px;
  }

  .message-box .avatar {
    width: 30px;
    height: 30px;
  }

  .message-content {
    max-width: 100%;
    font-size: 12px;
  }
  .prompt-box .avatar {
    width: 37px;
    height: auto;
  }
  .prompt-box textarea {
    font-size: 12px;
    height: 40px;
  }

  .prompt-box button {
    padding: 5px;
  }
}
</style>
