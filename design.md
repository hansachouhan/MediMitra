# MediMitra - Web Application Design Document

## Document Information

**Project Name:** MediMitra (Web Version)  
**Version:** 1.0  
**Date:** February 2026  
**Platform:** Progressive Web Application (PWA)  
**Status:** Hackathon MVP Design  
**Authors:** The real Akatsuki

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Technology Stack](#2-technology-stack)
3. [Frontend Architecture](#3-frontend-architecture)
4. [Backend Architecture](#4-backend-architecture)
5. [API Design](#5-api-design)
6. [UI/UX Design](#6-uiux-design)
7. [PWA Implementation](#7-pwa-implementation)
8. [Performance Optimization](#8-performance-optimization)
9. [SEO Strategy](#9-seo-strategy)
10. [Deployment Architecture](#10-deployment-architecture)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      User's Browser                          │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              React/Next.js Application                 │ │
│  │  ┌──────────────┬──────────────┬──────────────────┐   │ │
│  │  │   UI Layer   │  State Mgmt  │  Service Worker  │   │ │
│  │  │  (Components)│  (Zustand/   │  (Offline Cache) │   │ │
│  │  │              │   TanStack)  │                  │   │ │
│  │  └──────────────┴──────────────┴──────────────────┘   │ │
│  │                                                        │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │        Browser APIs                              │ │ │
│  │  │  • Web Speech API (Voice)                        │ │ │
│  │  │  • MediaDevices API (Camera)                     │ │ │
│  │  │  • localStorage/IndexedDB                        │ │ │
│  │  │  • Web Share API                                 │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────┘
                            │
                    HTTPS / REST API
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                   CDN (Cloudflare)                           │
│          (Static Assets, Image Optimization)                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│              Vercel Edge Network                             │
│         (Next.js App, Serverless Functions)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼────────┐  ┌──────▼────────────┐
│  OCR Service   │  │  NLP Service  │  │  Medicine Service │
│  (Cloud Run)   │  │  (Cloud Run)  │  │  (Cloud Run)      │
│                │  │               │  │                   │
│ - Google       │  │ - Speech API  │  │ - DB Queries      │
│   Vision API   │  │ - TTS         │  │ - Risk Analysis   │
│ - Preprocessing│  │ - Translation │  │ - Interactions    │
└────────────────┘  └───────────────┘  └───────────────────┘
                            │
                ┌───────────┼───────────┐
                │           │           │
        ┌───────▼──┐  ┌────▼─────┐  ┌─▼──────────┐
        │PostgreSQL│  │  Redis   │  │Cloud Storage│
        │  (Neon)  │  │(Upstash) │  │   (CDN)    │
        └──────────┘  └──────────┘  └────────────┘
```

### 1.2 Web-Specific Architecture Patterns

**Pattern 1: JAMstack Architecture**
- **J**avaScript: React/Next.js for dynamic functionality
- **A**PIs: Serverless functions + external APIs
- **M**arkup: Pre-rendered HTML for fast loads

**Pattern 2: Progressive Enhancement**
- Core content works without JavaScript
- Enhanced features with JavaScript enabled
- Offline capabilities with service workers

**Pattern 3: Edge-First Rendering**
- Static generation for content pages
- Edge functions for personalization
- CDN caching for global performance

**Pattern 4: Offline-First Data**
- Service worker caching strategy
- IndexedDB for large datasets
- Background sync for queued operations

---

## 2. Technology Stack

### 2.1 Frontend Stack

```yaml
Core Framework:
  - Next.js: 14.x (App Router)
  - React: 18.x
  - TypeScript: 5.x

Styling:
  - Tailwind CSS: 3.4.x
  - shadcn/ui: Latest (Radix UI components)
  - Framer Motion: 10.x (animations)

State Management:
  - Zustand: 4.x (global state)
  - TanStack Query: 5.x (server state)
  - React Hook Form: 7.x (form state)

PWA:
  - next-pwa: 5.6.x
  - Workbox: 7.x

Browser APIs:
  - Web Speech API (native)
  - MediaDevices API (native)
  - Web Share API (native)
  - IndexedDB (via Dexie.js)

Image Processing:
  - Tesseract.js: 5.x (offline OCR)
  - browser-image-compression: 2.x
  - react-image-crop: 11.x

Internationalization:
  - next-i18next: 15.x
  - i18next: 23.x

Analytics:
  - Vercel Analytics
  - Google Analytics 4
  - Sentry (error tracking)
```

### 2.2 Backend Stack

```yaml
API Framework:
  - FastAPI: 0.104.x (Python)
  - Uvicorn: 0.24.x (ASGI server)

Database:
  - PostgreSQL: 15.x (Neon serverless)
  - Redis: 7.x (Upstash serverless)

AI/ML Services:
  - Google Cloud Vision API (OCR)
  - Google Cloud Speech API (STT)
  - Google Cloud Text-to-Speech API

ORM & Migrations:
  - SQLAlchemy: 2.x
  - Alembic: 1.13.x

Authentication (Optional):
  - NextAuth.js: 4.x
  - JWT tokens

Background Jobs:
  - Celery: 5.3.x (if needed)
```

### 2.3 DevOps & Hosting

```yaml
Frontend Hosting:
  - Vercel (primary)
  - Cloudflare Pages (alternative)

Backend Hosting:
  - Google Cloud Run
  - Railway.app (alternative)

Database:
  - Neon (serverless Postgres)
  - Upstash (serverless Redis)

CDN:
  - Cloudflare (images, assets)
  - Vercel Edge Network

CI/CD:
  - GitHub Actions
  - Vercel Auto-deploy

Monitoring:
  - Vercel Analytics
  - Sentry (errors)
  - Uptime Robot (availability)
```

---

## 3. Frontend Architecture

### 3.1 Next.js Project Structure

```
MediMitra-web/
├── public/
│   ├── icons/                    # PWA icons
│   ├── images/                   # Static images
│   ├── manifest.json             # PWA manifest
│   └── sw.js                     # Service worker
│
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (marketing)/          # Marketing pages group
│   │   │   ├── page.tsx          # Homepage
│   │   │   ├── about/
│   │   │   └── privacy/
│   │   │
│   │   ├── (app)/                # App pages group
│   │   │   ├── scan/
│   │   │   │   └── page.tsx      # Scan medicine page
│   │   │   ├── cabinet/
│   │   │   │   └── page.tsx      # Medicine cabinet
│   │   │   ├── medicine/
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx  # Medicine details
│   │   │   └── search/
│   │   │       └── page.tsx      # Search medicines
│   │   │
│   │   ├── api/                  # API routes (serverless)
│   │   │   ├── ocr/
│   │   │   ├── medicine/
│   │   │   └── voice/
│   │   │
│   │   ├── layout.tsx            # Root layout
│   │   ├── error.tsx             # Error boundary
│   │   └── loading.tsx           # Loading UI
│   │
│   ├── components/               # React components
│   │   ├── ui/                   # shadcn/ui components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   └── ...
│   │   │
│   │   ├── features/             # Feature components
│   │   │   ├── scanner/
│   │   │   │   ├── CameraCapture.tsx
│   │   │   │   ├── ImagePreview.tsx
│   │   │   │   └── OCRResult.tsx
│   │   │   │
│   │   │   ├── medicine/
│   │   │   │   ├── MedicineCard.tsx
│   │   │   │   ├── RiskIndicator.tsx
│   │   │   │   └── InteractionChecker.tsx
│   │   │   │
│   │   │   ├── voice/
│   │   │   │   ├── VoiceInput.tsx
│   │   │   │   ├── VoiceOutput.tsx
│   │   │   │   └── ConversationHistory.tsx
│   │   │   │
│   │   │   └── cabinet/
│   │   │       ├── CabinetList.tsx
│   │   │       ├── ExpiryAlerts.tsx
│   │   │       └── AddMedicineDialog.tsx
│   │   │
│   │   ├── layout/               # Layout components
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── MobileNav.tsx
│   │   │
│   │   └── shared/               # Shared components
│   │       ├── LoadingSpinner.tsx
│   │       ├── ErrorMessage.tsx
│   │       ├── LanguageSelector.tsx
│   │       └── InstallPrompt.tsx
│   │
│   ├── lib/                      # Utility libraries
│   │   ├── api/                  # API clients
│   │   │   ├── ocr.ts
│   │   │   ├── medicine.ts
│   │   │   └── voice.ts
│   │   │
│   │   ├── hooks/                # Custom hooks
│   │   │   ├── useOCR.ts
│   │   │   ├── useVoice.ts
│   │   │   ├── useMedicineCabinet.ts
│   │   │   ├── useInstallPrompt.ts
│   │   │   └── useOnlineStatus.ts
│   │   │
│   │   ├── storage/              # Browser storage
│   │   │   ├── localStorage.ts
│   │   │   ├── indexedDB.ts
│   │   │   └── sessionStorage.ts
│   │   │
│   │   ├── utils/                # Helper functions
│   │   │   ├── image.ts          # Image processing
│   │   │   ├── validation.ts     # Form validation
│   │   │   ├── formatting.ts     # Data formatting
│   │   │   └── constants.ts      # App constants
│   │   │
│   │   └── services/             # Business logic
│   │       ├── ocr-service.ts
│   │       ├── medicine-service.ts
│   │       └── interaction-checker.ts
│   │
│   ├── store/                    # State management
│   │   ├── useCabinetStore.ts    # Zustand store
│   │   ├── useUserStore.ts
│   │   └── useUIStore.ts
│   │
│   ├── styles/                   # Global styles
│   │   └── globals.css
│   │
│   └── types/                    # TypeScript types
│       ├── medicine.ts
│       ├── api.ts
│       └── user.ts
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── next.config.js                # Next.js config
├── tailwind.config.ts            # Tailwind config
├── tsconfig.json                 # TypeScript config
└── package.json
```

### 3.2 Key Component Examples

#### Camera Capture Component

```typescript
// src/components/features/scanner/CameraCapture.tsx
'use client';

import { useRef, useState, useCallback } from 'react';
import { Camera, Upload, Image as ImageIcon } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { useOCR } from '@/lib/hooks/useOCR';
import { compressImage } from '@/lib/utils/image';

export function CameraCapture() {
  const videoRef = useRef<HTMLVideoElement>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);
  const [stream, setStream] = useState<MediaStream | null>(null);
  const [capturedImage, setCapturedImage] = useState<string | null>(null);
  const { processImage, isProcessing } = useOCR();

  const startCamera = useCallback(async () => {
    try {
      const mediaStream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'environment' }, // Rear camera on mobile
        audio: false
      });
      
      if (videoRef.current) {
        videoRef.current.srcObject = mediaStream;
        setStream(mediaStream);
      }
    } catch (error) {
      console.error('Camera access denied:', error);
      // Fallback to file upload
    }
  }, []);

  const capturePhoto = useCallback(() => {
    if (!videoRef.current) return;

    const canvas = document.createElement('canvas');
    canvas.width = videoRef.current.videoWidth;
    canvas.height = videoRef.current.videoHeight;
    
    const ctx = canvas.getContext('2d');
    if (!ctx) return;
    
    ctx.drawImage(videoRef.current, 0, 0);
    const imageData = canvas.toDataURL('image/jpeg', 0.9);
    
    setCapturedImage(imageData);
    stopCamera();
  }, []);

  const stopCamera = useCallback(() => {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      setStream(null);
    }
  }, [stream]);

  const handleFileUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    // Compress image before processing
    const compressed = await compressImage(file);
    const reader = new FileReader();
    
    reader.onload = (event) => {
      setCapturedImage(event.target?.result as string);
    };
    
    reader.readAsDataURL(compressed);
  };

  const handleProcess = async () => {
    if (!capturedImage) return;
    
    const result = await processImage(capturedImage);
    // Navigate to results page or show modal
  };

  return (
    <div className="space-y-4">
      {!capturedImage ? (
        <div className="relative aspect-video bg-gray-100 rounded-lg overflow-hidden">
          {stream ? (
            <>
              <video
                ref={videoRef}
                autoPlay
                playsInline
                className="w-full h-full object-cover"
              />
              <div className="absolute bottom-4 left-1/2 -translate-x-1/2">
                <Button
                  size="lg"
                  onClick={capturePhoto}
                  className="rounded-full w-16 h-16"
                >
                  <Camera className="w-6 h-6" />
                </Button>
              </div>
            </>
          ) : (
            <div className="flex flex-col items-center justify-center h-full gap-4">
              <Button onClick={startCamera} size="lg">
                <Camera className="mr-2" />
                Open Camera
              </Button>
              
              <span className="text-sm text-gray-500">or</span>
              
              <Button
                variant="outline"
                onClick={() => fileInputRef.current?.click()}
              >
                <Upload className="mr-2" />
                Upload Image
              </Button>
              
              <input
                ref={fileInputRef}
                type="file"
                accept="image/*"
                onChange={handleFileUpload}
                className="hidden"
              />
            </div>
          )}
        </div>
      ) : (
        <div className="space-y-4">
          <img
            src={capturedImage}
            alt="Captured medicine"
            className="w-full rounded-lg"
          />
          
          <div className="flex gap-2">
            <Button
              onClick={handleProcess}
              disabled={isProcessing}
              className="flex-1"
            >
              {isProcessing ? 'Processing...' : 'Analyze Medicine'}
            </Button>
            
            <Button
              variant="outline"
              onClick={() => setCapturedImage(null)}
            >
              Retake
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

#### Voice Assistant Component

```typescript
// src/components/features/voice/VoiceInput.tsx
'use client';

import { useState, useEffect } from 'react';
import { Mic, MicOff, Volume2 } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { useVoice } from '@/lib/hooks/useVoice';

export function VoiceInput() {
  const {
    isListening,
    transcript,
    response,
    startListening,
    stopListening,
    speak,
    browserSupportsVoice
  } = useVoice();

  if (!browserSupportsVoice) {
    return (
      <div className="p-4 bg-yellow-50 border border-yellow-200 rounded-lg">
        <p className="text-sm text-yellow-800">
          Voice input is not supported in your browser. Please use text input instead.
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="flex items-center gap-4">
        <Button
          size="lg"
          variant={isListening ? 'destructive' : 'default'}
          onClick={isListening ? stopListening : startListening}
          className="rounded-full w-16 h-16"
        >
          {isListening ? (
            <MicOff className="w-6 h-6 animate-pulse" />
          ) : (
            <Mic className="w-6 h-6" />
          )}
        </Button>
        
        <div className="flex-1">
          <p className="text-sm text-gray-500">
            {isListening ? 'Listening...' : 'Tap to speak'}
          </p>
          {transcript && (
            <p className="font-medium mt-1">{transcript}</p>
          )}
        </div>
      </div>

      {response && (
        <div className="p-4 bg-blue-50 rounded-lg space-y-2">
          <div className="flex items-start justify-between">
            <p className="flex-1">{response.text}</p>
            <Button
              size="sm"
              variant="ghost"
              onClick={() => speak(response.text)}
            >
              <Volume2 className="w-4 h-4" />
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

#### Custom Hook: useOCR

```typescript
// src/lib/hooks/useOCR.ts
import { useState } from 'react';
import { useMutation } from '@tanstack/react-query';
import { ocrApi } from '@/lib/api/ocr';
import type { OCRResult } from '@/types/api';

export function useOCR() {
  const [result, setResult] = useState<OCRResult | null>(null);

  const mutation = useMutation({
    mutationFn: async (imageData: string) => {
      // Convert base64 to blob
      const blob = await fetch(imageData).then(r => r.blob());
      
      // Call OCR API
      const response = await ocrApi.processImage(blob);
      return response;
    },
    onSuccess: (data) => {
      setResult(data);
    },
    onError: (error) => {
      console.error('OCR failed:', error);
    }
  });

  return {
    processImage: mutation.mutate,
    isProcessing: mutation.isPending,
    result,
    error: mutation.error
  };
}
```

#### Custom Hook: useVoice

```typescript
// src/lib/hooks/useVoice.ts
import { useState, useEffect, useCallback } from 'react';

// Check browser support
const browserSupportsVoice = 
  typeof window !== 'undefined' &&
  'SpeechRecognition' in window || 'webkitSpeechRecognition' in window;

const SpeechRecognition = browserSupportsVoice
  ? window.SpeechRecognition || window.webkitSpeechRecognition
  : null;

export function useVoice(language = 'hi-IN') {
  const [isListening, setIsListening] = useState(false);
  const [transcript, setTranscript] = useState('');
  const [response, setResponse] = useState<{ text: string; audio?: string } | null>(null);
  const [recognition, setRecognition] = useState<SpeechRecognition | null>(null);

  useEffect(() => {
    if (!SpeechRecognition) return;

    const recognitionInstance = new SpeechRecognition();
    recognitionInstance.continuous = false;
    recognitionInstance.interimResults = true;
    recognitionInstance.lang = language;

    recognitionInstance.onresult = (event) => {
      const current = event.resultIndex;
      const transcriptText = event.results[current][0].transcript;
      setTranscript(transcriptText);

      if (event.results[current].isFinal) {
        // Process the final transcript
        handleVoiceQuery(transcriptText);
      }
    };

    recognitionInstance.onend = () => {
      setIsListening(false);
    };

    setRecognition(recognitionInstance);

    return () => {
      recognitionInstance.stop();
    };
  }, [language]);

  const startListening = useCallback(() => {
    if (!recognition) return;
    
    setTranscript('');
    setResponse(null);
    recognition.start();
    setIsListening(true);
  }, [recognition]);

  const stopListening = useCallback(() => {
    if (!recognition) return;
    
    recognition.stop();
    setIsListening(false);
  }, [recognition]);

  const handleVoiceQuery = async (query: string) => {
    try {
      // Call NLP API to get response
      const result = await fetch('/api/voice/query', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query, language })
      });

      const data = await result.json();
      setResponse({ text: data.response });
      
      // Auto-play response
      speak(data.response);
    } catch (error) {
      console.error('Voice query failed:', error);
    }
  };

  const speak = useCallback((text: string) => {
    if (!('speechSynthesis' in window)) return;

    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = language;
    window.speechSynthesis.speak(utterance);
  }, [language]);

  return {
    isListening,
    transcript,
    response,
    startListening,
    stopListening,
    speak,
    browserSupportsVoice
  };
}
```

### 3.3 State Management with Zustand

```typescript
// src/store/useCabinetStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { Medicine, UserMedicine } from '@/types/medicine';

interface CabinetStore {
  medicines: UserMedicine[];
  addMedicine: (medicine: UserMedicine) => void;
  removeMedicine: (id: string) => void;
  updateMedicine: (id: string, updates: Partial<UserMedicine>) => void;
  getExpiringMedicines: (days: number) => UserMedicine[];
  clearCabinet: () => void;
}

export const useCabinetStore = create<CabinetStore>()(
  persist(
    (set, get) => ({
      medicines: [],

      addMedicine: (medicine) =>
        set((state) => ({
          medicines: [...state.medicines, medicine]
        })),

      removeMedicine: (id) =>
        set((state) => ({
          medicines: state.medicines.filter((m) => m.id !== id)
        })),

      updateMedicine: (id, updates) =>
        set((state) => ({
          medicines: state.medicines.map((m) =>
            m.id === id ? { ...m, ...updates } : m
          )
        })),

      getExpiringMedicines: (days) => {
        const targetDate = new Date();
        targetDate.setDate(targetDate.getDate() + days);

        return get().medicines.filter((m) => {
          const expiryDate = new Date(m.expiryDate);
          return expiryDate <= targetDate && expiryDate >= new Date();
        });
      },

      clearCabinet: () => set({ medicines: [] })
    }),
    {
      name: 'medicine-cabinet', // localStorage key
    }
  )
);
```

---

## 4. Backend Architecture

### 4.1 FastAPI Structure

```
backend/
├── app/
│   ├── main.py                   # FastAPI app entry
│   ├── config.py                 # Configuration
│   │
│   ├── api/                      # API routes
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── ocr.py            # OCR endpoints
│   │   │   ├── medicine.py       # Medicine endpoints
│   │   │   ├── voice.py          # Voice endpoints
│   │   │   └── health.py         # Health checks
│   │   │
│   │   └── dependencies.py       # Route dependencies
│   │
│   ├── core/                     # Core functionality
│   │   ├── security.py           # Auth, encryption
│   │   ├── config.py             # Settings
│   │   └── logging.py            # Logging setup
│   │
│   ├── db/                       # Database
│   │   ├── base.py               # Base class
│   │   ├── session.py            # DB sessions
│   │   └── models/
│   │       ├── medicine.py
│   │       ├── interaction.py
│   │       └── manufacturer.py
│   │
│   ├── schemas/                  # Pydantic models
│   │   ├── ocr.py
│   │   ├── medicine.py
│   │   └── voice.py
│   │
│   ├── services/                 # Business logic
│   │   ├── ocr_service.py
│   │   ├── medicine_service.py
│   │   ├── interaction_service.py
│   │   └── voice_service.py
│   │
│   └── utils/                    # Utilities
│       ├── image.py
│       ├── validation.py
│       └── cache.py
│
├── migrations/                   # Alembic migrations
├── tests/
├── requirements.txt
└── Dockerfile
```

### 4.2 FastAPI Main Application

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from app.api.v1 import ocr, medicine, voice
from app.core.config import settings

app = FastAPI(
    title="MediMitra API",
    description="AI-powered medicine safety platform API",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Gzip compression
app.add_middleware(GZipMiddleware, minimum_size=1000)

# Include routers
app.include_router(
    ocr.router,
    prefix="/api/v1/ocr",
    tags=["OCR"]
)

app.include_router(
    medicine.router,
    prefix="/api/v1/medicine",
    tags=["Medicine"]
)

app.include_router(
    voice.router,
    prefix="/api/v1/voice",
    tags=["Voice"]
)

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": "1.0.0"
    }

@app.on_event("startup")
async def startup_event():
    # Initialize database connection
    # Initialize cache
    # Load ML models
    pass

@app.on_event("shutdown")
async def shutdown_event():
    # Close database connections
    # Cleanup resources
    pass
```

### 4.3 Example API Route

```python
# app/api/v1/ocr.py
from fastapi import APIRouter, UploadFile, File, HTTPException
from fastapi.responses import JSONResponse
from app.services.ocr_service import OCRService
from app.schemas.ocr import OCRResponse
import logging

router = APIRouter()
ocr_service = OCRService()
logger = logging.getLogger(__name__)

@router.post("/scan", response_model=OCRResponse)
async def scan_medicine(
    file: UploadFile = File(...),
    language: str = "en"
):
    """
    Process medicine image and extract text using OCR.
    
    - **file**: Image file (JPEG, PNG, max 10MB)
    - **language**: Language code for OCR (default: en)
    """
    try:
        # Validate file type
        if file.content_type not in ["image/jpeg", "image/png", "image/webp"]:
            raise HTTPException(
                status_code=400,
                detail="Invalid file type. Only JPEG, PNG, WebP allowed."
            )
        
        # Read file content
        image_bytes = await file.read()
        
        # Validate file size (max 10MB)
        if len(image_bytes) > 10 * 1024 * 1024:
            raise HTTPException(
                status_code=413,
                detail="File size exceeds 10MB limit"
            )
        
        # Process image
        result = await ocr_service.process_image(
            image_bytes=image_bytes,
            language=language
        )
        
        logger.info(f"OCR processed successfully: {result.get('medicine_name')}")
        
        return JSONResponse(content=result)
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"OCR processing failed: {str(e)}")
        raise HTTPException(
            status_code=500,
            detail=f"OCR processing failed: {str(e)}"
        )
```

---

## 5. API Design

### 5.1 RESTful Endpoints

**Base URL:** `https://api.MediMitra.com/v1`

#### OCR Endpoints

```yaml
POST /api/v1/ocr/scan
Description: Process medicine image and extract text
Request:
  Content-Type: multipart/form-data
  Body:
    file: Image file
    language: string (optional)
Response: 200 OK
  {
    "ocr_id": "uuid",
    "full_text": "extracted text",
    "extracted_data": {
      "name": "Paracetamol",
      "composition": "Paracetamol 500mg",
      "mrp": 15.00,
      "batch_number": "XYZ123",
      "mfg_date": "01/2024",
      "exp_date": "12/2026"
    },
    "confidence": 0.95
  }
```

#### Medicine Endpoints

```yaml
GET /api/v1/medicine/search
Description: Search medicines by name
Parameters:
  q: string (query)
  limit: integer (default: 10)
Response: 200 OK
  {
    "results": [...],
    "total": 42
  }

GET /api/v1/medicine/{id}
Description: Get medicine details
Response: 200 OK
  {
    "id": 123,
    "name": "Paracetamol",
    "generic_name": "Acetaminophen",
    ...
  }

POST /api/v1/medicine/verify
Description: Verify medicine authenticity
Request:
  {
    "medicine_data": {
      "name": "Paracetamol",
      "mrp": 15.00,
      "batch_number": "XYZ123"
    }
  }
Response: 200 OK
  {
    "risk_assessment": {
      "overall_score": 25.0,
      "risk_level": "low",
      "checks": [...]
    }
  }

POST /api/v1/medicine/check-interactions
Description: Check drug interactions
Request:
  {
    "medicine_id": 123,
    "cabinet_medicine_ids": [45, 67]
  }
Response: 200 OK
  {
    "interactions": [...],
    "max_severity": "moderate"
  }
```

#### Voice Endpoints

```yaml
POST /api/v1/voice/query
Description: Process natural language query
Request:
  {
    "query": "Yeh dawai kab leni chahiye?",
    "language": "hi-IN"
  }
Response: 200 OK
  {
    "query_text": "Yeh dawai kab leni chahiye?",
    "response_text": "Yeh dawai din mein do baar...",
    "intent": "dosage_timing"
  }
```

### 5.2 Next.js API Routes (Serverless Functions)

```typescript
// app/api/ocr/scan/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const formData = await request.formData();
    const file = formData.get('file') as File;
    
    if (!file) {
      return NextResponse.json(
        { error: 'No file provided' },
        { status: 400 }
      );
    }
    
    // Forward to backend OCR service
    const backendFormData = new FormData();
    backendFormData.append('file', file);
    
    const response = await fetch(
      `${process.env.BACKEND_URL}/api/v1/ocr/scan`,
      {
        method: 'POST',
        body: backendFormData
      }
    );
    
    const data = await response.json();
    
    return NextResponse.json(data);
    
  } catch (error) {
    console.error('OCR API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export const config = {
  api: {
    bodyParser: false // Required for file uploads
  }
};
```

---

## 6. UI/UX Design

### 6.1 Design System

**Color Palette:**
```css
:root {
  /* Primary */
  --primary-50: #E3F2FD;
  --primary-500: #2196F3;
  --primary-700: #1976D2;
  
  /* Success (Low Risk) */
  --success-500: #4CAF50;
  
  /* Warning (Medium Risk) */
  --warning-500: #FF9800;
  
  /* Danger (High Risk) */
  --danger-500: #F44336;
  
  /* Neutrals */
  --gray-50: #FAFAFA;
  --gray-100: #F5F5F5;
  --gray-500: #9E9E9E;
  --gray-900: #212121;
}
```

**Typography (Tailwind):**
```javascript
// tailwind.config.ts
export default {
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter var', 'system-ui', 'sans-serif'],
        hindi: ['Noto Sans Devanagari', 'sans-serif']
      },
      fontSize: {
        'xs': '0.75rem',    // 12px
        'sm': '0.875rem',   // 14px
        'base': '1rem',     // 16px
        'lg': '1.125rem',   // 18px
        'xl': '1.25rem',    // 20px
        '2xl': '1.5rem',    // 24px
      }
    }
  }
}
```

### 6.2 Responsive Layouts

**Homepage (Desktop):**
```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <div className="min-h-screen">
      {/* Hero Section */}
      <section className="container mx-auto px-4 py-16 lg:py-24">
        <div className="grid lg:grid-cols-2 gap-12 items-center">
          <div>
            <h1 className="text-4xl lg:text-6xl font-bold mb-6">
              Verify Medicine Safety with AI
            </h1>
            <p className="text-xl text-gray-600 mb-8">
              Scan, verify, and understand your medicines in seconds.
              Available in 8+ Indian languages.
            </p>
            <div className="flex flex-col sm:flex-row gap-4">
              <Button size="lg" asChild>
                <Link href="/scan">
                  <Camera className="mr-2" />
                  Scan Medicine Now
                </Link>
              </Button>
              <Button size="lg" variant="outline" asChild>
                <Link href="/search">
                  Search Medicines
                </Link>
              </Button>
            </div>
          </div>
          
          <div className="relative">
            <Image
              src="/hero-image.webp"
              alt="Medicine verification"
              width={600}
              height={400}
              priority
              className="rounded-2xl shadow-2xl"
            />
          </div>
        </div>
      </section>

      {/* Features Section */}
      <section className="bg-gray-50 py-16">
        <div className="container mx-auto px-4">
          <h2 className="text-3xl font-bold text-center mb-12">
            How It Works
          </h2>
          
          <div className="grid md:grid-cols-3 gap-8">
            <FeatureCard
              icon={<Camera />}
              title="Scan Medicine"
              description="Take a photo of your medicine packaging"
            />
            <FeatureCard
              icon={<Shield />}
              title="Get Safety Report"
              description="AI analyzes and verifies authenticity"
            />
            <FeatureCard
              icon={<CheckCircle />}
              title="Stay Safe"
              description="Make informed decisions about your health"
            />
          </div>
        </div>
      </section>

      {/* Install PWA Prompt */}
      <InstallPrompt />
    </div>
  );
}
```

**Mobile-First Scan Page:**
```tsx
// app/scan/page.tsx
'use client';

export default function ScanPage() {
  return (
    <div className="min-h-screen bg-gray-50">
      <div className="container max-w-2xl mx-auto px-4 py-8">
        <h1 className="text-2xl font-bold mb-6">
          Scan Medicine
        </h1>
        
        {/* Camera Capture Component */}
        <CameraCapture />
        
        {/* Instructions */}
        <div className="mt-8 p-4 bg-blue-50 rounded-lg">
          <h3 className="font-semibold mb-2">Tips for best results:</h3>
          <ul className="space-y-1 text-sm text-gray-700">
            <li>• Ensure good lighting</li>
            <li>• Hold camera steady</li>
            <li>• Capture the medicine name clearly</li>
            <li>• Include batch number and MRP if visible</li>
          </ul>
        </div>
      </div>
    </div>
  );
}
```

### 6.3 Accessibility Implementation

```tsx
// Example: Accessible Button Component
import { ButtonHTMLAttributes, forwardRef } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary-500 text-white hover:bg-primary-600 focus-visible:ring-primary-500',
        outline: 'border border-gray-300 bg-white hover:bg-gray-50 focus-visible:ring-gray-500',
        ghost: 'hover:bg-gray-100 focus-visible:ring-gray-500'
      },
      size: {
        sm: 'h-9 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg'
      }
    },
    defaultVariants: {
      variant: 'default',
      size: 'md'
    }
  }
);

interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, isLoading, children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={buttonVariants({ variant, size, className })}
        disabled={disabled || isLoading}
        aria-busy={isLoading}
        {...props}
      >
        {isLoading && (
          <svg
            className="animate-spin -ml-1 mr-2 h-4 w-4"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            aria-hidden="true"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
          </svg>
        )}
        {children}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

---

## 7. PWA Implementation

### 7.1 Service Worker Strategy

```javascript
// next.config.js
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development',
  register: true,
  skipWaiting: true,
  runtimeCaching: [
    {
      urlPattern: /^https?.*/,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'offlineCache',
        expiration: {
          maxEntries: 200,
          maxAgeSeconds: 24 * 60 * 60 // 24 hours
        }
      }
    },
    {
      urlPattern: /\.(?:png|jpg|jpeg|svg|webp|gif)$/,
      handler: 'CacheFirst',
      options: {
        cacheName: 'image-cache',
        expiration: {
          maxEntries: 500,
          maxAgeSeconds: 30 * 24 * 60 * 60 // 30 days
        }
      }
    },
    {
      urlPattern: /^https:\/\/api\.MediMitra\.com\/api\/v1\/medicine\/.*/,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'medicine-api-cache',
        networkTimeoutSeconds: 10,
        expiration: {
          maxEntries: 100,
          maxAgeSeconds: 60 * 60 // 1 hour
        }
      }
    }
  ]
});

