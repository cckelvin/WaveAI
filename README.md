<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
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
      transition: background 0.3s, color 0.3s;
    }
    body.light-mode {
      background-color: #f5f7fa;
      color: #1f2937;
    }
    header {
      padding: 15px;
      text-align: center;
      font-size: 1.5em;
      font-weight: bold;
      color: #00e6e6;
      border-bottom: 1px solid #1a1a1a;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .light-mode header {
      border-bottom: 1px solid #d1d5db;
      background-color: #3b82f6;
    }
    #chat {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
    .light-mode #chat {
      background-color: #f9fafb;
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
    .light-mode .user {
      background: #2563eb;
    }
    .bot {
      align-self: flex-start;
      background: #1a1a1a;
      color: #e6e6e6;
      border: 1px solid #333;
    }
    .light-mode .bot {
      background: #e5e7eb;
      color: #1f2937;
      border: 1px solid #d1d5db;
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
    .light-mode footer {
      border-top: 1px solid #d1d5db;
      background: #ffffff;
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
    .light-mode input {
      background: #f9fafb;
      border: 1px solid #d1d5db;
      color: #1f2937;
    }
    input::placeholder {
      color: #6b7280;
    }
    .light-mode input::placeholder {
      color: #9ca3af;
    }
    button.send-btn {
      width: 40px;
      height: 40px;
      margin-left: 10px;
      border: none;
      border-radius: 50%;
      background: #00e6e6;
      color: #0d0d0d;
      font-size: 1.2em;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .light-mode button.send-btn {
      background: #2563eb;
      color: #ffffff;
    }
    button.send-btn:hover {
      background: #00b3b3;
    }
    .light-mode button.send-btn:hover {
      background: #1d4ed8;
    }
    button.theme-btn {
      padding: 8px;
      background: #004466;
      color: #e6e6e6;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 1em;
    }
    .light-mode button.theme-btn {
      background: #d1d5db;
      color: #1f2937;
    }
    button.theme-btn:hover {
      background: #006688;
    }
    .light-mode button.theme-btn:hover {
      background: #9ca3af;
    }
    button.clear-btn {
      padding: 8px 16px;
      background: #004466;
      color: #e6e6e6;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.9em;
      margin-left: 8px;
    }
    .light-mode button.clear-btn {
      background: #d1d5db;
      color: #1f2937;
    }
    button.clear-btn:hover {
      background: #006688;
    }
    .light-mode button.clear-btn:hover {
      background: #9ca3af;
    }
  </style>
</head>
<body>
  <header>
    <span>Wave AI</span>
    <div>
      <button class="theme-btn" onclick="toggleTheme()">☀️</button>
      <button class="clear-btn" onclick="clearHistory()">Clear History</button>
    </div>
  </header>
  <div id="chat">
    <div class="message bot">Welcome to the future! I'm Wave AI, your cosmic guide to answers pulled straight from the web. What's sparking your curiosity? 🌌</div>
  </div>
  <footer>
    <input id="userInput" type="text" placeholder="Ask Wave AI anything...">
    <button class="send-btn" onclick="sendMessage()">➡️</button>
  </footer>

  <script>
    const API_KEY = "AIzaSyCM7rgQ8J-Rw2fIyURuga-KfNDW3dZn9r4";
    const CSE_ID = "c7c4ecb03db19434e";
    const chat = document.getElementById("chat");
    const input = document.getElementById("userInput");
    let chatHistory = [];

    // Initialize chat
    document.addEventListener("DOMContentLoaded", () => {
      console.log("Page loaded, initializing Wave AI");
      const savedHistory = localStorage.getItem("chatHistory");
      if (savedHistory) {
        chat.innerHTML = savedHistory;
        chatHistory = JSON.parse(localStorage.getItem("chatLearningHistory") || "[]");
      }
      chat.scrollTop = chat.scrollHeight;
      if (localStorage.getItem("theme") === "light") {
        document.body.classList.add("light-mode");
      }
      if (input) {
        input.focus();
        console.log("Input field focused");
      } else {
        console.error("Input field not found");
      }
    });

    function appendMessage(content, sender, isTyping = false) {
      const msg = document.createElement("div");
      msg.classList.add("message", sender);
      if (isTyping) {
        msg.innerHTML = `<div class="typing"><div class="dot"></div><div class="dot"></div><div class="dot"></div></div>`;
      } else {
        msg.textContent = content;
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

      try {
        console.log("Searching web for:", text);
        const url = `https://www.googleapis.com/customsearch/v1?key=${API_KEY}&cx=${CSE_ID}&q=${encodeURIComponent(text)}`;
        const res = await fetch(url);
        const data = await res.json();
        chat.removeChild(typingMsg);

        if (data.items && data.items.length > 0) {
          const answer = summarizeAndConfirmSearchResults(data, text);
          appendMessage(answer, "bot");
          chatHistory.push({ role: "assistant", content: answer });
        } else {
          const fallback = `I couldn't find reliable info on "${text}". Please clarify or try a different question, and I'll dive back into the web for you!`;
          appendMessage(fallback, "bot");
          chatHistory.push({ role: "assistant", content: fallback });
        }
      } catch (e) {
        console.error("Search error:", e);
        chat.removeChild(typingMsg);
        const errorMsg = "Hmm, something broke in the cosmos—try again?";
        appendMessage(errorMsg, "bot");
        chatHistory.push({ role: "assistant", content: errorMsg });
      }

      // Save history
      localStorage.setItem("chatHistory", chat.innerHTML);
      localStorage.setItem("chatLearningHistory", JSON.stringify(chatHistory.slice(-50)));
      console.log("Chat and learning history saved");
    }

    function getContextFromHistory(query) {
      const recent = chatHistory.slice(-5);
      for (let msg of recent) {
        if (msg.role === "user" && msg.content.toLowerCase().includes(query.toLowerCase().split(" ")[0])) {
          return msg.content.split(" ")[0].toLowerCase();
        }
      }
      return null;
    }

    function summarizeAndConfirmSearchResults(data, originalQuery) {
      if (!data.items || data.items.length === 0) {
        return `I'm sorry, I couldn't find reliable info on "${originalQuery}". Please clarify or try a different question, and I'll dive back into the web for you!`;
      }

      const topResults = data.items.slice(0, 5);
      let keyFacts = [];
      let alternatives = [];

      // Extract key facts and alternatives
      topResults.forEach((item) => {
        const snippet = item.snippet.toLowerCase();
        const title = item.title.toLowerCase();
        const appMatch = snippet.match(/(paypal|wise|metamask|zengo|cash app|venmo|revolut|coinbase|binance|skrill)/gi) || title.match(/(paypal|wise|metamask|zengo|cash app|venmo|revolut|coinbase|binance|skrill)/gi);
        if (appMatch) {
          alternatives.push(appMatch[0]);
        }
        const factMatches = snippet.match(/([a-zA-Z\s]+?)\s+(won|is|are|means|results? in|key|definition|supports)\s*[:\s]*([a-zA-Z\s\d.,;]+?)(?=\.|,|$)/gi) || [];
        if (factMatches && factMatches.length >= 3) {
          keyFacts.push(factMatches[3].trim());
        } else {
          keyFacts.push(snippet.substring(0, 100));
        }
      });

      // Add context
      let response = "";
      const context = getContextFromHistory(originalQuery);
      if (context) {
        response = `Since you asked about ${context} earlier, here's a tailored answer:\n\n`;
      }

      // Handle MiniPay-like queries
      const queryLower = originalQuery.toLowerCase();
      if (queryLower.includes("mini pay") || queryLower.includes("minipay")) {
        response += `I'm assuming you meant "apps similar to MiniPay" based on your query and the context from search results. MiniPay is a self-custodial stablecoin wallet built on the Celo blockchain, designed for fast, low-cost global transactions using USD stablecoins like USDT, USDC, and cUSD. It supports sending and receiving funds in over 50 countries, converting to 35+ local currencies, and everyday use cases like paying bills or buying gift cards. If you meant something else by “Mini Pay,” please clarify, and I’ll tailor the response accordingly.\n\n`;
        response += `Here are some apps similar to MiniPay, focusing on mobile payment apps, stablecoin wallets, and money transfer services with comparable features:\n\n`;

        alternatives = [...new Set(alternatives)].slice(0, 3);
        if (alternatives.length === 0) {
          alternatives = ["PayPal", "Wise", "MetaMask"];
        }

        alternatives.forEach((app, index) => {
          let appDetails = "";
          switch (app.toLowerCase()) {
            case "paypal":
              appDetails = `**${app}**\n` +
                           `**Why Similar**: PayPal is a widely used payment platform supporting online and in-store transactions, peer-to-peer (P2P) transfers, and international payments in multiple currencies. It’s accessible on Web, iOS, and Android, with features like PayPal Credit and cryptocurrency support (e.g., Bitcoin).\n` +
                           `**Key Features**: Secure payments, integration with 200+ countries’ banks, two-factor authentication, and recurring billing.\n` +
                           `**Differences**: PayPal is not blockchain-based, and international transfers may incur higher fees (e.g., $2.99–$3.99 for some countries) compared to MiniPay’s near-zero fees. It’s more focused on traditional fiat transactions.\n` +
                           `**Best For**: Online shopping and P2P payments globally, though less specialized for stablecoin use.\n`;
              break;
            case "wise":
              appDetails = `**${app}**\n` +
                           `**Why Similar**: Wise specializes in low-cost international money transfers, supporting multiple currencies and direct bank account integration, similar to MiniPay’s global reach and local currency conversion.\n` +
                           `**Key Features**: Transparent low fees, real-time exchange rates, and support for sending money to 70+ countries.\n` +
                           `**Differences**: Wise is not a crypto wallet and focuses on fiat currency transfers, lacking MiniPay’s stablecoin functionality and blockchain-based transparency.\n` +
                           `**Best For**: Cost-effective international transfers without crypto involvement.\n`;
              break;
            case "metamask":
              appDetails = `**${app}**\n` +
                           `**Why Similar**: Like MiniPay, MetaMask is a non-custodial wallet with strong support for stablecoins (e.g., USDC, USDT) and blockchain transactions, including DeFi integrations and NFT marketplaces. It's available on mobile (iOS/Android) and browser extensions.\n` +
                           `**Key Features**: Customizable transaction settings, built-in token swaps, slippage controls, and high security scores.\n` +
                           `**Differences**: MetaMask is more geared toward Ethereum and multi-chain ecosystems, requiring seed phrase management, unlike MiniPay's phone-number simplicity and Celo focus. Fees can vary based on network gas.\n` +
                           `**Best For**: Advanced users exploring DeFi and crypto beyond basic stablecoin sends.\n`;
              break;
            default:
              appDetails = `**${app}**\n` +
                           `**Why Similar**: This app offers payment or transfer services with global reach, similar to MiniPay’s focus on accessible transactions.\n` +
                           `**Key Features**: Supports multiple currencies and mobile payments.\n` +
                           `**Differences**: It may not use blockchain or stablecoins, differing from MiniPay’s crypto focus.\n` +
                           `**Best For**: General payment needs, though specifics depend on the platform.\n`;
          }
          response += `${index + 1}. ${appDetails}\n`;
        });

        response += `These alternatives capture MiniPay's essence of accessible, low-cost global payments but vary in crypto focus. If you're looking for something specific like more stablecoin options or fiat-only, let me know!`;
      } else {
        // General query handling
        const primaryFact = keyFacts[0] || `Here's what I found on ${originalQuery}`;
        response += `${primaryFact}\n\nBased on available information, here are some key insights:\n\n`;
        keyFacts.slice(0, 3).forEach((fact, index) => {
          response += `${index + 1}. ${fact}\n`;
        });
        response += `\nIf you want more details or a specific angle, just let me know!`;
      }

      // Add cosmic flair
      if (Math.random() > 0.7) {
        response += `\n\nReady to explore more cosmic queries? 🌌`;
      }

      return response;
    }

    function toggleTheme() {
      console.log("Toggling theme");
      document.body.classList.toggle("light-mode");
      localStorage.setItem("theme", document.body.classList.contains("light-mode") ? "light" : "dark");
    }

    function clearHistory() {
      console.log("Clearing history");
      chatHistory = [];
      chat.innerHTML = `<div class="message bot">Welcome to the future! I'm Wave AI, your cosmic guide to answers pulled straight from the web. What's sparking your curiosity? 🌌</div>`;
      localStorage.setItem("chatHistory", chat.innerHTML);
      localStorage.setItem("chatLearningHistory", JSON.stringify(chatHistory));
      chat.scrollTop = chat.scrollHeight;
    }

    input.addEventListener("keypress", e => {
      if (e.key === "Enter") {
        console.log("Enter key pressed");
        sendMessage();
      }
    });
  </script>
</body>
</html>