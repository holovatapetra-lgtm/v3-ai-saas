gh repo create v3-ai-saas --public --clone && cd v3-ai-saas && \
npx create-next-app@latest . --js --app --eslint --tailwind --no-git && \
mkdir -p app/api/chat lib && \
cat > app/page.js << 'EOF'
"use client";
import { useState } from "react";

export default function Home() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");

  const send = async () => {
    const res = await fetch("/api/chat", {
      method: "POST",
      body: JSON.stringify({
        messages: [...messages, { role: "user", content: input }]
      })
    });

    const reader = res.body.getReader();
    const decoder = new TextDecoder();
    let text = "";

    while (true) {
      const { value, done } = await reader.read();
      if (done) break;
      text += decoder.decode(value);
      setMessages(prev => [...prev, { role: "assistant", content: text }]);
    }
  };

  return (
    <div className="h-screen bg-black text-white flex flex-col">
      <div className="flex-1 p-4 overflow-y-auto">
        {messages.map((m, i) => <div key={i}>{m.content}</div>)}
      </div>

      <div className="p-3 flex gap-2">
        <input className="flex-1 bg-zinc-900 p-2"
          value={input}
          onChange={(e) => setInput(e.target.value)} />
        <button onClick={send}>Send</button>
      </div>
    </div>
  );
}
EOF