module.exports = withPWA({
  // Next.js config
  reactStrictMode: true,
  swcMinify: true
});
```

### 7.2 PWA Manifest

```json
// public/manifest.json
{
  "name": "MediMitra - Medicine Safety Companion",
  "short_name": "MediMitra",
  "description": "Verify medicine safety with AI-powered tools. Check authenticity, drug interactions, and safety information.",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2196F3",
  "orientation": "portrait-primary",
  "categories": ["health", "medical", "productivity"],
  "lang": "en-IN",
  "dir": "ltr",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/scan-mobile.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/results-mobile.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/cabinet-desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    }
  ],
  "shortcuts": [
    {
      "name": "Scan Medicine",
      "short_name": "Scan",
      "description": "Quickly scan a medicine",
      "url": "/scan",
      "icons": [{ "src": "/icons/scan-shortcut.png", "sizes": "96x96" }]
    },
    {
      "name": "My Cabinet",
      "short_name": "Cabinet",
      "description": "View your medicine cabinet",
      "url": "/cabinet",
      "icons": [{ "src": "/icons/cabinet-shortcut.png", "sizes": "96x96" }]
    }
  ],
  "share_target": {
    "action": "/scan",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "files": [
        {
          "name": "medicine_image",
          "accept": ["image/*"]
        }
      ]
    }
  }
}
```

### 7.3 Install Prompt Component

```tsx
// components/shared/InstallPrompt.tsx
'use client';

