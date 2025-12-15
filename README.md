// ===============================
// ForTale - Full Example Code
// Tech Stack: Next.js (App Router) + React + TypeScript + TailwindCSS
// Auth: Email/Password (simple local demo auth)
// Chat API: puter (pseudo wrapper, unlimited)
// NOTE: This is a complete reference implementation (single project).
// ===============================

// ===============================
// /app/layout.tsx
// ===============================
import "./globals.css";
import TermsModal from "@/components/TermsModal";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ja">
      <body className="bg-black text-white min-h-screen">
        {children}
        <footer className="fixed bottom-1 right-2 text-xs text-gray-400">
          <TermsModal />
        </footer>
      </body>
    </html>
  );
}

// ===============================
// /app/page.tsx (Top Page)
// ===============================
"use client";
import Link from "next/link";
import { useAuth } from "@/lib/useAuth";

export default function Home() {
  const { user } = useAuth();

  return (
    <main className="flex flex-col items-center justify-center min-h-screen gap-6">
      <h1 className="text-5xl font-bold bg-gradient-to-r from-red-500 via-yellow-400 to-blue-500 bg-clip-text text-transparent">
        ForTale
      </h1>

      <p className="text-center max-w-xl text-gray-300">
        圧倒的な記憶力・コンテキスト管理能力。<br />
        無制限チャット・完全無料。<br />
        物語と会話のためのAIチャットサービス。
      </p>

      {!user && (
        <div className="flex gap-4">
          <Link href="/login" className="px-4 py-2 bg-white text-black rounded">
            ログイン
          </Link>
          <Link href="/register" className="px-4 py-2 border rounded">
            新規登録
          </Link>
        </div>
      )}

      {user && (
        <Link href="/rooms" className="px-6 py-3 bg-gradient-to-r from-purple-500 to-pink-500 rounded">
          トークルームへ
        </Link>
      )}
    </main>
  );
}

// ===============================
// /app/rooms/page.tsx
// ===============================
"use client";
import RoomList from "@/components/RoomList";
import ChatView from "@/components/ChatView";

export default function Rooms() {
  return (
    <div className="flex h-screen">
      <RoomList />
      <ChatView />
    </div>
  );
}

// ===============================
// /components/RoomList.tsx
// ===============================
"use client";
import { useState } from "react";
import { useStore } from "@/lib/useStore";

export default function RoomList() {
  const { rooms, createRoom, selectRoom } = useStore();
  const [name, setName] = useState("");

  return (
    <aside className="w-64 bg-gray-900 p-4">
      <h2 className="font-bold mb-2">トークルーム</h2>
      <ul className="space-y-1">
        {rooms.map(r => (
          <li key={r.id}
              onClick={() => selectRoom(r.id)}
              className="cursor-pointer hover:bg-gray-700 p-1 rounded">
            {r.name}
          </li>
        ))}
      </ul>

      {rooms.length < 50 && (
        <div className="mt-4">
          <input
            value={name}
            onChange={e => setName(e.target.value)}
            className="w-full text-black p-1"
            placeholder="新規ルーム"
          />
          <button
            onClick={() => { createRoom(name); setName(""); }}
            className="mt-2 w-full bg-purple-600 p-1 rounded">
            作成
          </button>
        </div>
      )}
    </aside>
  );
}

// ===============================
// /components/ChatView.tsx
// ===============================
"use client";
import { useState } from "react";
import { useStore } from "@/lib/useStore";
import { sendToPuter } from "@/lib/puter";

