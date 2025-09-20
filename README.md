<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Wave AI</title>
  <style>
    body {
      background-color: #0d0d0d;
      color: #e6e6e6;
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    header {
      padding: 15px;
      text-align: center;
      font-size: 1.5em;
      font-weight: bold;
      color: #00e6e6;
      border-bottom: 1px solid #1a1a1a;
    }
    #chat {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
    .message {
      max-width: 75%;
      padding: 12px 16px;
      border-radius: 18px;
      line-height: 1.5;
      animation: fadeIn 0.3s ease-in;
      white-space: pre-wrap;
    }
    .user {
      align-self: flex-end;
      background: #004466;
      color: #fff;
    }
    .bot {
      align-self: flex-start;
      background: #1a1a1a;
      color: #e6e6e6;
      border: 1px solid #333;
      position: relative;
    }
    .typing {
      display: flex;
      gap: 4px;
    }
    .dot {
      width: 6px;
      height: 6px;
      border-radius: 50%;
      background: #00e6e6;
      animation: blink 1s infinite alternate;
    }
    .dot:nth-child(2) { animation-delay: 0.2s; }
    .dot:nth-child(3) { animation-delay: 0.4s; }

    @keyframes blink { from {opacity: 0.3;} to {opacity: 1;} }
    @keyframes fadeIn { from {opacity:0; transform:translateY(10px);} to {opacity:1; transform:translateY(0);} }

    footer {
      display: flex;
      padding: 12px;
      border-top: 1px solid #1a1a1a;
      background: #0d0d0d;
    }
    input {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 12px;
      background: #1a1a1a;
      color: #e6e6e6;
      font-size: 1em;
      outline: none;
    }
    button {
      margin-left: 10px;
      padding: 12px 18px;
      border: none;
      border-radius: 12px;
      background: #00e6e6;
      color: #0d0d0d;
      font-weight: bold;
      cursor: pointer;
    }
    audio {
      margin-top: 10px;
      width: 100%;
    }
    code {
      background: #111;
      padding: 2px 6px;
      border-radius: 6px;
      font-family: monospace;
      color: #00e6e6;
    }
  </style>
</head>
<body>
  <header>Wave AI</header>
  <div id="chat"></div>
  <footer>
    <input id="userInput" type="text" placeholder="Type your message...">
    <button onclick="sendMessage()">Send</button>
  </footer>

  <script>
    const API_KEY = "AIzaSyCM7rgQ8J-Rw2fIyURuga-KfNDW3dZn9r4";
    const CSE_ID  = "c7c4ecb03db19434e";
    const chat = document.getElementById("chat");
    const input = document.getElementById("userInput");

    const typeSound = new Audio("https://actions.google.com/sounds/v1/cartoon/typing.ogg");

    function appendMessage(content, sender, isTyping=false) {
      const msg = document.createElement("div");
      msg.classList.add("message", sender);
      if (isTyping) {
        msg.innerHTML = `<div class="typing"><div class="dot"></div><div class="dot"></div><div class="dot"></div></div>`;
      } else {
        msg.innerHTML = content;
      }
      chat.appendChild(msg);
      chat.scrollTop = chat.scrollHeight;
      return msg;
    }

    async function sendMessage() {
      const text = input.value.trim();
      if (!text) return;
      appendMessage(text, "user");
      input.value = "";
      const typingMsg = appendMessage("", "bot", true);

      if (text.toLowerCase() === "hi" || text.toLowerCase() === "hello") {
        chat.removeChild(typingMsg);
        appendMessage("Hello, I'm Wave AI, your future AI assistant.", "bot");
        return;
      }

      // Detect mode: code, audio, or general
      if (/code|program|script/i.test(text)) {
        chat.removeChild(typingMsg);
        const codeSample = generateCode(text);
        simulateTyping(codeSample, "bot");
        return;
      }
      if (/music|mp3|song/i.test(text)) {
        chat.removeChild(typingMsg);
        const audioHTML = `<p>I found an audio for you:</p>
          <audio controls src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"></audio>`;
        simulateTyping(audioHTML, "bot");
        return;
      }

      try {
        const url = `https://www.googleapis.com/customsearch/v1?key=${API_KEY}&cx=${CSE_ID}&q=${encodeURIComponent(text)}`;
        const res = await fetch(url);
        const data = await res.json();
        chat.removeChild(typingMsg);

        if (data.items && data.items.length > 0) {
          let snippets = data.items.map(i => i.snippet).join(" ");
          let answer = refineAnswer(snippets);
          const mp3Item = data.items.find(i => i.link && i.link.endsWith(".mp3"));
          if (mp3Item) {
            answer += `<br><audio controls src="${mp3Item.link}"></audio>`;
          }
          simulateTyping(answer, "bot");
        } else {
          simulateTyping(generateCreativeAnswer(text), "bot");
        }
      } catch (e) {
        chat.removeChild(typingMsg);
        simulateTyping("Something went wrong. Please try again.", "bot");
      }
    }

    function refineAnswer(text) {
      let sentences = text.split(/(?<=[.?!])\s+/);
      let unique = [...new Set(sentences)];
      return unique.slice(0, 5).join(" ");
    }

    function generateCreativeAnswer(query) {
      // For queries without results: fallback creative generator
      return `Here’s my take: ${query} could be understood in a creative way. Imagine expanding it with context, ideas, or analogies. Wave AI is thinking deeper and giving you a synthesized insight.`;
    }

    function generateCode(request) {
      if (/html/i.test(request)) {
        return `<pre><code>&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;&lt;title&gt;Demo&lt;/title&gt;&lt;/head&gt;
&lt;body&gt;Hello World&lt;/body&gt;
&lt;/html&gt;</code></pre>`;
      }
      if (/python/i.test(request)) {
        return `<pre><code>def greet():
    print("Hello from Wave AI")</code></pre>`;
      }
      return `<pre><code>// Example JS function
function waveAI() {
  console.log("Wave AI generated this code!");
}</code></pre>`;
    }

    function simulateTyping(text, sender) {
      let msg = appendMessage("", sender);
      let i = 0;
      let interval = setInterval(() => {
        msg.innerHTML += text.charAt(i);
        if (text.charAt(i).trim()) typeSound.play().catch(()=>{});
        i++;
        chat.scrollTop = chat.scrollHeight;
        if (i >= text.length) clearInterval(interval);
      }, 25);
    }

    input.addEventListener("keypress", e => { if (e.key === "Enter") sendMessage(); });
  </script>
</body>
</html>
