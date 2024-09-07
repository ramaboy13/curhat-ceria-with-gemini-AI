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
      maxOutputTokens: 100000,
    },
  });

// Function to check for inappropriate content
const isContentInappropriate = (content) => {
  const inappropriateWords = ["ngentot", "porno", "seksual", "kriminalitas"]; // Tambahkan kata-kata tidak pantas di sini
  return inappropriateWords.some((word) =>
    content.toLowerCase().includes(word)
  );
};

// Submit handler untuk mengirim pesan
const submitForm = async () => {
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
  } catch (e) {
    chatHistory.value.push({
      role: "error",
      text: e.toString(),
    });
  }
};
</script>

<template>
  <div class="chat-container">
    <h1>Curhat ceria bersama <span class="fancy">GEMINI</span></h1>
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
    <form @submit.prevent="submitForm">
      <div class="prompt-box">
        <img class="avatar" src="../public/img/user.png" alt="User Avatar" />
        <textarea
          v-model="promptInput"
          name="prompt"
          placeholder="Type message"
          type="text"
        ></textarea>
        <button type="submit">Send</button>
      </div>
    </form>
  </div>
</template>

<style>
h1 {
  margin-left: 23px;
  font-size: 1.1vw;
  text-transform: uppercase;
  text-align: center;
  line-height: 1;
  font-family: "Work Sans", sans-serif;
  font-weight: 900;
}
.made {
  text-align: center;
  color: lightblue;
}

.fancy {
  @supports (background-clip: text) or (-webkit-background-clip: text) {
    background-image: url("data:image/svg+xml,%3Csvg width='2250' height='900' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cg%3E%3Cpath fill='%2300A080' d='M0 0h2255v899H0z'/%3E%3Ccircle cx='366' cy='207' r='366' fill='%2300FDCF'/%3E%3Ccircle cx='1777.5' cy='318.5' r='477.5' fill='%2300FDCF'/%3E%3Ccircle cx='1215' cy='737' r='366' fill='%23008060'/%3E%3C/g%3E%3C/svg%3E%0A");
    background-size: 110% auto;
    background-position: center;
    color: transparent;
    -webkit-background-clip: text;
    background-clip: text;
  }
}
.chat-container {
  background-color: #2d2d2d;
  color: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 500px;
  margin: auto;
  font-family: "Arial", sans-serif;
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
  align-self: flex-end; /* Tambahkan ini */
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

/* User message bubble */
.user .message-content {
  background-color: #1976d2;
  color: white;
  text-align: left;
  margin-left: auto; /* Tambahkan ini agar message content menempel di kanan */
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
  border-radius: 5px;
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
}

.prompt-box button {
  background-color: #1976d2;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}

.prompt-box button:hover {
  background-color: #155a9d;
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
    max-width: 85%;
    font-size: 12px;
  }

  .prompt-box textarea {
    font-size: 12px;
    height: 30px;
  }

  .prompt-box button {
    padding: 7px;
  }
}
</style>
