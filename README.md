# Chacha
import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { initializeApp } from "firebase/app";
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  onAuthStateChanged,
  signOut,
} from "firebase/auth";

// 🔥 Firebase Config (FIXED + CLEAN)
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY","AIzaSyDuN_RY6uze3qREPuKLOmenq0MJEW7aXds",
  authDomain: "web-app-add.firebaseapp.com",
  projectId: "web-app-add",
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);

export default function AIChacha() {
  const [user, setUser] = useState<any>(null);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [messages, setMessages] = useState<any[]>([]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const unsub = onAuthStateChanged(auth, (u) => setUser(u));
    return () => unsub();
  }, []);

  const login = async () => {
    try {
      await signInWithEmailAndPassword(auth, email, password);
    } catch (err: any) {
      alert(err.message);
    }
  };

  const signup = async () => {
    try {
      await createUserWithEmailAndPassword(auth, email, password);
    } catch (err: any) {
      alert(err.message);
    }
  };

  const logout = () => signOut(auth);

  // 🔥 REAL AI API CALL (SAFE + STABLE)
  const sendMessage = async () => {
    if (!input.trim()) return;

    const userMsg = { role: "user", text: input };
    setMessages((prev) => [...prev, userMsg]);
    setInput("");
    setLoading(true);

    try {
      const res = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: "Bearer YOUR_OPENAI_API_KEY","AIzaSyDuN_RY6uze3qREPuKLOmenq0MJEW7aXds",
        },
        body: JSON.stringify({
          model: "gpt-4o-mini",
          messages: [
            { role: "system", content: "You are AI Chacha, a helpful desi AI assistant." },
            ...messages.map((m) => ({ role: m.role, content: m.text })),
            { role: "user", content: input },
          ],
        }),
      });

      if (!res.ok) throw new Error("API failed");

      const data = await res.json();
      const aiText = data?.choices?.[0]?.message?.content || "No response";

      setMessages((prev) => [...prev, { role: "bot", text: aiText }]);
    } catch (error) {
      setMessages((prev) => [...prev, { role: "bot", text: "API Error ❌" }]);
    } finally {
      setLoading(false);
    }
  };

  // 🔐 LOGIN UI
  if (!user) {
    return (
      <div className="bg-black text-white min-h-screen flex items-center justify-center">
        <Card className="bg-gray-900 w-80">
          <CardContent className="p-6 space-y-4">
            <h2 className="text-center text-xl">Login</h2>
            <input
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Email"
              className="w-full p-2 bg-gray-800"
            />
            <input
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Password"
              type="password"
              className="w-full p-2 bg-gray-800"
            />
            <Button onClick={login}>Login</Button>
            <Button onClick={signup} variant="outline">
              Signup
            </Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  // 🚀 DASHBOARD
  return (
    <div className="bg-black text-white min-h-screen p-4">
      <div className="flex justify-between mb-4">
        <h1>AI Chacha 🚀</h1>
        <Button onClick={logout}>Logout</Button>
      </div>

      <Card className="bg-gray-900 max-w-2xl mx-auto">
        <CardContent className="p-4 h-[300px] overflow-y-auto space-y-2">
          {messages.map((m, i) => (
            <div
              key={i}
              className={`p-2 rounded ${m.role === "user" ? "bg-blue-600 ml-auto" : "bg-gray-700"}`}
            >
              {m.text}
            </div>
          ))}
          {loading && <div className="text-gray-400">AI is typing...</div>}
        </CardContent>
      </Card>

      <div className="flex gap-2 max-w-2xl mx-auto mt-4">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          className="flex-1 p-2 bg-gray-800"
          placeholder="Type your message..."
        />
        <Button onClick={sendMessage}>Send</Button>
      </div>
    </div>
  );
}