import { useState, useEffect } from 'react';
import { Download, X } from 'lucide-react';
import { Button } from '@/components/ui/button';

export function InstallPrompt() {
  const [deferredPrompt, setDeferredPrompt] = useState<any>(null);
  const [showPrompt, setShowPrompt] = useState(false);

  useEffect(() => {
    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e);
      
      // Show prompt after 30 seconds or on second visit
      setTimeout(() => {
        if (!localStorage.getItem('pwa-dismissed')) {
          setShowPrompt(true);
        }
      }, 30000);
    };

    window.addEventListener('beforeinstallprompt', handler);

    return () => {
      window.removeEventListener('beforeinstallprompt', handler);
    };
  }, []);

  const handleInstall = async () => {
    if (!deferredPrompt) return;

    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    
    if (outcome === 'accepted') {
      console.log('PWA installed');
    }
    
    setDeferredPrompt(null);
    setShowPrompt(false);
  };

  const handleDismiss = () => {
    setShowPrompt(false);
    localStorage.setItem('pwa-dismissed', 'true');
  };

  if (!showPrompt) return null;

  return (
    <div className="fixed bottom-4 left-4 right-4 md:left-auto md:right-4 md:w-96 bg-white rounded-lg shadow-2xl p-4 border border-gray-200 z-50 animate-slide-up">
      <button
        onClick={handleDismiss}
        className="absolute top-2 right-2 text-gray-400 hover:text-gray-600"
        aria-label="Dismiss"
      >
        <X className="w-4 h-4" />
      </button>

      <div className="flex items-start gap-4">
        <div className="flex-shrink-0 w-12 h-12 bg-primary-100 rounded-lg flex items-center justify-center">
          <Download className="w-6 h-6 text-primary-500" />
        </div>
        
        <div className="flex-1">
          <h3 className="font-semibold mb-1">Install MediMitra</h3>
          <p className="text-sm text-gray-600 mb-3">
            Install our app for faster access, offline features, and a better experience
          </p>
          
          <div className="flex gap-2">
            <Button size="sm" onClick={handleInstall}>
              Install App
            </Button>
            <Button size="sm" variant="outline" onClick={handleDismiss}>
              Not Now
            </Button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### 7.4 Offline Detection

```tsx
// components/shared/OfflineIndicator.tsx
'use client';

import { useEffect, useState } from 'react';
import { WifiOff } from 'lucide-react';

export function OfflineIndicator() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    setIsOnline(navigator.onLine);

    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  if (isOnline) return null;

  return (
    <div className="fixed top-0 left-0 right-0 bg-yellow-500 text-white px-4 py-2 text-center text-sm font-medium z-50">
      <WifiOff className="inline w-4 h-4 mr-2" />
      You're offline. Some features may be limited.
    </div>
  );
}
```

---

## 8. Performance Optimization

### 8.1 Image Optimization

```typescript
// lib/utils/image.ts
import imageCompression from 'browser-image-compression';

export async function compressImage(file: File): Promise<File> {
  const options = {
    maxSizeMB: 1,
    maxWidthOrHeight: 1920,
    useWebWorker: true,
    fileType: 'image/webp' as const
  };

  try {
    return await imageCompression(file, options);
  } catch (error) {
    console.error('Image compression failed:', error);
    return file; // Return original if compression fails
  }
}

export function getOptimizedImageUrl(
  src: string,
  width: number,
  quality: number = 85
): string {
  // Use Cloudflare Image Resizing or similar
  return `https://cdn.MediMitra.com/cdn-cgi/image/width=${width},quality=${quality},format=auto/${src}`;
}
```

**Next.js Image Component Usage:**
```tsx
import Image from 'next/image';

