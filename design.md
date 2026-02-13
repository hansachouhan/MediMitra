# MediMitra - Design Document

## Document Information

**Project Name:** MediMitra  
**Version:** 1.0  
**Date:** February 2026  
**Status:** Hackathon MVP Design  
**Authors:** MediMitra Team

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Technology Stack](#2-technology-stack)
3. [Component Design](#3-component-design)
4. [Data Design](#4-data-design)
5. [API Design](#5-api-design)
6. [UI/UX Design](#6-uiux-design)
7. [Security Design](#7-security-design)
8. [Deployment Architecture](#8-deployment-architecture)
9. [Performance Optimization](#9-performance-optimization)
10. [Testing Strategy](#10-testing-strategy)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Mobile Application                       │
│  ┌──────────────┬──────────────┬──────────────────────────┐ │
│  │ Presentation │   Business   │      Data Access         │ │
│  │    Layer     │     Logic    │        Layer             │ │
│  │              │              │                          │ │
│  │  - UI        │  - OCR       │  - Local DB (SQLite)     │ │
│  │  - Voice I/O │  - NLP       │  - Cache (AsyncStorage)  │ │
│  │  - Camera    │  - Analysis  │  - API Client            │ │
│  └──────────────┴──────────────┴──────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────┘
                            │
                    HTTPS / REST API
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                      API Gateway                             │
│              (Rate Limiting, Auth, Routing)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼────────┐  ┌──────▼────────────┐
│  OCR Service   │  │  NLP Service  │  │  Medicine Service │
│                │  │               │  │                   │
│ - Image Proc   │  │ - Voice STT   │  │ - DB Queries      │
│ - Text Extract │  │ - NLU         │  │ - Risk Analysis   │
│ - ML Kit API   │  │ - TTS         │  │ - Interactions    │
└────────────────┘  └───────────────┘  └───────────────────┘
                            │
                ┌───────────┼───────────┐
                │           │           │
        ┌───────▼──┐  ┌────▼─────┐  ┌─▼──────────┐
        │ Database │  │  Cache   │  │   Storage  │
        │          │  │          │  │            │
        │PostgreSQL│  │  Redis   │  │  S3/GCS    │
        └──────────┘  └──────────┘  └────────────┘
```

### 1.2 Architecture Patterns

**Pattern 1: Offline-First Architecture**
- Local SQLite database for core medicine information
- Queue-based sync when network available
- Progressive Web App (PWA) capabilities for web version

**Pattern 2: Microservices (Simplified for MVP)**
- OCR Service: Image processing and text extraction
- NLP Service: Voice processing and language understanding
- Medicine Service: Business logic and database operations
- Each service independently scalable

**Pattern 3: Event-Driven Design**
- Asynchronous processing for non-critical tasks
- Message queue for image processing pipeline
- Event logging for analytics

**Pattern 4: Clean Architecture**
- Separation of concerns (UI, Business Logic, Data)
- Dependency inversion (abstractions over implementations)
- Testable components (unit, integration, E2E)

---

## 2. Technology Stack

### 2.1 Mobile Application

**Framework:** React Native (v0.72+)
- **Why:** Cross-platform (iOS + Android), large ecosystem, hot reload
- **Alternatives considered:** Flutter (rejected due to team expertise)

**Key Libraries:**
```javascript
// UI & Navigation
- react-navigation: Navigation stack
- react-native-paper: Material Design components
- react-native-vector-icons: Icon library

// Camera & Media
- react-native-vision-camera: High-performance camera
- react-native-image-picker: Image selection
- react-native-fs: File system access

// OCR & ML
- @react-native-ml-kit/text-recognition: On-device OCR
- tensorflow-lite-react-native: Custom ML models

// Voice
- @react-native-voice/voice: Speech recognition
- react-native-tts: Text-to-speech

// Storage
- react-native-sqlite-storage: Local database
- @react-native-async-storage/async-storage: Key-value storage

// Networking
- axios: HTTP client
- react-native-netinfo: Network status

// State Management
- redux-toolkit: Global state
- react-query: Server state & caching

// Localization
- react-i18next: Internationalization
- i18next: Translation framework
```

### 2.2 Backend Services

**Language:** Python 3.11
- **Why:** Rich ML/AI ecosystem, fast development

**Framework:** FastAPI
- **Why:** Async support, automatic API docs, type safety

**Key Libraries:**
```python
# Web Framework
fastapi==0.104.0
uvicorn==0.24.0  # ASGI server
pydantic==2.5.0  # Data validation

# Database
sqlalchemy==2.0.23  # ORM
alembic==1.13.0  # Migrations
psycopg2-binary==2.9.9  # PostgreSQL driver
redis==5.0.1  # Cache

# ML/AI
google-cloud-vision==3.5.0  # OCR API
google-cloud-speech==2.23.0  # STT
google-cloud-texttospeech==2.15.0  # TTS
transformers==4.35.0  # NLP models
spacy==3.7.2  # NLP processing
opencv-python==4.8.1  # Image processing
Pillow==10.1.0  # Image manipulation

# Utilities
python-dotenv==1.0.0  # Environment variables
python-jose==3.3.0  # JWT tokens
passlib==1.7.4  # Password hashing
httpx==0.25.2  # Async HTTP client
celery==5.3.4  # Task queue
```

### 2.3 Database

**Primary Database:** PostgreSQL 15
- **Why:** ACID compliance, JSON support, full-text search

**Cache Layer:** Redis 7
- **Why:** In-memory speed, pub/sub support, session storage

**Vector Database (Future):** Pinecone / Weaviate
- **For:** Semantic search of medicines

### 2.4 Cloud Services

**Platform:** Google Cloud Platform (GCP)
- **Compute:** Cloud Run (serverless containers)
- **Storage:** Cloud Storage (image storage)
- **Database:** Cloud SQL (managed PostgreSQL)
- **CDN:** Cloud CDN
- **APIs:** ML Kit, Speech-to-Text, Text-to-Speech

**Why GCP over AWS:**
- Superior AI/ML APIs (especially for Indian languages)
- Generous free tier for hackathon
- Cloud Run auto-scaling

### 2.5 DevOps & Tools

```yaml
Version Control: Git + GitHub
CI/CD: GitHub Actions
Containerization: Docker
Monitoring: 
  - Sentry (error tracking)
  - Google Cloud Monitoring
  - Mixpanel (product analytics)
API Testing: Postman / Insomnia
Documentation: Swagger UI (auto-generated from FastAPI)
```

---

## 3. Component Design

### 3.1 Mobile App Components

#### Component Hierarchy

```
App
├── Navigation
│   ├── AuthStack (Login/Register)
│   └── MainStack
│       ├── HomeScreen
│       ├── ScanScreen
│       ├── MedicineCabinetScreen
│       ├── MedicineDetailsScreen
│       └── SettingsScreen
│
├── Features
│   ├── Scanner
│   │   ├── CameraCapture
│   │   ├── ImagePreview
│   │   └── OCRProcessor
│   │
│   ├── MedicineInfo
│   │   ├── InfoCard
│   │   ├── InteractionChecker
│   │   └── RiskAssessment
│   │
│   ├── VoiceAssistant
│   │   ├── VoiceInput
│   │   ├── VoiceOutput
│   │   └── ConversationHistory
│   │
│   └── MedicineCabinet
│       ├── MedicineList
│       ├── ExpiryAlerts
│       └── ReminderSettings
│
├── Components (Shared)
│   ├── Button
│   ├── Card
│   ├── LoadingSpinner
│   ├── ErrorBoundary
│   └── LanguageSelector
│
└── Services
    ├── APIService
    ├── StorageService
    ├── OCRService
    └── VoiceService
```

#### Key Components Detail

**ScanScreen Component:**
```javascript
// ScanScreen.tsx
import React, { useState, useEffect } from 'react';
import { Camera } from 'react-native-vision-camera';
import { useOCR } from '../hooks/useOCR';

const ScanScreen = () => {
  const [hasPermission, setHasPermission] = useState(false);
  const { extractText, isProcessing } = useOCR();
  
  useEffect(() => {
    requestCameraPermission();
  }, []);
  
  const handleCapture = async (photo) => {
    const result = await extractText(photo.path);
    // Navigate to results screen
  };
  
  return (
    <SafeAreaView>
      <Camera 
        device={device}
        isActive={true}
        photo={true}
      />
      <CaptureButton onPress={handleCapture} />
    </SafeAreaView>
  );
};
```

**OCR Service:**
```javascript
// services/OCRService.ts
import TextRecognition from '@react-native-ml-kit/text-recognition';

class OCRService {
  async extractText(imagePath: string) {
    try {
      const result = await TextRecognition.recognize(imagePath);
      
      return {
        fullText: result.text,
        blocks: result.blocks.map(block => ({
          text: block.text,
          confidence: block.confidence,
          boundingBox: block.frame
        })),
        extractedData: this.parseText(result.text)
      };
    } catch (error) {
      throw new Error('OCR failed: ' + error.message);
    }
  }
  
  parseText(text: string) {
    // Extract medicine name, MRP, batch, expiry using regex
    const nameMatch = text.match(/^[A-Z][a-z]+(?:\s[A-Z][a-z]+)*/);
    const mrpMatch = text.match(/MRP[:\s]*₹?\s*(\d+\.?\d*)/i);
    const batchMatch = text.match(/Batch[:\s]*([A-Z0-9]+)/i);
    const expiryMatch = text.match(/Exp[:\s]*(\d{2}\/\d{4})/i);
    
    return {
      name: nameMatch?.[0],
      mrp: parseFloat(mrpMatch?.[1] || '0'),
      batchNumber: batchMatch?.[1],
      expiryDate: expiryMatch?.[1]
    };
  }
}

export default new OCRService();
```

**Redux Store Structure:**
```javascript
// store/slices/medicineSlice.ts
interface MedicineState {
  cabinet: Medicine[];
  currentMedicine: Medicine | null;
  scanHistory: ScanRecord[];
  isLoading: boolean;
  error: string | null;
}

const medicineSlice = createSlice({
  name: 'medicine',
  initialState,
  reducers: {
    addToCabinet: (state, action) => {
      state.cabinet.push(action.payload);
    },
    removeMedicine: (state, action) => {
      state.cabinet = state.cabinet.filter(m => m.id !== action.payload);
    },
    updateMedicine: (state, action) => {
      const index = state.cabinet.findIndex(m => m.id === action.payload.id);
      if (index !== -1) state.cabinet[index] = action.payload;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchMedicineInfo.pending, (state) => {
        state.isLoading = true;
      })
      .addCase(fetchMedicineInfo.fulfilled, (state, action) => {
        state.currentMedicine = action.payload;
        state.isLoading = false;
      })
      .addCase(fetchMedicineInfo.rejected, (state, action) => {
        state.error = action.error.message;
        state.isLoading = false;
      });
  }
});
```

### 3.2 Backend Components

#### Service Layer Architecture

```python
# app/main.py (FastAPI entry point)
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import ocr, medicine, voice, auth

app = FastAPI(title="MediMitra API", version="1.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(ocr.router, prefix="/api/v1/ocr", tags=["OCR"])
app.include_router(medicine.router, prefix="/api/v1/medicine", tags=["Medicine"])
app.include_router(voice.router, prefix="/api/v1/voice", tags=["Voice"])
app.include_router(auth.router, prefix="/api/v1/auth", tags=["Auth"])

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

**OCR Service (Backend):**
```python
# app/services/ocr_service.py
from google.cloud import vision
from PIL import Image
import re

class OCRService:
    def __init__(self):
        self.client = vision.ImageAnnotatorClient()
    
    async def process_image(self, image_bytes: bytes) -> dict:
        """Process image and extract text using Google Vision API"""
        image = vision.Image(content=image_bytes)
        response = self.client.text_detection(image=image)
        
        if response.error.message:
            raise Exception(f'OCR Error: {response.error.message}')
        
        texts = response.text_annotations
        full_text = texts[0].description if texts else ""
        
        return {
            'full_text': full_text,
            'extracted_data': self._parse_medicine_text(full_text),
            'confidence': self._calculate_confidence(texts)
        }
    
    def _parse_medicine_text(self, text: str) -> dict:
        """Extract structured data from OCR text"""
        return {
            'name': self._extract_medicine_name(text),
            'composition': self._extract_composition(text),
            'mrp': self._extract_mrp(text),
            'batch_number': self._extract_batch(text),
            'mfg_date': self._extract_date(text, 'Mfg'),
            'exp_date': self._extract_date(text, 'Exp'),
            'manufacturer': self._extract_manufacturer(text)
        }
    
    def _extract_mrp(self, text: str) -> float:
        """Extract MRP from text"""
        pattern = r'MRP[:\s]*₹?\s*(\d+\.?\d*)'
        match = re.search(pattern, text, re.IGNORECASE)
        return float(match.group(1)) if match else None
    
    def _extract_batch(self, text: str) -> str:
        """Extract batch number"""
        pattern = r'Batch[:\s]*([A-Z0-9]+)'
        match = re.search(pattern, text, re.IGNORECASE)
        return match.group(1) if match else None
    
    # ... other extraction methods
```

**Medicine Service:**
```python
# app/services/medicine_service.py
from sqlalchemy.orm import Session
from app.models import Medicine, DrugInteraction
from typing import List, Optional

class MedicineService:
    def __init__(self, db: Session):
        self.db = db
    
    async def get_medicine_info(self, name: str) -> Optional[dict]:
        """Fetch medicine information from database"""
        medicine = self.db.query(Medicine).filter(
            Medicine.name.ilike(f'%{name}%')
        ).first()
        
        if not medicine:
            # Fallback to external API
            medicine = await self._fetch_from_external_api(name)
        
        return self._serialize_medicine(medicine)
    
    async def check_risk_factors(self, medicine_data: dict) -> dict:
        """Analyze medicine for risk factors"""
        risks = {
            'price_risk': self._check_price_risk(medicine_data),
            'batch_risk': self._check_batch_format(medicine_data),
            'expiry_risk': self._check_expiry(medicine_data),
            'overall_score': 0
        }
        
        # Calculate overall risk score (0-100, lower is better)
        risk_weights = {'price_risk': 0.4, 'batch_risk': 0.3, 'expiry_risk': 0.3}
        risks['overall_score'] = sum(
            risks[k] * v for k, v in risk_weights.items()
        )
        
        return risks
    
    def _check_price_risk(self, data: dict) -> float:
        """Check if MRP is suspiciously low"""
        expected_price = self._get_expected_price(data['name'])
        if not expected_price:
            return 0.0
        
        actual_price = data.get('mrp', 0)
        deviation = (expected_price - actual_price) / expected_price
        
        if deviation > 0.3:  # 30% below expected
            return 80.0  # High risk
        elif deviation > 0.15:
            return 50.0  # Medium risk
        return 10.0  # Low risk
    
    async def check_interactions(
        self, 
        medicine_id: int, 
        cabinet_medicine_ids: List[int]
    ) -> List[dict]:
        """Check for drug interactions"""
        interactions = []
        
        for cab_med_id in cabinet_medicine_ids:
            interaction = self.db.query(DrugInteraction).filter(
                (DrugInteraction.drug_a_id == medicine_id) &
                (DrugInteraction.drug_b_id == cab_med_id) |
                (DrugInteraction.drug_a_id == cab_med_id) &
                (DrugInteraction.drug_b_id == medicine_id)
            ).first()
            
            if interaction:
                interactions.append({
                    'severity': interaction.severity,
                    'description': interaction.description,
                    'recommendation': interaction.recommendation
                })
        
        return interactions
```

**Voice Service:**
```python
# app/services/voice_service.py
from google.cloud import speech_v1, texttospeech_v1
import io

class VoiceService:
    def __init__(self):
        self.stt_client = speech_v1.SpeechClient()
        self.tts_client = texttospeech_v1.TextToSpeechClient()
    
    async def speech_to_text(
        self, 
        audio_bytes: bytes, 
        language_code: str = 'hi-IN'
    ) -> str:
        """Convert speech to text"""
        audio = speech_v1.RecognitionAudio(content=audio_bytes)
        config = speech_v1.RecognitionConfig(
            encoding=speech_v1.RecognitionConfig.AudioEncoding.LINEAR16,
            sample_rate_hertz=16000,
            language_code=language_code,
            enable_automatic_punctuation=True
        )
        
        response = self.stt_client.recognize(config=config, audio=audio)
        
        if not response.results:
            return ""
        
        return response.results[0].alternatives[0].transcript
    
    async def text_to_speech(
        self, 
        text: str, 
        language_code: str = 'hi-IN',
        voice_name: str = 'hi-IN-Wavenet-A'
    ) -> bytes:
        """Convert text to speech"""
        synthesis_input = texttospeech_v1.SynthesisInput(text=text)
        
        voice = texttospeech_v1.VoiceSelectionParams(
            language_code=language_code,
            name=voice_name
        )
        
        audio_config = texttospeech_v1.AudioConfig(
            audio_encoding=texttospeech_v1.AudioEncoding.MP3
        )
        
        response = self.tts_client.synthesize_speech(
            input=synthesis_input,
            voice=voice,
            audio_config=audio_config
        )
        
        return response.audio_content
    
    async def process_voice_query(
        self, 
        audio_bytes: bytes, 
        language: str
    ) -> dict:
        """Process voice query end-to-end"""
        # 1. Convert speech to text
        query_text = await self.speech_to_text(audio_bytes, language)
        
        # 2. Process query with NLP
        intent, entities = await self._understand_intent(query_text, language)
        
        # 3. Generate response
        response_text = await self._generate_response(intent, entities, language)
        
        # 4. Convert response to speech
        audio_response = await self.text_to_speech(response_text, language)
        
        return {
            'query_text': query_text,
            'response_text': response_text,
            'audio_response': audio_response,
            'intent': intent
        }
```

---

## 4. Data Design

### 4.1 Database Schema

**Entity-Relationship Diagram:**

```
┌─────────────────┐         ┌──────────────────┐
│    Medicine     │         │   Manufacturer   │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │────┐    │ id (PK)          │
│ name            │    │    │ name             │
│ generic_name    │    └───▶│ country          │
│ composition     │         │ website          │
│ therapeutic_cls │         └──────────────────┘
│ manufacturer_id │
│ nlem_listed     │         ┌──────────────────┐
│ schedule        │         │  MedicineBatch   │
│ created_at      │         ├──────────────────┤
└─────────────────┘    ┌───▶│ id (PK)          │
                       │    │ medicine_id (FK) │
                       │    │ batch_number     │
                       │    │ mfg_date         │
                       │    │ exp_date         │
                       └────│ mrp              │
                            │ storage_temp     │
                            └──────────────────┘

┌─────────────────┐         ┌──────────────────┐
│      User       │         │  UserMedicine    │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │    ┌───▶│ id (PK)          │
│ email           │    │    │ user_id (FK)     │
│ phone_number    │    │    │ medicine_id (FK) │
│ preferred_lang  │────┘    │ quantity         │
│ created_at      │         │ purchase_date    │
└─────────────────┘         │ expiry_date      │
                            │ reminder_enabled │
                            │ dosage_schedule  │
                            └──────────────────┘

┌─────────────────┐         ┌──────────────────┐
│DrugInteraction  │         │   ScanHistory    │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │         │ id (PK)          │
│ drug_a_id (FK)  │         │ user_id (FK)     │
│ drug_b_id (FK)  │         │ medicine_id (FK) │
│ severity        │         │ image_url        │
│ description     │         │ ocr_result       │
│ mechanism       │         │ risk_score       │
│ recommendation  │         │ scanned_at       │
└─────────────────┘         └──────────────────┘
```

### 4.2 SQL Table Definitions

```sql
-- Medicine table
CREATE TABLE medicines (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    generic_name VARCHAR(255),
    composition TEXT,
    therapeutic_class VARCHAR(100),
    manufacturer_id INTEGER REFERENCES manufacturers(id),
    nlem_listed BOOLEAN DEFAULT FALSE,
    schedule VARCHAR(20), -- Schedule H, H1, X, etc.
    typical_mrp_min DECIMAL(10,2),
    typical_mrp_max DECIMAL(10,2),
    description TEXT,
    indications TEXT,
    side_effects TEXT,
    dosage_info TEXT,
    storage_instructions TEXT,
    prescription_required BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_medicine_name ON medicines(name);
CREATE INDEX idx_medicine_generic ON medicines(generic_name);
CREATE FULLTEXT INDEX idx_medicine_search ON medicines(name, generic_name, composition);

-- Medicine batches
CREATE TABLE medicine_batches (
    id SERIAL PRIMARY KEY,
    medicine_id INTEGER REFERENCES medicines(id) ON DELETE CASCADE,
    batch_number VARCHAR(50) NOT NULL,
    mfg_date DATE,
    exp_date DATE,
    mrp DECIMAL(10,2),
    storage_temp VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(medicine_id, batch_number)
);

-- Users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    phone_number VARCHAR(20) UNIQUE,
    preferred_language VARCHAR(10) DEFAULT 'en',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

-- User medicine cabinet
CREATE TABLE user_medicines (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    medicine_id INTEGER REFERENCES medicines(id),
    quantity INTEGER DEFAULT 1,
    purchase_date DATE,
    expiry_date DATE,
    reminder_enabled BOOLEAN DEFAULT FALSE,
    dosage_schedule JSONB, -- {"morning": 1, "evening": 1}
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Drug interactions
CREATE TABLE drug_interactions (
    id SERIAL PRIMARY KEY,
    drug_a_id INTEGER REFERENCES medicines(id),
    drug_b_id INTEGER REFERENCES medicines(id),
    severity VARCHAR(20) CHECK (severity IN ('severe', 'moderate', 'minor')),
    description TEXT,
    mechanism TEXT,
    recommendation TEXT,
    source VARCHAR(100), -- Source of interaction data
    CHECK (drug_a_id < drug_b_id), -- Ensure no duplicates
    UNIQUE(drug_a_id, drug_b_id)
);

-- Scan history
CREATE TABLE scan_history (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    medicine_id INTEGER REFERENCES medicines(id),
    image_url TEXT,
    ocr_result JSONB,
    risk_assessment JSONB,
    scanned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Manufacturers
CREATE TABLE manufacturers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    country VARCHAR(100),
    website VARCHAR(255),
    verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4.3 NoSQL Data Models (Redis Cache)

```python
# Cache keys structure

# Medicine info cache (TTL: 24 hours)
CACHE_KEY_MEDICINE = "medicine:{medicine_id}"
# Value: JSON of medicine details

# User session cache (TTL: 7 days)
CACHE_KEY_SESSION = "session:{user_id}"
# Value: JWT token, user preferences

# Recent scans cache (TTL: 1 hour)
CACHE_KEY_RECENT_SCANS = "scans:recent:{user_id}"
# Value: List of scan IDs

# Popular medicines cache (TTL: 12 hours)
CACHE_KEY_POPULAR = "medicines:popular"
# Value: Sorted set of medicine IDs by scan count

# Price comparison cache (TTL: 6 hours)
CACHE_KEY_PRICE = "price:{medicine_id}"
# Value: JSON of price data from multiple sources
```

### 4.4 Data Migration Strategy

```python
# alembic/versions/001_initial_schema.py
from alembic import op
import sqlalchemy as sa

def upgrade():
    # Create tables in correct order (respecting foreign keys)
    op.create_table(
        'manufacturers',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(255), nullable=False),
        sa.Column('country', sa.String(100)),
        sa.Column('website', sa.String(255)),
        sa.Column('verified', sa.Boolean(), default=False),
        sa.Column('created_at', sa.TIMESTAMP(), server_default=sa.func.now()),
        sa.PrimaryKeyConstraint('id')
    )
    
    op.create_table(
        'medicines',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(255), nullable=False),
        # ... other columns
        sa.Column('manufacturer_id', sa.Integer()),
        sa.ForeignKeyConstraint(['manufacturer_id'], ['manufacturers.id']),
        sa.PrimaryKeyConstraint('id')
    )
    
    # Create indexes
    op.create_index('idx_medicine_name', 'medicines', ['name'])
    # ... other indexes

def downgrade():
    op.drop_table('medicines')
    op.drop_table('manufacturers')
```

---

## 5. API Design

### 5.1 API Endpoint Structure

**Base URL:** `https://api.MediMitra.com/v1`

**Authentication:** Bearer token (JWT) in Authorization header

### 5.2 Endpoint Specifications

#### OCR Endpoints

**POST /api/v1/ocr/scan**
```yaml
Summary: Process medicine image and extract text
Request:
  Content-Type: multipart/form-data
  Body:
    image: File (JPEG/PNG, max 10MB)
    language: String (optional, default: 'en')
Response:
  200 OK:
    {
      "ocr_id": "uuid",
      "full_text": "Paracetamol 500mg...",
      "extracted_data": {
        "name": "Paracetamol",
        "composition": "Paracetamol 500mg",
        "mrp": 15.00,
        "batch_number": "XYZ123",
        "mfg_date": "01/2024",
        "exp_date": "12/2026"
      },
      "confidence": 0.95,
      "processing_time_ms": 1234
    }
  400 Bad Request:
    {
      "error": "Invalid image format",
      "code": "INVALID_IMAGE"
    }
  413 Payload Too Large:
    {
      "error": "Image size exceeds 10MB limit"
    }
```

#### Medicine Endpoints

**GET /api/v1/medicine/search**
```yaml
Summary: Search for medicine by name
Parameters:
  q: String (query, required)
  limit: Integer (default: 10, max: 50)
  language: String (default: 'en')
Response:
  200 OK:
    {
      "results": [
        {
          "id": 123,
          "name": "Paracetamol",
          "generic_name": "Acetaminophen",
          "manufacturer": "Sun Pharma",
          "typical_mrp": "15-20",
          "nlem_listed": true
        }
      ],
      "total": 42,
      "page": 1
    }
```

**GET /api/v1/medicine/{id}**
```yaml
Summary: Get detailed medicine information
Parameters:
  id: Integer (path parameter)
  language: String (query, default: 'en')
Response:
  200 OK:
    {
      "id": 123,
      "name": "Paracetamol",
      "generic_name": "Acetaminophen",
      "composition": "Paracetamol 500mg",
      "therapeutic_class": "Analgesic",
      "indications": "Fever, pain relief",
      "side_effects": ["Nausea", "Allergic reactions"],
      "dosage": {
        "adult": "1-2 tablets every 4-6 hours",
        "child": "Consult physician"
      },
      "storage": "Store below 25°C",
      "prescription_required": false,
      "nlem_listed": true,
      "manufacturer": {
        "name": "Sun Pharma",
        "country": "India"
      }
    }
  404 Not Found:
    {
      "error": "Medicine not found",
      "code": "MEDICINE_NOT_FOUND"
    }
```

**POST /api/v1/medicine/verify**
```yaml
Summary: Verify medicine authenticity and check risk factors
Request:
  {
    "medicine_data": {
      "name": "Paracetamol",
      "mrp": 12.00,
      "batch_number": "XYZ123",
      "exp_date": "12/2026"
    }
  }
Response:
  200 OK:
    {
      "risk_assessment": {
        "overall_score": 25.0,
        "risk_level": "low",
        "checks": [
          {
            "type": "price_verification",
            "status": "pass",
            "details": "Price within expected range (₹15-20)",
            "risk_score": 10.0
          },
          {
            "type": "batch_format",
            "status": "warning",
            "details": "Batch format could not be verified",
            "risk_score": 40.0
          },
          {
            "type": "expiry_check",
            "status": "pass",
            "details": "Valid until Dec 2026",
            "risk_score": 0.0
          }
        ],
        "recommendations": [
          "Verify batch number with manufacturer",
          "Check for hologram sticker on packaging"
        ]
      },
      "verified_at": "2026-02-13T10:30:00Z"
    }
```

**POST /api/v1/medicine/check-interactions**
```yaml
Summary: Check for drug interactions
Request:
  {
    "medicine_id": 123,
    "cabinet_medicine_ids": [45, 67, 89]
  }
Response:
  200 OK:
    {
      "interactions": [
        {
          "interacting_medicine": {
            "id": 45,
            "name": "Aspirin"
          },
          "severity": "moderate",
          "description": "May increase risk of stomach bleeding",
          "mechanism": "Both drugs inhibit prostaglandin synthesis",
          "recommendation": "Take at least 2 hours apart. Consult doctor if experiencing stomach pain."
        }
      ],
      "total_interactions": 1,
      "max_severity": "moderate"
    }
```

#### Voice Endpoints

**POST /api/v1/voice/query**
```yaml
Summary: Process voice query (speech-to-text + NLP + text-to-speech)
Request:
  Content-Type: multipart/form-data
  Body:
    audio: File (WAV/MP3, max 5MB)
    language: String (default: 'hi-IN')
Response:
  200 OK:
    {
      "query_text": "Yeh dawai kab leni chahiye?",
      "intent": "dosage_timing",
      "response_text": "Yeh dawai din mein do baar, khane ke baad leni chahiye",
      "audio_url": "https://cdn.MediMitra.com/audio/xyz.mp3",
      "processing_time_ms": 2500
    }
```

#### User Cabinet Endpoints

**GET /api/v1/cabinet**
```yaml
Summary: Get user's medicine cabinet
Headers:
  Authorization: Bearer {token}
Response:
  200 OK:
    {
      "medicines": [
        {
          "id": 1,
          "medicine": {
            "id": 123,
            "name": "Paracetamol",
            "image_url": "..."
          },
          "quantity": 10,
          "expiry_date": "2026-12-31",
          "purchase_date": "2026-01-15",
          "reminder_enabled": true,
          "dosage_schedule": {
            "morning": 1,
            "evening": 1
          }
        }
      ],
      "expiring_soon": 2,
      "expired": 0
    }
```

**POST /api/v1/cabinet**
```yaml
Summary: Add medicine to cabinet
Request:
  {
    "medicine_id": 123,
    "quantity": 10,
    "purchase_date": "2026-02-13",
    "expiry_date": "2026-12-31",
    "reminder_enabled": true,
    "dosage_schedule": {
      "morning": 1,
      "evening": 1
    }
  }
Response:
  201 Created:
    {
      "id": 1,
      "medicine_id": 123,
      "created_at": "2026-02-13T10:00:00Z"
    }
```

### 5.3 Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "mrp",
        "message": "Must be a positive number"
      }
    ],
    "timestamp": "2026-02-13T10:00:00Z",
    "request_id": "req_123456"
  }
}
```

### 5.4 Rate Limiting

```yaml
Public Endpoints:
  - 100 requests per minute per IP
  
Authenticated Endpoints:
  Free Tier:
    - 1000 requests per day
    - 50 requests per minute
  
  Premium Tier:
    - Unlimited requests
    - 200 requests per minute

Headers:
  X-RateLimit-Limit: 50
  X-RateLimit-Remaining: 42
  X-RateLimit-Reset: 1708430400
```

---

## 6. UI/UX Design

### 6.1 Design Principles

1. **Simplicity First:** Minimize cognitive load, especially for elderly users
2. **Voice-First for Accessibility:** Voice should be as prominent as visual UI
3. **Regional Language Priority:** Hindi/regional languages as default, not English
4. **Offline Resilience:** Clear indicators when offline, graceful degradation
5. **Trust Through Transparency:** Show confidence scores, not absolute claims
6. **Scannable Information:** Use icons, colors, and card-based layouts

### 6.2 Color Scheme

```
Primary Colors:
  - Medicine Blue: #2196F3 (Trust, healthcare)
  - Safety Green: #4CAF50 (Low risk, verified)
  - Warning Orange: #FF9800 (Medium risk)
  - Alert Red: #F44336 (High risk, expired)

Neutral Colors:
  - Background: #FAFAFA
  - Surface: #FFFFFF
  - Text Primary: #212121
  - Text Secondary: #757575

Accessibility:
  - All color combinations meet WCAG AA contrast ratio (4.5:1)
  - Color is never the only indicator (use icons + text)
```

### 6.3 Typography

```
Font Family: Inter (Latin), Noto Sans Devanagari (Hindi)

Font Sizes:
  - Heading 1: 24sp (Bold)
  - Heading 2: 20sp (Semi-bold)
  - Heading 3: 18sp (Medium)
  - Body: 16sp (Regular)
  - Caption: 14sp (Regular)
  - Small: 12sp (Regular)

Line Height: 1.5x font size
Letter Spacing: -0.01em for headings

Adjustable: Users can increase font size up to 200% (accessibility)
```

### 6.4 Screen Designs (Wireframes)

#### Home Screen

```
╔══════════════════════════════════╗
║ 🏥 MediMitra        🔔 ⚙️    ║
╠══════════════════════════════════╣
║                                  ║
║   ┌────────────────────────┐    ║
║   │    📷 Scan Medicine    │    ║
║   └────────────────────────┘    ║
║                                  ║
║   ┌────────────────────────┐    ║
║   │   🎤 Ask About Medicine│    ║
║   └────────────────────────┘    ║
║                                  ║
║   My Medicine Cabinet            ║
║   ┌──────┬──────┬──────┐        ║
║   │ Para │ Aspi │ Diab │  +     ║
║   │ cetl │ rin  │ etic │        ║
║   └──────┴──────┴──────┘        ║
║                                  ║
║   ⚠️ 2 medicines expiring soon  ║
║                                  ║
╚══════════════════════════════════╝
```

#### Scan Result Screen

```
╔══════════════════════════════════╗
║ ← Medicine Details               ║
╠══════════════════════════════════╣
║  [Medicine Image Preview]        ║
║                                  ║
║  Paracetamol 500mg               ║
║  Sun Pharma                      ║
║                                  ║
║  Safety Check: ✅ Low Risk       ║
║  ┌────────────────────────────┐ ║
║  │ Price Verified      ✅ Pass│ ║
║  │ Batch Format        ⚠️ Warn│ ║
║  │ Expiry Valid        ✅ Pass│ ║
║  └────────────────────────────┘ ║
║                                  ║
║  What is this medicine for?      ║
║  Used for fever, pain relief...  ║
║                                  ║
║  📋 View Full Details            ║
║  ➕ Add to Cabinet               ║
║                                  ║
╚══════════════════════════════════╝
```

#### Voice Assistant Screen

```
╔══════════════════════════════════╗
║ ← Voice Assistant                ║
╠══════════════════════════════════╣
║                                  ║
║       🎤                          ║
║    [Listening...]                ║
║                                  ║
║  You asked:                      ║
║  "Yeh dawai kab leni chahiye?"   ║
║                                  ║
║  Response:                       ║
║  यह दवाई दिन में दो बार,        ║
║  खाने के बाद लेनी चाहिए।        ║
║                                  ║
║  [🔊 Play Audio]                 ║
║                                  ║
║  ┌────────────────────────────┐ ║
║  │ Ask Another Question   🎤  │ ║
║  └────────────────────────────┘ ║
║                                  ║
╚══════════════════════════════════╝
```

### 6.5 Component Library

**Button Component:**
```javascript
<Button
  variant="primary" | "secondary" | "outline"
  size="small" | "medium" | "large"
  icon={IconComponent}
  loading={boolean}
  disabled={boolean}
>
  Button Text
</Button>
```

**Medicine Card Component:**
```javascript
<MedicineCard
  name="Paracetamol"
  manufacturer="Sun Pharma"
  expiryDate="2026-12-31"
  quantity={10}
  riskLevel="low" | "medium" | "high"
  onPress={() => {}}
  onRemove={() => {}}
/>
```

**Risk Indicator Component:**
```javascript
<RiskIndicator
  score={25.0}
  checks={[
    { type: 'price', status: 'pass', score: 10 },
    { type: 'batch', status: 'warn', score: 40 },
    { type: 'expiry', status: 'pass', score: 0 }
  ]}
/>
```

### 6.6 User Flow Diagrams

**Primary Flow: Scanning a Medicine**

```
Start
  ↓
Open App
  ↓
Tap "Scan Medicine"
  ↓
Grant Camera Permission?
  ├─ Yes → Open Camera
  └─ No → Show Permission Explanation
             ↓
           Retry Permission
  ↓
Point at Medicine Strip
  ↓
Auto-Capture (or Manual Tap)
  ↓
Show Loading (OCR Processing)
  ↓
Display Results:
  - Medicine Info
  - Safety Check
  - Risk Indicators
  ↓
User Actions:
  ├─ View Full Details
  ├─ Add to Cabinet
  ├─ Share Result
  └─ Scan Another
```

**Secondary Flow: Voice Query**

```
From Any Screen
  ↓
Tap Voice Icon
  ↓
Grant Microphone Permission?
  ├─ Yes → Start Recording
  └─ No → Show Permission Explanation
  ↓
Speak Question
  ↓
Tap to Stop (or Auto-Stop)
  ↓
Show Transcription
  ↓
Processing (NLP)
  ↓
Display Text Response
  ↓
Play Audio Response (Auto)
  ↓
User Actions:
  ├─ Ask Follow-up
  ├─ Save to History
  └─ Close Assistant
```

---

## 7. Security Design

### 7.1 Authentication & Authorization

**Strategy:** JWT-based stateless authentication

```python
# JWT Token Structure
{
  "sub": "user_id_123",  # Subject (user ID)
  "email": "user@example.com",
  "role": "user",  # user | premium | admin
  "iat": 1708430400,  # Issued at
  "exp": 1708516800,  # Expiration (24 hours)
  "jti": "token_id_xyz"  # JWT ID (for revocation)
}
```

**Token Flow:**
1. User logs in with email/phone + OTP
2. Server generates JWT with 24-hour expiry
3. Client stores JWT in secure storage (Keychain/KeyStore)
4. Client includes JWT in `Authorization: Bearer <token>` header
5. Server validates JWT signature and expiry on each request
6. Refresh token used to obtain new JWT after expiry

**Security Measures:**
- Tokens signed with RS256 (asymmetric encryption)
- Short-lived access tokens (24 hours)
- Refresh tokens (30 days, stored securely)
- Token revocation list (Redis) for logout
- Rate limiting on login endpoint (5 attempts per 15 minutes)

### 7.2 Data Encryption

**At Rest:**
- Database: PostgreSQL with Transparent Data Encryption (TDE)
- File Storage: AES-256 encryption for stored images
- Local Mobile Storage: AES-256 using device secure enclave
- Sensitive fields (email, phone) encrypted at column level

**In Transit:**
- TLS 1.3 for all API communications
- Certificate pinning in mobile app
- HSTS headers enforced
- No mixed content allowed

### 7.3 API Security

**Protection Mechanisms:**

```python
# 1. Rate Limiting (via API Gateway)
@limiter.limit("50 per minute")
async def scan_medicine(file: UploadFile):
    ...

# 2. Input Validation (Pydantic models)
class MedicineVerifyRequest(BaseModel):
    medicine_data: MedicineData
    
    @validator('medicine_data')
    def validate_mrp(cls, v):
        if v.mrp < 0 or v.mrp > 10000:
            raise ValueError('Invalid MRP value')
        return v

# 3. SQL Injection Prevention (ORM)
# BAD: f"SELECT * FROM medicines WHERE name = '{name}'"
# GOOD:
medicines = db.query(Medicine).filter(Medicine.name == name).all()

# 4. XSS Prevention
# Sanitize all user inputs before storing
# Escape outputs when rendering
import bleach
safe_text = bleach.clean(user_input, tags=[], strip=True)

# 5. CSRF Protection (for web version)
# Use CSRF tokens for state-changing operations
```

**Security Headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

### 7.4 Privacy Protection

**Data Minimization:**
- Collect only essential data (no unnecessary personal info)
- Anonymous analytics (no PII in analytics events)
- User consent required for optional data collection

**Data Retention:**
- Scanned images auto-deleted after 7 days
- User can request data export (GDPR-style)
- Account deletion removes all user data within 30 days

**Third-Party Sharing:**
- No data shared with third parties without explicit consent
- Google Cloud APIs used with data processing agreements
- Analytics anonymized before sending to third-party tools

### 7.5 OWASP Top 10 Mitigation

| Vulnerability | Mitigation Strategy |
|---------------|---------------------|
| Injection | ORM usage, parameterized queries, input validation |
| Broken Authentication | JWT with expiry, secure token storage, MFA option |
| Sensitive Data Exposure | TLS 1.3, encryption at rest, secure logging |
| XML External Entities | N/A (no XML parsing) |
| Broken Access Control | Role-based access, authorization checks |
| Security Misconfiguration | Security headers, disabled debug mode in prod |
| XSS | Input sanitization, output escaping, CSP headers |
| Insecure Deserialization | JSON-only APIs, schema validation |
| Using Components with Known Vulnerabilities | Dependency scanning (Snyk), regular updates |
| Insufficient Logging & Monitoring | Centralized logging, alerting on suspicious activity |

---

## 8. Deployment Architecture

### 8.1 Cloud Infrastructure (GCP)

```
┌──────────────────────────────────────────────┐
│            Global Load Balancer              │
│         (Google Cloud Load Balancing)        │
└────────────────┬─────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
┌───────▼──────┐  ┌───────▼──────┐
│   Region 1   │  │   Region 2   │
│  (Mumbai)    │  │ (Singapore)  │
└──────┬───────┘  └──────┬───────┘
       │                 │
┌──────▼──────────────────▼──────┐
│      Cloud Run Services         │
│  ┌───────┬────────┬──────────┐ │
│  │  API  │  OCR   │   NLP    │ │
│  │Service│Service │ Service  │ │
│  └───────┴────────┴──────────┘ │
│     (Auto-scaling 0-100)        │
└──────┬──────────────────────────┘
       │
┌──────▼──────────────────────────┐
│   Cloud SQL (PostgreSQL)        │
│   - Primary (Mumbai)            │
│   - Read Replica (Singapore)    │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│   Memorystore (Redis)           │
│   - Cache Layer                 │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│   Cloud Storage                 │
│   - Medicine Images             │
│   - User-uploaded Images (temp) │
│   - Backup Files                │
└─────────────────────────────────┘
```

### 8.2 Container Configuration

**Dockerfile (FastAPI Backend):**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY ./app ./app

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8080/health')"

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

**Cloud Run Service Configuration:**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: MediMitra-api
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "100"
    spec:
      containers:
      - image: gcr.io/MediMitra/api:latest
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: "2"
            memory: "2Gi"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-credentials
              key: url
```

### 8.3 CI/CD Pipeline

**GitHub Actions Workflow:**
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    - name: Install dependencies
      run: pip install -r requirements.txt
    - name: Run tests
      run: pytest tests/ --cov=app --cov-report=xml
    - name: Upload coverage
      uses: codecov/codecov-action@v3
  
  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
    - name: Build Docker image
      run: |
        docker build -t gcr.io/${{ secrets.GCP_PROJECT }}/api:${{ github.sha }} .
    - name: Push to GCR
      run: |
        docker push gcr.io/${{ secrets.GCP_PROJECT }}/api:${{ github.sha }}
    - name: Deploy to Cloud Run
      run: |
        gcloud run deploy MediMitra-api \
          --image gcr.io/${{ secrets.GCP_PROJECT }}/api:${{ github.sha }} \
          --platform managed \
          --region asia-south1 \
          --allow-unauthenticated
```

### 8.4 Environment Management

**Development:**
- Local Docker Compose setup
- SQLite database (for quick iteration)
- Mock external APIs

**Staging:**
- GCP Cloud Run (separate project)
- Smaller resource allocation
- Subset of production data

**Production:**
- GCP Cloud Run (production project)
- Full resource allocation
- Regular backups
- Monitoring and alerting

---

## 9. Performance Optimization

### 9.1 Mobile App Optimization

**Techniques:**
```javascript
// 1. Image Compression before upload
import ImageResizer from 'react-native-image-resizer';

const compressImage = async (imagePath) => {
  return await ImageResizer.createResizedImage(
    imagePath,
    1024,  // maxWidth
    1024,  // maxHeight
    'JPEG',
    80,  // quality
    0,   // rotation
  );
};

// 2. Lazy loading of components
import { lazy, Suspense } from 'react';
const MedicineDetails = lazy(() => import('./MedicineDetails'));

// 3. Memoization for expensive computations
import { useMemo } from 'react';
const sortedMedicines = useMemo(
  () => medicines.sort((a, b) => a.name.localeCompare(b.name)),
  [medicines]
);

// 4. Virtual lists for large datasets
import { FlatList } from 'react-native';
<FlatList
  data={medicines}
  renderItem={renderMedicineItem}
  keyExtractor={(item) => item.id.toString()}
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  windowSize={5}
/>

// 5. Debounce search inputs
import { debounce } from 'lodash';
const debouncedSearch = debounce((query) => {
  searchMedicines(query);
}, 300);
```

**Bundle Size Optimization:**
- Code splitting by route
- Tree shaking unused code
- Remove unused dependencies
- Use Hermes JavaScript engine (Android)

### 9.2 Backend Optimization

**Database Query Optimization:**
```python
# 1. Use indexes for frequently queried fields
# Already defined in schema (idx_medicine_name, etc.)

# 2. Eager loading to avoid N+1 queries
medicines = db.query(Medicine).options(
    joinedload(Medicine.manufacturer)
).all()

# 3. Query only required fields
medicines = db.query(
    Medicine.id, 
    Medicine.name, 
    Medicine.mrp
).all()

# 4. Use database connection pooling
engine = create_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True
)

# 5. Pagination for large result sets
def get_medicines(page: int = 1, page_size: int = 20):
    offset = (page - 1) * page_size
    return db.query(Medicine).offset(offset).limit(page_size).all()
```

**Caching Strategy:**
```python
import redis
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_result(ttl=3600):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{args}:{kwargs}"
            
            # Try to get from cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # If not in cache, execute function
            result = await func(*args, **kwargs)
            
            # Store in cache
            redis_client.setex(
                cache_key, 
                ttl, 
                json.dumps(result)
            )
            
            return result
        return wrapper
    return decorator

@cache_result(ttl=3600)  # Cache for 1 hour
async def get_medicine_info(medicine_id: int):
    return db.query(Medicine).filter(Medicine.id == medicine_id).first()
```

**Async Processing for Heavy Tasks:**
```python
from celery import Celery

celery_app = Celery('tasks', broker='redis://localhost:6379/0')

@celery_app.task
def process_medicine_image(image_path: str):
    # Heavy OCR processing
    ocr_result = ocr_service.process_image(image_path)
    # Store result in database
    store_ocr_result(ocr_result)
    # Delete temporary image
    os.remove(image_path)

# In API endpoint
@app.post("/ocr/scan")
async def scan_medicine(file: UploadFile):
    # Save file temporarily
    temp_path = save_temp_file(file)
    
    # Trigger async processing
    task = process_medicine_image.delay(temp_path)
    
    # Return task ID for status checking
    return {"task_id": task.id}
```

### 9.3 CDN & Asset Optimization

**Static Asset Delivery:**
- Use Cloud CDN for medicine images
- Serve multiple image sizes (thumbnail, medium, full)
- WebP format for images (40% smaller than JPEG)
- Cache static assets for 1 year

**Image URL Structure:**
```
https://cdn.MediMitra.com/medicines/
  ├── thumbnails/paracetamol-500mg-thumb.webp (50x50)
  ├── medium/paracetamol-500mg-med.webp (300x300)
  └── full/paracetamol-500mg-full.webp (1024x1024)
```

### 9.4 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| App Launch Time | < 2 seconds | From tap to interactive |
| OCR Processing | < 3 seconds | Image upload to result display |
| API Response (p95) | < 500ms | Server processing time |
| Database Query (p95) | < 100ms | Query execution time |
| Time to Interactive (TTI) | < 5 seconds | Full page load + interaction |
| First Contentful Paint (FCP) | < 2 seconds | First visual element |

---

## 10. Testing Strategy

### 10.1 Testing Pyramid

```
        ┌─────────────┐
        │     E2E     │  (10% of tests)
        │   Testing   │
        └─────────────┘
      ┌─────────────────┐
      │   Integration   │  (30% of tests)
      │    Testing      │
      └─────────────────┘
    ┌───────────────────────┐
    │     Unit Testing      │  (60% of tests)
    │  (Functions, Classes) │
    └───────────────────────┘
```

### 10.2 Unit Testing

**Backend (Python/Pytest):**
```python
# tests/test_ocr_service.py
import pytest
from app.services.ocr_service import OCRService

@pytest.fixture
def ocr_service():
    return OCRService()

@pytest.fixture
def sample_medicine_text():
    return """
    Paracetamol 500mg
    MFG: 01/2024
    EXP: 12/2026
    Batch: XYZ123
    MRP: Rs. 15.00
    """

def test_extract_mrp(ocr_service, sample_medicine_text):
    result = ocr_service._extract_mrp(sample_medicine_text)
    assert result == 15.00

def test_extract_batch(ocr_service, sample_medicine_text):
    result = ocr_service._extract_batch(sample_medicine_text)
    assert result == "XYZ123"

def test_extract_expiry_date(ocr_service, sample_medicine_text):
    result = ocr_service._extract_date(sample_medicine_text, 'Exp')
    assert result == "12/2026"

@pytest.mark.asyncio
async def test_process_image_success(ocr_service, mocker):
    # Mock Google Vision API
    mock_response = mocker.Mock()
    mock_response.text_annotations = [
        mocker.Mock(description="Paracetamol 500mg...")
    ]
    mocker.patch.object(
        ocr_service.client, 
        'text_detection', 
        return_value=mock_response
    )
    
    result = await ocr_service.process_image(b'fake_image_bytes')
    
    assert 'full_text' in result
    assert 'extracted_data' in result
```

**Mobile (React Native/Jest):**
```javascript
// __tests__/services/OCRService.test.js
import OCRService from '../../src/services/OCRService';

describe('OCRService', () => {
  describe('parseText', () => {
    const sampleText = `
      Paracetamol 500mg
      Batch: XYZ123
      MRP: ₹15
      Exp: 12/2026
    `;
    
    it('should extract medicine name', () => {
      const result = OCRService.parseText(sampleText);
      expect(result.name).toBe('Paracetamol');
    });
    
    it('should extract MRP correctly', () => {
      const result = OCRService.parseText(sampleText);
      expect(result.mrp).toBe(15);
    });
    
    it('should extract batch number', () => {
      const result = OCRService.parseText(sampleText);
      expect(result.batchNumber).toBe('XYZ123');
    });
  });
});

// __tests__/components/MedicineCard.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import MedicineCard from '../../src/components/MedicineCard';

describe('MedicineCard', () => {
  const mockMedicine = {
    name: 'Paracetamol',
    manufacturer: 'Sun Pharma',
    expiryDate: '2026-12-31',
    riskLevel: 'low'
  };
  
  it('renders medicine information correctly', () => {
    const { getByText } = render(<MedicineCard medicine={mockMedicine} />);
    
    expect(getByText('Paracetamol')).toBeTruthy();
    expect(getByText('Sun Pharma')).toBeTruthy();
  });
  
  it('calls onPress when card is tapped', () => {
    const onPressMock = jest.fn();
    const { getByTestId } = render(
      <MedicineCard medicine={mockMedicine} onPress={onPressMock} />
    );
    
    fireEvent.press(getByTestId('medicine-card'));
    expect(onPressMock).toHaveBeenCalledTimes(1);
  });
});
```

### 10.3 Integration Testing

```python
# tests/integration/test_medicine_api.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_search_medicine():
    response = client.get("/api/v1/medicine/search?q=paracetamol")
    
    assert response.status_code == 200
    data = response.json()
    assert "results" in data
    assert len(data["results"]) > 0
    assert data["results"][0]["name"].lower().contains("paracetamol")

def test_get_medicine_details():
    response = client.get("/api/v1/medicine/123")
    
    assert response.status_code == 200
    data = response.json()
    assert "id" in data
    assert "name" in data
    assert "composition" in data

def test_verify_medicine():
    payload = {
        "medicine_data": {
            "name": "Paracetamol",
            "mrp": 15.00,
            "batch_number": "XYZ123",
            "exp_date": "12/2026"
        }
    }
    
    response = client.post("/api/v1/medicine/verify", json=payload)
    
    assert response.status_code == 200
    data = response.json()
    assert "risk_assessment" in data
    assert "overall_score" in data["risk_assessment"]
```

### 10.4 End-to-End Testing

**Mobile (Detox):**
```javascript
// e2e/medicine-scan.e2e.js
describe('Medicine Scan Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });
  
  it('should scan medicine and show results', async () => {
    // Navigate to scan screen
    await element(by.id('scan-medicine-btn')).tap();
    
    // Grant camera permission (if needed)
    // await device.grantPermission('camera');
    
    // Simulate image capture
    await element(by.id('capture-btn')).tap();
    
    // Wait for OCR processing
    await waitFor(element(by.id('medicine-result')))
      .toBeVisible()
      .withTimeout(5000);
    
    // Verify result screen elements
    await expect(element(by.id('medicine-name'))).toBeVisible();
    await expect(element(by.id('risk-indicator'))).toBeVisible();
    await expect(element(by.id('add-to-cabinet-btn'))).toBeVisible();
  });
  
  it('should add medicine to cabinet', async () => {
    // Assuming we're on the result screen from previous test
    await element(by.id('add-to-cabinet-btn')).tap();
    
    // Fill in additional details
    await element(by.id('quantity-input')).typeText('10');
    await element(by.id('save-btn')).tap();
    
    // Verify success message
    await expect(element(by.text('Added to cabinet'))).toBeVisible();
    
    // Navigate to cabinet
    await element(by.id('cabinet-tab')).tap();
    
    // Verify medicine appears in cabinet
    await expect(element(by.text('Paracetamol'))).toBeVisible();
  });
});
```

### 10.5 Performance Testing

**Load Testing (Locust):**
```python
# locustfile.py
from locust import HttpUser, task, between

class MediMitraUser(HttpUser):
    wait_time = between(1, 3)
    
    @task(3)
    def search_medicine(self):
        self.client.get("/api/v1/medicine/search?q=paracetamol")
    
    @task(2)
    def get_medicine_details(self):
        self.client.get("/api/v1/medicine/123")
    
    @task(1)
    def verify_medicine(self):
        self.client.post("/api/v1/medicine/verify", json={
            "medicine_data": {
                "name": "Paracetamol",
                "mrp": 15.00,
                "batch_number": "XYZ123",
                "exp_date": "12/2026"
            }
        })

# Run: locust -f locustfile.py --host=https://api.MediMitra.com
```

### 10.6 Test Coverage Goals

```
Overall Code Coverage: > 80%
  - Backend (Python): > 85%
  - Mobile (React Native): > 75%
  - Critical paths (auth, OCR, interactions): > 95%

Test Types Distribution:
  - Unit Tests: 60%
  - Integration Tests: 30%
  - E2E Tests: 10%
```

---

## Appendix

### A. Glossary of Terms

- **OCR:** Optical Character Recognition
- **NLP:** Natural Language Processing
- **STT:** Speech-to-Text
- **TTS:** Text-to-Speech
- **JWT:** JSON Web Token
- **ORM:** Object-Relational Mapping
- **CDN:** Content Delivery Network
- **NLEM:** National List of Essential Medicines
- **CDSCO:** Central Drugs Standard Control Organization

### B. References

- Google ML Kit Documentation: https://developers.google.com/ml-kit
- React Native Documentation: https://reactnative.dev/
- FastAPI Documentation: https://fastapi.tiangolo.com/
- PostgreSQL Documentation: https://www.postgresql.org/docs/
- WCAG 2.1 Guidelines: https://www.w3.org/WAI/WCAG21/quickref/

### C. Change Log

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2026-02-13 | Initial design document | MediMitra Team |

---

**Document Status:** Ready for Hackathon Development  
**Next Review Date:** Post-Hackathon (2026-02-15)  
**Approved By:** [To be filled during hackathon]