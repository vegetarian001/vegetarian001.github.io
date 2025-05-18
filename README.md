<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Discord荒らしツール</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #2c2f33;
      color: #ffffff;
      padding: 20px;
    }
    input, textarea {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      margin-bottom: 15px;
      border-radius: 5px;
      border: none;
      font-size: 14px;
    }
    textarea {
      resize: vertical;
    }
    label {
      font-weight: bold;
    }
    button {
      padding: 10px 20px;
      margin-right: 10px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #sendButton {
      background-color: #43b581;
      color: white;
    }
    #stopButton {
      background-color: #f04747;
      color: white;
    }
    #status {
      margin-top: 20px;
      font-size: 16px;
      color: #ddd;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <h2>連投ツール</h2>
<h2>※このツールの使用でアカウントが停止される可能性がありますので大事なアカウントで本ツールを使用しないで下さい</h2>

  <label>ユーザートークン（複数の場合改行区切り）:</label>
  <textarea id="tokens" rows="5"></textarea>

  <label>チャンネルID:</label>
  <input type="text" id="channel">

  <label>送信するメッセージ:</label>
  <textarea id="messages" rows="6"></textarea>

  <label>送信間隔（秒）:</label>
  <input type="number" id="interval" value="0.1">

  <label>送信数:</label>
  <input type="number" id="maxCount" value="10">

  <button id="sendButton" onclick="startSending()">送信開始</button>
  <button id="stopButton" onclick="stopSending()">停止</button>

  <p id="status">状態: 待機中</p>

  <script>
    let stopFlag = false;

    function shortToken(token) {
      return token.slice(0, 4) + "…" + token.slice(-4);
    }

    async function sendMessage(token, channelId, message) {
      const response = await fetch(`https://discord.com/api/v10/channels/${channelId}/messages`, {
        method: "POST",
        headers: {
          "Authorization": token,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ content: message })
      });
      return response.ok;
    }

    async function startSending() {
      stopFlag = false;
      const tokens = document.getElementById("tokens").value.split("\n").map(t => t.trim()).filter(t => t !== "");
      const channelId = document.getElementById("channel").value.trim();
      const messages = document.getElementById("messages").value.split("\n").filter(m => m.trim() !== "");
      const interval = parseInt(document.getElementById("interval").value) * 1000;
      const maxCount = parseInt(document.getElementById("maxCount").value);
      const status = document.getElementById("status");

      status.textContent = "送信中...\n";
      let sent = 0;

      outerLoop:
      while (sent < maxCount && !stopFlag) {
        for (let token of tokens) {
          if (stopFlag || sent >= maxCount) break outerLoop;

          const message = messages[sent % messages.length];
          const ok = await sendMessage(token, channelId, message);
          const short = shortToken(token);
          status.textContent += (ok ? "🟢" : "❌") + short + " → " + (ok ? "送信成功" : "送信失敗") + "\n";
          sent++;

          if (sent < maxCount && !stopFlag) {
            await new Promise(r => setTimeout(r, interval));
          }
        }
      }

      status.textContent += stopFlag ? "\n⛔ 送信停止しました。" : "\n✅ すべてのメッセージ送信完了。";
    }

    function stopSending() {
      stopFlag = true;
    }
  </script>
</body>
</html>