<Image
  src="/medicine-packaging.jpg"
  alt="Medicine packaging"
  width={800}
  height={600}
  quality={85}
  loading="lazy"
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

### 8.2 Code Splitting & Lazy Loading

```tsx
// Lazy load heavy components
import dynamic from 'next/dynamic';

const CameraCapture = dynamic(
  () => import('@/components/features/scanner/CameraCapture'),
  {
    loading: () => <LoadingSpinner />,
    ssr: false // Don't render on server (uses browser APIs)
  }
);

const VoiceAssistant = dynamic(
  () => import('@/components/features/voice/VoiceInput'),
  {
    loading: () => <p>Loading voice assistant...</p>,
    ssr: false
  }
);

// Route-based code splitting (automatic in Next.js App Router)
// Each page in app/ directory is automatically code-split
```

### 8.3 Caching Strategy

```typescript
// TanStack Query configuration
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 2,
      refetchOnWindowFocus: false,
      refetchOnReconnect: true
    }
  }
});

// Example usage
import { useQuery } from '@tanstack/react-query';

export function useMedicineInfo(medicineId: string) {
  return useQuery({
    queryKey: ['medicine', medicineId],
    queryFn: () => fetchMedicineInfo(medicineId),
    staleTime: 60 * 60 * 1000, // 1 hour
    enabled: !!medicineId
  });
}
```