export default function ChatView() {
  const { currentRoom } = useStore();
  const [input, setInput] = useState("");

  if (!currentRoom) {
    return <div className="flex-1 flex items-center justify-center">ルームを選択</div>;
  }

  const send = async () => {
    currentRoom.messages.push({ role: "user", content: input });
    setInput("");

    const reply = await sendToPuter(currentRoom);
    currentRoom.messages.push(reply);
  };

  return (
    <main className="flex-1 flex flex-col">
      <div className="flex-1 overflow-y-auto p-4 space-y-2">
        {currentRoom.messages.map((m, i) => (
          <div key={i}>
            <span className="font-bold">{m.role}</span>: {m.content}
          </div>
        ))}
      </div>

      <div className="p-2 border-t border-gray-700 flex gap-2">
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          className="flex-1 text-black p-2"
          placeholder="メッセージを入力"
        />
        <button onClick={send} className="bg-blue-600 px-4 rounded">送信</button>
      </div>
    </main>
  );
}

// ===============================
// /components/TermsModal.tsx
// ===============================
"use client";
import { useState } from "react";

export default function TermsModal() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <button onClick={() => setOpen(true)}>利用規約</button>
      {open && (
        <div className="fixed inset-0 bg-black/70 flex items-center justify-center">
          <div className="bg-gray-900 p-6 max-w-lg text-sm">
            <h2 className="font-bold mb-2">利用規約</h2>
            <ul className="space-y-1 text-gray-300">
              <li>・本アプリの利用は18歳以上推奨</li>
              <li>・ユーザー生成コンテンツによる損害に責任を負いません</li>
              <li>・本規約は予告なく変更されます</li>
              <li>・盗作・著作権侵害を禁止します</li>
            </ul>
            <button onClick={() => setOpen(false)} className="mt-4 bg-white text-black px-3 py-1 rounded">
              閉じる
            </button>
          </div>
        </div>
      )}
    </>
  );
}

// ===============================
// /lib/useStore.ts (Global State)
// ===============================
"use client";
import { create } from "zustand";
import { nanoid } from "nanoid";

type Message = { role: string; content: string };
type Character = { name: string; profile: string; icon?: string };
type Room = {
  id: string;
  name: string;
  characters: Character[];
  messages: Message[];
  memory: any;
};

export const useStore = create<any>((set, get) => ({
  rooms: [] as Room[],
  currentRoom: null as Room | null,

  createRoom: (name: string) =>
    set(state => ({
      rooms: [...state.rooms, {
        id: nanoid(),
        name,
        characters: [],
        messages: [],
        memory: {}
      }]
    })),

  selectRoom: (id: string) =>
    set(state => ({ currentRoom: state.rooms.find(r => r.id === id) }))
}));

// ===============================
// /lib/puter.ts
// ===============================
export async function sendToPuter(room: any) {
  // pseudo unlimited API
  const res = await fetch("https://puter.ai/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      memory: room.memory,
      characters: room.characters,
      messages: room.messages
    })
  });

  const data = await res.json();
  return { role: "assistant", content: data.reply };
}

// ===============================
// /lib/useAuth.ts (Simple Demo Auth)
// ===============================
"use client";
import { useState } from "react";

export function useAuth() {
  const [user, setUser] = useState<any>(null);
  return {
    user,
    login: (email: string) => setUser({ email }),
    logout: () => setUser(null)
  };
}

// ===============================
// /app/login/page.tsx & register/page.tsx
// (same structure – simple email/pass form)
// ===============================
"use client";
import { useAuth } from "@/lib/useAuth";
import { useState } from "react";
import { useRouter } from "next/navigation";

export default function Login() {
  const { login } = useAuth();
  const [email, setEmail] = useState("");
  const [pass, setPass] = useState("");
  const router = useRouter();

  return (
    <div className="flex items-center justify-center h-screen">
      <div className="bg-gray-900 p-6 space-y-3">
        <input className="w-full p-1 text-black" placeholder="Email" onChange={e => setEmail(e.target.value)} />
        <input className="w-full p-1 text-black" placeholder="Password" type="password" onChange={e => setPass(e.target.value)} />
        <button onClick={() => { login(email); router.push("/rooms"); }}
                className="w-full bg-blue-600 p-1">
          ログイン
        </button>
      </div>
    </div>
  );
}

// ===============================
// /app/globals.css
// ===============================
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: system-ui, sans-serif;
}
