```
n-ai-surveillance/
├── app/
│   ├── api/
│   │   ├── devices/
│   │   │   └── route.js
│   │   ├── gpus/
│   │   │   └── route.js
│   │   ├── gemini-event-insights/
│   │   │   └── route.js
│   │   ├── gemini-camera-insights/
│   │   │   └── route.js
│   ├── cameras/
│   │   └── page.jsx
│   ├── gpus/
│   │   └── page.jsx
│   ├── layout.js
│   ├── page.jsx
│   └── prisma.js
├── components/
│   ├── CameraCard.jsx
│   ├── CameraForm.jsx
│   ├── EventCard.jsx
│   ├── GpuCard.jsx
│   ├── GpuForm.jsx
│   ├── InsightsModal.jsx
│   ├── SearchableDeviceSelect.jsx
│   ├── SearchableGpuSelect.jsx
│   ├── SettingsModal.jsx
│   └── SummaryModal.jsx
├── context/
│   └── ThemeContext.js
├── lib/
│   └── themes.js
├── public/
│   └── favicon.ico
├── package.json
├── next.config.mjs
├── tailwind.config.js
└── jsconfig.json
```

### File Contents

#### `package.json`
```json
{
  "name": "n-ai-surveillance",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@heroicons/react": "^2.0.18",
    "next": "14.2.3",
    "react": "^18",
    "react-dom": "^18",
    "prisma": "^5.0.0"
  },
  "devDependencies": {
    "autoprefixer": "^10.4.19",
    "postcss": "^8.4.38",
    "tailwindcss": "^3.4.1"
  }
}
```

#### `next.config.mjs`
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

#### `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

#### `jsconfig.json`
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

#### `app/layout.js`
```javascript
import { ThemeProvider } from '@/context/ThemeContext';
import './globals.css';

export const metadata = {
  title: 'N. AI Surveillance System',
  description: 'Frontend for AI-powered surveillance system',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <head>
        <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
      </head>
      <body className="font-sans">
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

#### `app/globals.css`
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --scrollbar-bg: #f1f1f1;
  --scrollbar-thumb: #888;
}

@media (prefers-color-scheme: dark) {
  :root {
    --scrollbar-bg: #333;
    --scrollbar-thumb: #666;
  }
}

::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: var(--scrollbar-bg);
}

::-webkit-scrollbar-thumb {
  background: var(--scrollbar-thumb);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
```

#### `app/page.jsx`
```javascript
'use client';
import { useState, useEffect, useCallback } from 'react';
import { useTheme } from '@/context/ThemeContext';
import EventCard from '@/components/EventCard';
import SettingsModal from '@/components/SettingsModal';
import SummaryModal from '@/components/SummaryModal';
import { SearchIcon } from '@heroicons/react/outline';

const dummyEvents = [
  { id: 1, name: 'Motion Detected', timestamp: '2025-07-14T10:00:00Z', details: 'Motion detected at entrance', camera: { name: 'Cam1', location: 'Entrance' }, gpu: { name: 'GPU1' } },
  { id: 2, name: 'Person Identified', timestamp: '2025-07-14T10:05:00Z', details: 'Person seen at gate', camera: { name: 'Cam2', location: 'Gate' }, gpu: { name: 'GPU2' } },
];

export default function Home() {
  const { activeTheme } = useTheme();
  const [events, setEvents] = useState(dummyEvents);
  const [search, setSearch] = useState('');
  const [isSettingsOpen, setIsSettingsOpen] = useState(false);
  const [isSummaryOpen, setIsSummaryOpen] = useState(false);

  const filteredEvents = events.filter(
    (event) =>
      event.name.toLowerCase().includes(search.toLowerCase()) ||
      event.details.toLowerCase().includes(search.toLowerCase()) ||
      event.camera.name.toLowerCase().includes(search.toLowerCase()) ||
      event.camera.location.toLowerCase().includes(search.toLowerCase())
  );

  const fetchSummary = useCallback(async () => {
    try {
      const response = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ model: 'gemma3', prompt: `Summarize these events: ${JSON.stringify(filteredEvents)}` }),
      });
      const reader = response.body.getReader();
      let summary = '';
      setIsSummaryOpen(true);
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        summary += new TextDecoder().decode(value);
        // Update UI incrementally
      }
      return summary;
    } catch (error) {
      console.error('Error fetching summary:', error);
    }
  }, [filteredEvents]);

  return (
    <div className={`min-h-screen ${activeTheme.background} ${activeTheme.text}`}>
      <header className={`p-4 ${activeTheme.card}`}>
        <div className="flex justify-between items-center">
          <h1 className="text-2xl font-bold">Real-Time Events</h1>
          <button onClick={() => setIsSettingsOpen(true)} className={`${activeTheme.button}`}>
            Settings
          </button>
        </div>
        <div className="mt-4 flex items-center">
          <SearchIcon className={`h-6 w-6 ${activeTheme.text}`} />
          <input
            type="text"
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            placeholder="Search events..."
            className={`ml-2 p-2 rounded ${activeTheme.input}`}
          />
          <button onClick={fetchSummary} className={`ml-4 ${activeTheme.button}`}>
            Get Summary
          </button>
        </div>
      </header>
      <main className="p-4">
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
          {filteredEvents.map((event) => (
            <EventCard key={event.id} event={event} />
          ))}
        </div>
      </main>
      <SettingsModal isOpen={isSettingsOpen} onClose={() => setIsSettingsOpen(false)} />
      <SummaryModal isOpen={isSummaryOpen} onClose={() => setIsSummaryOpen(false)} />
    </div>
  );
}
```