### 8.4 Bundle Size Optimization

```javascript
// next.config.js
module.exports = {
  // Remove unused CSS
  experimental: {
    optimizeCss: true
  },
  
  // Analyze bundle size
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          // Vendor bundle
          vendor: {
            name: 'vendor',
            chunks: 'all',
            test: /node_modules/,
            priority: 20
          },
          // Common components
          common: {
            name: 'common',
            minChunks: 2,
            chunks: 'async',
            priority: 10,
            reuseExistingChunk: true,
            enforce: true
          }
        }
      };
    }
    return config;
  }
};
```

---

## 9. SEO Strategy

### 9.1 Meta Tags

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://MediMitra.com'),
  title: {
    default: 'MediMitra - Verify Medicine Safety with AI',
    template: '%s | MediMitra'
  },
  description: 'AI-powered medicine verification platform. Check medicine authenticity, drug interactions, and safety information in multiple Indian languages.',
  keywords: ['medicine verification', 'fake medicine detection', 'drug interaction checker', 'medicine safety', 'India healthcare'],
  authors: [{ name: 'MediMitra Team' }],
  creator: 'MediMitra',
  publisher: 'MediMitra',
  
  // Open Graph
  openGraph: {
    type: 'website',
    locale: 'en_IN',
    url: 'https://MediMitra.com',
    siteName: 'MediMitra',
    title: 'MediMitra - Medicine Safety Companion',
    description: 'Verify medicine safety with AI-powered tools',
    images: [
      {
        url: '/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'MediMitra'
      }
    ]
  },
  
  // Twitter
  twitter: {
    card: 'summary_large_image',
    site: '@MediMitra',
    creator: '@MediMitra',
    title: 'MediMitra - Medicine Safety Companion',
    description: 'Verify medicine safety with AI-powered tools',
    images: ['/twitter-card.jpg']
  },
  
  // Mobile
  viewport: {
    width: 'device-width',
    initialScale: 1,
    maximumScale: 5
  },
  
  // PWA
  manifest: '/manifest.json',
  themeColor: '#2196F3',
  
  // Icons
  icons: {
    icon: '/favicon.ico',
    apple: '/apple-touch-icon.png'
  },
  
  // Robots
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1
    }
  }
};
```

### 9.2 Structured Data (Schema.org)

```tsx
// app/page.tsx (homepage)
export default function HomePage() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'WebApplication',
    name: 'MediMitra',
    description: 'AI-powered medicine safety verification platform',
    url: 'https://MediMitra.com',
    applicationCategory: 'HealthApplication',
    operatingSystem: 'Web Browser',
    offers: {
      '@type': 'Offer',
      price: '0',
      priceCurrency: 'INR'
    },
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: '4.8',
      ratingCount: '1250'
    }
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* Rest of homepage */}
    </>
  );
}
```

### 9.3 Sitemap Generation

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://MediMitra.com',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1
    },
    {
      url: 'https://MediMitra.com/scan',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.9
    },
    {
      url: 'https://MediMitra.com/search',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8
    },
    {
      url: 'https://MediMitra.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.5
    }
    // Add medicine detail pages dynamically
  ];
}
```