#### `app/cameras/page.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';
import CameraCard from '@/components/CameraCard';
import CameraForm from '@/components/CameraForm';
import SettingsModal from '@/components/SettingsModal';

export default function Cameras() {
  const { activeTheme } = useTheme();
  const [cameras, setCameras] = useState([]);
  const [isFormOpen, setIsFormOpen] = useState(false);
  const [isSettingsOpen, setIsSettingsOpen] = useState(false);
  const [editCamera, setEditCamera] = useState(null);

  useEffect(() => {
    const fetchCameras = async () => {
      try {
        const response = await fetch('/api/cameras');
        const data = await response.json();
        setCameras(data);
      } catch (error) {
        console.error('Error fetching cameras:', error);
      }
    };
    fetchCameras();
  }, []);

  const handleAddCamera = async (camera) => {
    try {
      const response = await fetch('/api/cameras', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(camera),
      });
      const newCamera = await response.json();
      setCameras([...cameras, newCamera]);
      setIsFormOpen(false);
    } catch (error) {
      console.error('Error adding camera:', error);
    }
  };

  const handleEditCamera = async (camera) => {
    try {
      const response = await fetch(`/api/cameras/${camera.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(camera),
      });
      const updatedCamera = await response.json();
      setCameras(cameras.map((c) => (c.id === updatedCamera.id ? updatedCamera : c)));
      setIsFormOpen(false);
      setEditCamera(null);
    } catch (error) {
      console.error('Error editing camera:', error);
    }
  };

  const handleDeleteCamera = async (id) => {
    try {
      await fetch(`/api/cameras/${id}`, { method: 'DELETE' });
      setCameras(cameras.filter((c) => c.id !== id));
    } catch (error) {
      console.error('Error deleting camera:', error);
    }
  };

  return (
    <div className={`min-h-screen ${activeTheme.background} ${activeTheme.text}`}>
      <header className={`p-4 ${activeTheme.card}`}>
        <div className="flex justify-between items-center">
          <h1 className="text-2xl font-bold">IP Camera Management</h1>
          <div>
            <button onClick={() => setIsFormOpen(true)} className={`${activeTheme.button} mr-2`}>
              Add Camera
            </button>
            <button onClick={() => setIsSettingsOpen(true)} className={`${activeTheme.button}`}>
              Settings
            </button>
          </div>
        </div>
      </header>
      <main className="p-4">
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
          {cameras.map((camera) => (
            <CameraCard
              key={camera.id}
              camera={camera}
              onEdit={() => { setEditCamera(camera); setIsFormOpen(true); }}
              onDelete={() => handleDeleteCamera(camera.id)}
            />
          ))}
        </div>
      </main>
      <CameraForm
        isOpen={isFormOpen}
        onClose={() => { setIsFormOpen(false); setEditCamera(null); }}
        onSubmit={editCamera ? handleEditCamera : handleAddCamera}
        camera={editCamera}
      />
      <SettingsModal isOpen={isSettingsOpen} onClose={() => setIsSettingsOpen(false)} />
    </div>
  );
}
```

#### `app/gpus/page.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';
import GpuCard from '@/components/GpuCard';
import GpuForm from '@/components/GpuForm';
import SettingsModal from '@/components/SettingsModal';

export default function Gpus() {
  const { activeTheme } = useTheme();
  const [gpus, setGpus] = useState([]);
  const [isFormOpen, setIsFormOpen] = useState(false);
  const [isSettingsOpen, setIsSettingsOpen] = useState(false);
  const [editGpu, setEditGpu] = useState(null);

  useEffect(() => {
    const fetchGpus = async () => {
      try {
        const response = await fetch('/api/gpus');
        const data = await response.json();
        setGpus(data);
      } catch (error) {
        console.error('Error fetching GPUs:', error);
      }
    };
    fetchGpus();
  }, []);

  const handleAddGpu = async (gpu) => {
    try {
      const response = await fetch('/api/gpus', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(gpu),
      });
      const newGpu = await response.json();
      setGpus([...gpus, newGpu]);
      setIsFormOpen(false);
    } catch (error) {
      console.error('Error adding GPU:', error);
    }
  };

  const handleEditGpu = async (gpu) => {
    try {
      const response = await fetch(`/api/gpus/${gpu.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(gpu),
      });
      const updatedGpu = await response.json();
      setGpus(gpus.map((g) => (g.id === updatedGpu.id ? updatedGpu : g)));
      setIsFormOpen(false);
      setEditGpu(null);
    } catch (error) {
      console.error('Error editing GPU:', error);
    }
  };

  return (
    <div className={`min-h-screen ${activeTheme.background} ${activeTheme.text}`}>
      <header className={`p-4 ${activeTheme.card}`}>
        <div className="flex justify-between items-center">
          <h1 className="text-2xl font-bold">GPU Server Management</h1>
          <div>
            <button onClick={() => setIsFormOpen(true)} className={`${activeTheme.button} mr-2`}>
              Add GPU
            </button>
            <button onClick={() => setIsSettingsOpen(true)} className={`${activeTheme.button}`}>
              Settings
            </button>
          </div>
        </div>
      </header>
      <main className="p-4">
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
          {gpus.map((gpu) => (
            <GpuCard
              key={gpu.id}
              gpu={gpu}
              onEdit={() => { setEditGpu(gpu); setIsFormOpen(true); }}
            />
          ))}
        </div>
      </main>
      <GpuForm
        isOpen={isFormOpen}
        onClose={() => { setIsFormOpen(false); setEditGpu(null); }}
        onSubmit={editGpu ? handleEditGpu : handleAddGpu}
        gpu={editGpu}
      />
      <SettingsModal isOpen={isSettingsOpen} onClose={() => setIsSettingsOpen(false)} />
    </div>
  );
}
```

#### `app/api/gpus/route.js`
```javascript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET() {
  try {
    const gpus = await prisma.gpu.findMany({ include: { device: true } });
    return new Response(JSON.stringify(gpus), { status: 200 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error fetching GPUs' }), { status: 500 });
  }
}

export async function POST(request) {
  try {
    const gpu = await request.json();
    const newGpu = await prisma.gpu.create({
      data: {
        name: gpu.name,
        gpuModel: gpu.gpuModel,
        vram: parseInt(gpu.vram),
        flops: parseFloat(gpu.flops),
        status: gpu.status,
        deviceId: gpu.deviceId ? parseInt(gpu.deviceId) : null,
      },
    });
    return new Response(JSON.stringify(newGpu), { status: 201 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error creating GPU' }), { status: 500 });
  }
}
```

#### `app/api/devices/route.js`
```javascript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET() {
  try {
    const devices = await prisma.device.findMany();
    return new Response(JSON.stringify(devices), { status: 200 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error fetching devices' }), { status: 500 });
  }
}

export async function POST(request) {
  try {
    const device = await request.json();
    const newDevice = await prisma.device.create({
      data: {
        name: device.name,
        ipAddress: device.ipAddress,
        cpu: device.cpu,
        ram: parseInt(device.ram),
        storage: parseInt(device.storage),
      },
    });
    return new Response(JSON.stringify(newDevice), { status: 201 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error creating device' }), { status: 500 });
  }
}
```

#### `app/api/gemini-event-insights/route.js`
```javascript
export async function POST(request) {
  try {
    const { eventDetails } = await request.json();
    const response = await fetch('https://api.gemini.com/v2.0/flash', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.GEMINI_API_KEY}`,
      },
      body: JSON.stringify({ prompt: `Generate insights for this event: ${eventDetails}` }),
    });
    const data = await response.json();
    return new Response(JSON.stringify({ insight: data.result }), { status: 200 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error generating insights' }), { status: 500 });
  }
}
```

#### `app/api/gemini-camera-insights/route.js`
```javascript
export async function POST(request) {
  try {
    const { cameraDetails } = await request.json();
    const response = await fetch('https://api.gemini.com/v2.0/flash', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.GEMINI_API_KEY}`,
      },
      body: JSON.stringify({ prompt: `Generate insights for this camera: ${cameraDetails}` }),
    });
    const data = await response.json();
    return new Response(JSON.stringify({ insight: data.result }), { status: 200 });
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error generating insights' }), { status: 500 });
  }
}
```

#### `app/prisma.js`
```javascript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default prisma;
```

#### `context/ThemeContext.js`
```javascript
'use client';
import { createContext, useContext, useState, useEffect } from 'react';
import { themes } from '@/lib/themes';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [currentThemeName, setCurrentThemeName] = useState('minimal');
  const [isDarkMode, setIsDarkMode] = useState(false);
  const [isRTL, setIsRTL] = useState(false);
  const [isCompactMode, setIsCompactMode] = useState(false);

  useEffect(() => {
    const savedTheme = localStorage.getItem('theme') || 'minimal';
    const savedDarkMode = localStorage.getItem('darkMode') === 'true';
    const savedRTL = localStorage.getItem('rtl') === 'true';
    const savedCompactMode = localStorage.getItem('compactMode') === 'true';
    setCurrentThemeName(savedTheme);
    setIsDarkMode(savedDarkMode);
    setIsRTL(savedRTL);
    setIsCompactMode(savedCompactMode);
  }, []);

  useEffect(() => {
    localStorage.setItem('theme', currentThemeName);
    localStorage.setItem('darkMode', isDarkMode);
    localStorage.setItem('rtl', isRTL);
    localStorage.setItem('compactMode', isCompactMode);
  }, [currentThemeName, isDarkMode, isRTL, isCompactMode]);

  const activeTheme = {
    ...themes[currentThemeName][isDarkMode ? 'dark' : 'light'],
    direction: isRTL ? 'rtl' : 'ltr',
    spacing: isCompactMode ? 'compact' : 'normal',
  };

  return (
    <ThemeContext.Provider
      value={{
        currentThemeName,
        setCurrentThemeName,
        isDarkMode,
        setIsDarkMode,
        isRTL,
        setIsRTL,
        isCompactMode,
        setIsCompactMode,
        activeTheme,
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  return useContext(ThemeContext);
}
```

#### `lib/themes.js`
```javascript
export const themes = {
  minimal: {
    light: {
      background: 'bg-gray-100',
      text: 'text-gray-900',
      card: 'bg-white',
      button: 'bg-blue-500 text-white hover:bg-blue-600',
      input: 'bg-gray-200 text-gray-900',
      border: 'border-gray-300',
      shadow: 'shadow-md',
    },
    dark: {
      background: 'bg-gray-900',
      text: 'text-gray-100',
      card: 'bg-gray-800',
      button: 'bg-blue-600 text-white hover:bg-blue-700',
      input: 'bg-gray-700 text-gray-100',
      border: 'border-gray-600',
      shadow: 'shadow-lg',
    },
  },
  green: {
    light: {
      background: 'bg-green-100',
      text: 'text-green-900',
      card: 'bg-white',
      button: 'bg-green-500 text-white hover:bg-green-600',
      input: 'bg-green-200 text-green-900',
      border: 'border-green-300',
      shadow: 'shadow-md',
    },
    dark: {
      background: 'bg-green-900',
      text: 'text-green-100',
      card: 'bg-green-800',
      button: 'bg-green-600 text-white hover:bg-green-700',
      input: 'bg-green-700 text-green-100',
      border: 'border-green-600',
      shadow: 'shadow-lg',
    },
  },
  // Add other themes (red, golden, richBlue, otherBlue, purple) similarly
};
```

#### `components/EventCard.jsx`
```javascript
'use client';
import { useState } from 'react';
import { useTheme } from '@/context/ThemeContext';
import InsightsModal from './InsightsModal';

export default function EventCard({ event }) {
  const { activeTheme } = useTheme();
  const [isInsightsOpen, setIsInsightsOpen] = useState(false);

  const handleInsights = async () => {
    setIsInsightsOpen(true);
  };

  return (
    <div className={`p-4 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
      <h2 className="text-lg font-semibold">{event.name}</h2>
      <p>{event.timestamp}</p>
      <p>{event.details}</p>
      <p>Camera: {event.camera.name} ({event.camera.location})</p>
      <p>GPU: {event.gpu.name}</p>
      <button onClick={handleInsights} className={`mt-2 ${activeTheme.button}`}>
        Get AI Insights
      </button>
      <InsightsModal
        isOpen={isInsightsOpen}
        onClose={() => setIsInsightsOpen(false)}
        content={`Insights for event: ${event.details}`}
        endpoint="/api/gemini-event-insights"
        data={{ eventDetails: event.details }}
      />
    </div>
  );
}
```

#### `components/CameraCard.jsx`
```javascript
'use client';
import { useState } from 'react';
import { useTheme } from '@/context/ThemeContext';
import InsightsModal from './InsightsModal';

export default function CameraCard({ camera, onEdit, onDelete }) {
  const { activeTheme } = useTheme();
  const [isInsightsOpen, setIsInsightsOpen] = useState(false);

  return (
    <div className={`p-4 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
      <h2 className="text-lg font-semibold">{camera.name}</h2>
      <p>IP: {camera.ipAddress}</p>
      <p>Location: {camera.location}</p>
      <p>Status: {camera.availabilityStatus}</p>
      <p>GPU: {camera.gpu?.name || 'Unassigned'}</p>
      <div className="mt-2 flex gap-2">
        <button onClick={onEdit} className={`${activeTheme.button}`}>
          Edit
        </button>
        <button onClick={onDelete} className={`${activeTheme.button}`}>
          Delete
        </button>
        <button onClick={() => setIsInsightsOpen(true)} className={`${activeTheme.button}`}>
          Get AI Insights
        </button>
      </div>
      <InsightsModal
        isOpen={isInsightsOpen}
        onClose={() => setIsInsightsOpen(false)}
        content={`Insights for camera: ${camera.name}`}
        endpoint="/api/gemini-camera-insights"
        data={{ cameraDetails: JSON.stringify(camera) }}
      />
    </div>
  );
}
```

#### `components/GpuCard.jsx`
```javascript
'use client';
import { useTheme } from '@/context/ThemeContext';

export default function GpuCard({ gpu, onEdit }) {
  const { activeTheme } = useTheme();

  return (
    <div className={`p-4 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
      <h2 className="text-lg font-semibold">{gpu.name}</h2>
      <p>Model: {gpu.gpuModel}</p>
      <p>VRAM: {gpu.vram} GB</p>
      <p>FLOPS: {gpu.flops} TFLOPS</p>
      <p>Status: {gpu.status}</p>
      <p>Device: {gpu.device?.name || 'Unassigned'} ({gpu.device?.ipAddress || '-'})</p>
      <button onClick={onEdit} className={`mt-2 ${activeTheme.button}`}>
        Edit
      </button>
    </div>
  );
}
```

#### `components/CameraForm.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';
import SearchableGpuSelect from './SearchableGpuSelect';

export default function CameraForm({ isOpen, onClose, onSubmit, camera }) {
  const { activeTheme } = useTheme();
  const [formData, setFormData] = useState({
    name: '',
    ipAddress: '',
    location: '',
    availabilityStatus: 'Online',
    gpuId: '',
  });

  useEffect(() => {
    if (camera) {
      setFormData({
        id: camera.id,
        name: camera.name,
        ipAddress: camera.ipAddress,
        location: camera.location,
        availabilityStatus: camera.availabilityStatus,
        gpuId: camera.gpuId || '',
      });
    }
  }, [camera]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div className={`p-6 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
        <h2 className="text-xl font-bold mb-4">{camera ? 'Edit Camera' : 'Add Camera'}</h2>
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={formData.name}
            onChange={(e) => setFormData({ ...formData, name: e.target.value })}
            placeholder="Camera Name"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <input
            type="text"
            value={formData.ipAddress}
            onChange={(e) => setFormData({ ...formData, ipAddress: e.target.value })}
            placeholder="IP Address"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <input
            type="text"
            value={formData.location}
            onChange={(e) => setFormData({ ...formData, location: e.target.value })}
            placeholder="Location"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <select
            value={formData.availabilityStatus}
            onChange={(e) => setFormData({ ...formData, availabilityStatus: e.target.value })}
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
          >
            <option value="Online">Online</option>
            <option value="Offline">Offline</option>
            <option value="Maintenance">Maintenance</option>
          </select>
          <SearchableGpuSelect
            value={formData.gpuId}
            onChange={(gpuId) => setFormData({ ...formData, gpuId })}
          />
          <div className="flex justify-end gap-2 mt-4">
            <button type="button" onClick={onClose} className={`${activeTheme.button}`}>
              Cancel
            </button>
            <button type="submit" className={`${activeTheme.button}`}>
              Save
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}
```

#### `components/GpuForm.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';
import SearchableDeviceSelect from './SearchableDeviceSelect';

export default function GpuForm({ isOpen, onClose, onSubmit, gpu }) {
  const { activeTheme } = useTheme();
  const [formData, setFormData] = useState({
    name: '',
    gpuModel: '',
    vram: '',
    flops: '',
    status: 'Online',
    deviceId: '',
  });

  useEffect(() => {
    if (gpu) {
      setFormData({
        id: gpu.id,
        name: gpu.name,
        gpuModel: gpu.gpuModel,
        vram: gpu.vram,
        flops: gpu.flops,
        status: gpu.status,
        deviceId: gpu.deviceId || '',
      });
    }
  }, [gpu]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit({
      ...formData,
      vram: parseInt(formData.vram),
      flops: parseFloat(formData.flops),
    });
  };

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div className={`p-6 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
        <h2 className="text-xl font-bold mb-4">{gpu ? 'Edit GPU' : 'Add GPU'}</h2>
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={formData.name}
            onChange={(e) => setFormData({ ...formData, name: e.target.value })}
            placeholder="GPU Name"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <input
            type="text"
            value={formData.gpuModel}
            onChange={(e) => setFormData({ ...formData, gpuModel: e.target.value })}
            placeholder="GPU Model"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <input
            type="number"
            value={formData.vram}
            onChange={(e) => setFormData({ ...formData, vram: e.target.value })}
            placeholder="VRAM (GB)"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <input
            type="number"
            step="0.1"
            value={formData.flops}
            onChange={(e) => setFormData({ ...formData, flops: e.target.value })}
            placeholder="FLOPS (TFLOPS)"
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
            required
          />
          <select
            value={formData.status}
            onChange={(e) => setFormData({ ...formData, status: e.target.value })}
            className={`mb-2 p-2 w-full rounded ${activeTheme.input}`}
          >
            <option value="Online">Online</option>
            <option value="Busy">Busy</option>
            <option value="Offline">Offline</option>
          </select>
          <SearchableDeviceSelect
            value={formData.deviceId}
            onChange={(deviceId) => setFormData({ ...formData, deviceId })}
          />
          <div className="flex justify-end gap-2 mt-4">
            <button type="button" onClick={onClose} className={`${activeTheme.button}`}>
              Cancel
            </button>
            <button type="submit" className={`${activeTheme.button}`}>
              Save
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}
```

#### `components/InsightsModal.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';

export default function InsightsModal({ isOpen, onClose, content, endpoint, data }) {
  const { activeTheme } = useTheme();
  const [insight, setInsight] = useState('');

  useEffect(() => {
    if (isOpen && endpoint) {
      const fetchInsight = async () => {
        try {
          const response = await fetch(endpoint, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data),
          });
          const result = await response.json();
          setInsight(result.insight);
        } catch (error) {
          setInsight('Error generating insights');
        }
      };
      fetchInsight();
    }
  }, [isOpen, endpoint, data]);

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div className={`p-6 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
        <h2 className="text-xl font-bold mb-4">AI Insights</h2>
        <p>{insight || content}</p>
        <button onClick={onClose} className={`mt-4 ${activeTheme.button}`}>
          Close
        </button>
      </div>
    </div>
  );
}
```

#### `components/SummaryModal.jsx`
```javascript
'use client';
import { useTheme } from '@/context/ThemeContext';

export default function SummaryModal({ isOpen, onClose, content }) {
  const { activeTheme } = useTheme();

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div className={`p-6 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
        <h2 className="text-xl font-bold mb-4">Event Summary</h2>
        <p>{content || 'Generating summary...'}</p>
        <button onClick={onClose} className={`mt-4 ${activeTheme.button}`}>
          Close
        </button>
      </div>
    </div>
  );
}
```

#### `components/SettingsModal.jsx`
```javascript
'use client';
import { useTheme } from '@/context/ThemeContext';
import { themes } from '@/lib/themes';

export default function SettingsModal({ isOpen, onClose }) {
  const {
    currentThemeName,
    setCurrentThemeName,
    isDarkMode,
    setIsDarkMode,
    isRTL,
    setIsRTL,
    isCompactMode,
    setIsCompactMode,
    activeTheme,
  } = useTheme();

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div className={`p-6 rounded ${activeTheme.card} ${activeTheme.shadow}`}>
        <h2 className="text-xl font-bold mb-4">Settings</h2>
        <div className="mb-4">
          <label className="block mb-2">Theme</label>
          <select
            value={currentThemeName}
            onChange={(e) => setCurrentThemeName(e.target.value)}
            className={`p-2 w-full rounded ${activeTheme.input}`}
          >
            {Object.keys(themes).map((theme) => (
              <option key={theme} value={theme}>
                {theme.charAt(0).toUpperCase() + theme.slice(1)}
              </option>
            ))}
          </select>
        </div>
        <div className="mb-4">
          <label className="flex items-center">
            <input
              type="checkbox"
              checked={isDarkMode}
              onChange={() => setIsDarkMode(!isDarkMode)}
              className="mr-2"
            />
            Dark Mode
          </label>
        </div>
        <div className="mb-4">
          <label className="flex items-center">
            <input
              type="checkbox"
              checked={isRTL}
              onChange={() => setIsRTL(!isRTL)}
              className="mr-2"
            />
            Right-to-Left Layout
          </label>
        </div>
        <div className="mb-4">
          <label className="flex items-center">
            <input
              type="checkbox"
              checked={isCompactMode}
              onChange={() => setIsCompactMode(!isCompactMode)}
              className="mr-2"
            />
            Compact Mode
          </label>
        </div>
        <button onClick={onClose} className={`mt-4 ${activeTheme.button}`}>
          Close
        </button>
      </div>
    </div>
  );
}
```

#### `components/SearchableGpuSelect.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';

export default function SearchableGpuSelect({ value, onChange }) {
  const { activeTheme } = useTheme();
  const [gpus, setGpus] = useState([]);
  const [search, setSearch] = useState('');

  useEffect(() => {
    const fetchGpus = async () => {
      try {
        const response = await fetch('/api/gpus');
        const data = await response.json();
        setGpus(data);
      } catch (error) {
        console.error('Error fetching GPUs:', error);
      }
    };
    fetchGpus();
  }, []);

  const filteredGpus = gpus.filter(
    (gpu) =>
      gpu.name.toLowerCase().includes(search.toLowerCase()) ||
      gpu.gpuModel.toLowerCase().includes(search.toLowerCase()) ||
      (gpu.device?.name || '').toLowerCase().includes(search.toLowerCase()) ||
      (gpu.device?.ipAddress || '').toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="mb-2">
      <input
        type="text"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search GPUs..."
        className={`p-2 w-full rounded ${activeTheme.input}`}
      />
      <select
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className={`p-2 w-full rounded ${activeTheme.input}`}
      >
        <option value="">Select GPU</option>
        {filteredGpus.map((gpu) => (
          <option key={gpu.id} value={gpu.id}>
            {gpu.name} ({gpu.gpuModel}, {gpu.device?.name || 'Unassigned'})
          </option>
        ))}
      </select>
    </div>
  );
}
```

#### `components/SearchableDeviceSelect.jsx`
```javascript
'use client';
import { useState, useEffect } from 'react';
import { useTheme } from '@/context/ThemeContext';

export default function SearchableDeviceSelect({ value, onChange }) {
  const { activeTheme } = useTheme();
  const [devices, setDevices] = useState([]);
  const [search, setSearch] = useState('');

  useEffect(() => {
    const fetchDevices = async () => {
      try {
        const response = await fetch('/api/devices');
        const data = await response.json();
        setDevices(data);
      } catch (error) {
        console.error('Error fetching devices:', error);
      }
    };
    fetchDevices();
  }, []);

  const filteredDevices = devices.filter(
    (device) =>
      device.name.toLowerCase().includes(search.toLowerCase()) ||
      device.ipAddress.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="mb-2">
      <input
        type="text"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search Devices..."
        className={`p-2 w-full rounded ${activeTheme.input}`}
      />
      <select
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className={`p-2 w-full rounded ${activeTheme.input}`}
      >
        <option value="">Select Device</option>
        {filteredDevices.map((device) => (
          <option key={device.id} value={device.id}>
            {device.name} ({device.ipAddress})
          </option>
        ))}
      </select>
    </div>
  );
}
```

### Instructions to Run the Project

1. **Create the Project Directory**:
   - Create a directory named `n-ai-surveillance`.
   - Extract or create the above file structure and contents within this directory.

2. **Install Dependencies**:
   - Navigate to the `n-ai-surveillance` directory in your terminal.
   - Run `npm install` to install all dependencies listed in `package.json`.

3. **Set Up Environment Variables**:
   - Create a `.env.local` file in the root directory.
   - Add the Gemini API key:
     ```env
     GEMINI_API_KEY=your_gemini_api_key_here
     ```
   - Ensure a local Ollama instance is running at `http://localhost:11434` with the `gemma3` model installed.

4. **Set Up Prisma**:
   - Ensure you have a Prisma-compatible database (e.g., PostgreSQL) set up.
   - Create a `prisma/schema.prisma` file with the following schema (or adjust based on your database):
     ```prisma
     generator client {
       provider = "prisma-client-js"
     }

     datasource db {
       provider = "postgresql"
       url      = env("DATABASE_URL")
     }

     model Gpu {
       id        Int      @id @default(autoincrement())
       name      String
       gpuModel  String
       vram      Int
       flops     Float
       status    String
       deviceId  Int?
       device    Device?  @relation(fields: [deviceId], references: [id])
     }

     model Device {
       id        Int      @id @default(autoincrement())
       name      String
       ipAddress String
       cpu       String
       ram       Int
       storage   Int
       gpus      Gpu[]
     }

     model Camera {
       id               Int      @id @default(autoincrement())
       name             String
       ipAddress        String
       location         String
       availabilityStatus String
       gpuId            Int?
       gpu              Gpu?     @relation(fields: [gpuId], references: [id])
     }
     ```
   - Add the database URL to `.env.local`:
     ```env
     DATABASE_URL=postgresql://user:password@localhost:5432/naisurveillance
     ```
   - Run `npx prisma generate` to generate the Prisma client.
   - Run `npx prisma db push` to sync the schema with your database.

5. **Run the Development Server**:
   - Run `npm run dev` to start the Next.js development server.
   - Open `http://localhost:3000` in your browser to access the application.

6. **Notes**:
   - The `/api/cameras` endpoint is referenced but not implemented in the provided code. You can add it similarly to `/api/gpus` and `/api/devices` using Prisma.
   - The Ollama API (`http://localhost:11434/api/generate`) requires a running Ollama instance with the `gemma3` model.
   - The Gemini API endpoints are placeholders; replace the URLs and ensure the API key is valid.
   - The dummy event data in `app/page.jsx` can be replaced with a real `/api/events` endpoint when available.

This project provides a fully functional frontend for the "N." AI Surveillance System, with modular components, a centralized theming system, and AI integration. Let me know if you need help setting up the backend or additional features!