---

## 10. Deployment Architecture

### 10.1 Hosting Strategy

```yaml
Frontend (Next.js):
  Platform: Vercel
  Region: Mumbai (asia-south1)
  Features:
    - Auto-deploy from Git
    - Edge Network (global CDN)
    - Serverless Functions
    - Preview deployments
    - Analytics

Backend (FastAPI):
  Platform: Google Cloud Run
  Region: Mumbai (asia-south1)
  Configuration:
    - Min instances: 1
    - Max instances: 10
    - Concurrency: 80
    - Memory: 2GB
    - CPU: 2 vCPU

Database:
  Platform: Neon (Serverless Postgres)
  Region: Mumbai
  Features:
    - Auto-scaling
    - Branching (for dev/staging)
    - Point-in-time recovery

Cache:
  Platform: Upstash (Serverless Redis)
  Region: Mumbai
  Features:
    - REST API
    - Global replication

CDN:
  Platform: Cloudflare
  Features:
    - Image optimization
    - Automatic compression
    - DDoS protection
    - SSL/TLS
```

### 10.2 Environment Configuration

```bash
# .env.local (Frontend - Next.js)
NEXT_PUBLIC_API_URL=https://api.MediMitra.com
NEXT_PUBLIC_APP_URL=https://MediMitra.com
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX

# Backend only (not exposed to browser)
DATABASE_URL=postgresql://user:pass@host:5432/dbname
REDIS_URL=redis://user:pass@host:6379
GOOGLE_CLOUD_PROJECT=MediMitra
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# .env.production (Backend - FastAPI)
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
GOOGLE_CLOUD_PROJECT=MediMitra
CORS_ORIGINS=https://MediMitra.com
SECRET_KEY=...
```

### 10.3 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
        with:
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}
      
      - name: Build and Deploy to Cloud Run
        run: |
          gcloud builds submit --tag gcr.io/${{ secrets.GCP_PROJECT_ID }}/api
          gcloud run deploy MediMitra-api \
            --image gcr.io/${{ secrets.GCP_PROJECT_ID }}/api \
            --platform managed \
            --region asia-south1 \
            --allow-unauthenticated
```

### 10.4 Performance Monitoring

```typescript
// lib/analytics.ts
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

// In app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```
