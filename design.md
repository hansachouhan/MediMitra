# MediMitra - Design Document
**Version:** 1.0 (Web Application)  
**Date:** February 2026  
**Project:** AI for Bharat Hackathon

---

## 1. System Architecture

### 1.1 High-Level Architecture

The system follows a three-tier architecture with client-side intelligence, stateless backend services, and persistent data storage.

```
┌──────────────────────────────────────────────────────┐
│             Client Layer (Browser)                   │
│  ┌────────────────────────────────────────────────┐ │
│  │  User Interface (React Components)             │ │
│  │  - Responsive layouts                           │ │
│  │  - Multilingual support                         │ │
│  │  - Accessibility features                       │ │
│  └────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────┐ │
│  │  Business Logic Layer                          │ │
│  │  - OCR Processing (Tesseract.js)               │ │
│  │  - Voice Recognition (Web Speech API)          │ │
│  │  - Text-to-Speech (Speech Synthesis)           │ │
│  │  - Medicine Analysis                            │ │
│  │  - Interaction Checking                         │ │
│  └────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────┐ │
│  │  Data Access Layer                             │ │
│  │  - IndexedDB (Local Storage)                   │ │
│  │  - HTTP Client (API Calls)                     │ │
│  │  - Cache Management                             │ │
│  │  - Service Worker (Offline)                    │ │
│  └────────────────────────────────────────────────┘ │
└───────────────────────┬──────────────────────────────┘
                        │
                    HTTPS / REST API
                        │
┌───────────────────────▼──────────────────────────────┐
│              Server Layer (Backend)                  │
│  ┌────────────────────────────────────────────────┐ │
│  │  API Gateway                                    │ │
│  │  - Request routing                              │ │
│  │  - Authentication                               │ │
│  │  - Rate limiting                                │ │
│  └────────────────────────────────────────────────┘ │
│  ┌───────────┬───────────┬───────────┬──────────┐ │
│  │   OCR     │    NLP    │ Medicine  │   Auth   │ │
│  │ Service   │  Service  │  Service  │ Service  │ │
│  └───────────┴───────────┴───────────┴──────────┘ │
└───────────────────────┬──────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────┐  ┌────▼────┐  ┌───────▼──────┐
│   Database   │  │  Cache  │  │   Storage    │
│  PostgreSQL  │  │  Redis  │  │  Cloudinary  │
│              │  │(optional)│  │   (images)   │
└──────────────┘  └─────────┘  └──────────────┘
```

### 1.2 Architecture Principles

**Client-Side Intelligence**
The application performs computation-intensive operations like OCR and voice recognition directly in the browser. This approach ensures user privacy (no image uploads), reduces server costs, and enables offline functionality.

**Progressive Web App Foundation**
Built as a PWA to provide native app-like experience with offline support, installability, and push notifications while maintaining web accessibility.

**Offline-First Design**
Core features function without internet connectivity by caching essential data locally and syncing when connection available.

**Microservices Backend**
Backend organized into independent services that can scale individually based on demand. Each service handles specific domain logic.

**RESTful API Design**
Communication between client and server follows REST principles with clear resource endpoints, standard HTTP methods, and JSON data format.

**Stateless Server**
Server maintains no session state, enabling horizontal scaling. All user context passed with each request via JWT tokens.

---

## 2. Technology Stack

### 2.1 Frontend Technologies

**Core Framework: React 18.2**
Modern component-based UI library chosen for its virtual DOM performance, large ecosystem, and excellent developer tooling. React's hooks enable clean state management and side effects handling.

**Build Tool: Vite 5.0**
Next-generation frontend build tool providing instant server start, lightning-fast hot module replacement, and optimized production builds. Significantly faster than traditional bundlers.

**Language: JavaScript (ES2022)**
Latest JavaScript features including async/await, optional chaining, and nullish coalescing for cleaner, more maintainable code.

**Key Frontend Libraries:**

**UI Component Library: shadcn/ui**
Accessible, customizable component library built on Radix UI primitives. Provides production-ready components with built-in accessibility and keyboard navigation.

**Styling: Tailwind CSS 3.4**
Utility-first CSS framework enabling rapid UI development with consistent design system. Custom configuration for MediMitra brand colors and spacing.

**Icons: Lucide React**
Modern icon library with consistent design and tree-shakeable imports for minimal bundle size.

**Routing: React Router 6.20**
Declarative routing for single-page application with nested routes, lazy loading, and programmatic navigation.

**State Management: Zustand 4.4**
Lightweight state management with minimal boilerplate. Simpler than Redux while providing similar capabilities for global state.

**Server State: TanStack Query (React Query) 5.0**
Powerful data synchronization library for fetching, caching, and updating server state. Automatic background refetching and optimistic updates.

---

### 2.2 OCR Technology

**Tesseract.js 5.0**
Pure JavaScript OCR library running entirely in browser via WebAssembly. Supports 100+ languages including English and Hindi. No server-side processing required, ensuring privacy and offline capability.

**Language Support:**
- English (Latin script)
- Hindi (Devanagari script)
- Combined eng+hin for bilingual packaging

**Optimization Strategies:**
- Image preprocessing for better accuracy
- Confidence threshold filtering
- Custom training data for pharmaceutical text
- Worker threads for non-blocking processing

---

### 2.3 Voice Technology

**Web Speech API**
Native browser API providing speech recognition and synthesis without external dependencies or API costs.

**Speech Recognition (SpeechRecognition/webkitSpeechRecognition):**
- Converts spoken words to text
- Supports Hindi and English
- Continuous recognition with interim results
- Confidence scores for transcription accuracy

**Speech Synthesis (SpeechSynthesisUtterance):**
- Converts text to natural speech
- Multiple voice options per language
- Adjustable speed, pitch, and volume
- Queuing for sequential utterances

**Browser Support:**
- Chrome, Edge: Full support
- Firefox: Partial support
- Safari: Limited but functional

---

### 2.4 Storage Solutions

**IndexedDB via Dexie.js 3.2**
Browser-based NoSQL database for storing structured data locally. Dexie.js provides Promise-based API wrapping IndexedDB's complex callback-based interface.

**Storage Categories:**
- Medicine cabinet (unlimited entries)
- Scan history (last 50 scans)
- Cached medicine database (top 500)
- User preferences
- Voice conversation history

**Storage Limits:**
- Minimum 50MB across browsers
- Chrome/Edge: Dynamic based on available space
- Firefox: 10% of disk space
- Safari: 1GB per origin

**localStorage**
Simple key-value storage for non-critical data like UI preferences and session tokens. 5-10MB limit across browsers.

---

### 2.5 Backend Technologies

**Web Framework: FastAPI 0.104 (Python 3.11)**
Modern, high-performance web framework for building APIs. Automatic OpenAPI documentation, request validation via Pydantic, and async support out of the box.

**Alternative: Express.js 4.18 (Node.js 18)**
Mature, minimalist web framework for Node.js. Lightweight with extensive middleware ecosystem.

**Database: PostgreSQL 15**
Robust relational database with excellent JSON support for flexible schema. ACID compliance ensures data integrity.

**ORM: SQLAlchemy 2.0 (Python) / Prisma 5.0 (Node.js)**
Database abstraction layer providing object-oriented interface to database. Migration management, query building, and relationship handling.

**Caching: Redis 7 (Optional)**
In-memory data structure store for caching frequently accessed data. Reduces database load and improves response times.

---

### 2.6 Deployment Infrastructure

**Frontend Hosting: Vercel**
Serverless platform optimized for frontend frameworks. Key features:
- Global CDN for fast content delivery
- Automatic HTTPS
- Instant rollback capability
- Preview deployments for branches
- Edge functions for dynamic content
- Free tier: Unlimited bandwidth and builds

**Backend Hosting: Render.com**
Cloud platform for deploying backend services and databases. Key features:
- Automatic deploys from Git
- Managed PostgreSQL database
- Free SSL certificates
- Automatic health checks
- Background workers
- Free tier: 750 hours/month compute

**Alternative: Railway**
Similar platform with generous free tier and excellent developer experience.

**Image Storage: Cloudinary**
Cloud-based image management with automatic optimization and transformation.
- Free tier: 25GB storage, 25GB bandwidth
- Image optimization and compression
- Responsive image delivery
- Backup and redundancy

---

### 2.7 Development Tools

**Version Control: Git with GitHub**
Distributed version control for code collaboration and history tracking.

**Package Manager: npm/pnpm**
Dependency management with lockfile for reproducible builds.

**Linting: ESLint**
Code quality tool for identifying problematic patterns and enforcing style guidelines.

**Formatting: Prettier**
Opinionated code formatter ensuring consistent style across codebase.

**Testing: Vitest + Testing Library**
Fast unit testing framework with React Testing Library for component testing.

**E2E Testing: Playwright**
Automated browser testing across Chrome, Firefox, and Safari.

---

## 3. Component Architecture

### 3.1 Frontend Component Structure

```
src/
├── app/
│   ├── App.jsx                    # Root component
│   ├── Router.jsx                 # Route configuration
│   └── Layout.jsx                 # Common layout wrapper
│
├── pages/
│   ├── HomePage/
│   │   ├── HomePage.jsx          # Main landing page
│   │   ├── QuickActions.jsx      # Scan and voice buttons
│   │   └── RecentScans.jsx       # Scan history preview
│   │
│   ├── ScanPage/
│   │   ├── ScanPage.jsx          # Image capture interface
│   │   ├── WebcamCapture.jsx     # Live camera feed
│   │   ├── ImageUpload.jsx       # File upload component
│   │   └── ProcessingOverlay.jsx # OCR progress indicator
│   │
│   ├── ResultPage/
│   │   ├── ResultPage.jsx        # Scan results display
│   │   ├── MedicineInfo.jsx      # Medicine details card
│   │   ├── RiskAssessment.jsx    # Safety verification
│   │   └── ActionButtons.jsx     # Add to cabinet, share
│   │
│   ├── CabinetPage/
│   │   ├── CabinetPage.jsx       # Medicine cabinet view
│   │   ├── MedicineList.jsx      # Cabinet entries list
│   │   ├── MedicineCard.jsx      # Individual medicine card
│   │   ├── ExpiryAlerts.jsx      # Expiring medicines
│   │   └── CabinetFilters.jsx    # Sort and filter controls
│   │
│   ├── VoiceAssistantPage/
│   │   ├── VoiceAssistantPage.jsx
│   │   ├── VoiceInput.jsx        # Microphone and recording
│   │   ├── TranscriptDisplay.jsx # Speech-to-text output
│   │   ├── ResponseDisplay.jsx   # Assistant response
│   │   └── ConversationHistory.jsx
│   │
│   └── SettingsPage/
│       ├── SettingsPage.jsx
│       ├── LanguageSelector.jsx
│       ├── AccessibilitySettings.jsx
│       └── DataManagement.jsx
│
├── components/
│   ├── common/
│   │   ├── Button.jsx            # Reusable button component
│   │   ├── Card.jsx              # Card container
│   │   ├── Input.jsx             # Form input field
│   │   ├── Select.jsx            # Dropdown select
│   │   ├── Modal.jsx             # Modal dialog
│   │   ├── Toast.jsx             # Notification toast
│   │   ├── LoadingSpinner.jsx    # Loading indicator
│   │   └── ErrorBoundary.jsx     # Error catching wrapper
│   │
│   ├── medicine/
│   │   ├── MedicineSearchBar.jsx
│   │   ├── DosageInfo.jsx
│   │   ├── SideEffectsList.jsx
│   │   ├── InteractionWarning.jsx
│   │   └── ManufacturerInfo.jsx
│   │
│   ├── navigation/
│   │   ├── Header.jsx            # Top navigation bar
│   │   ├── Sidebar.jsx           # Side menu
│   │   ├── BottomNav.jsx         # Mobile bottom nav
│   │   └── Breadcrumbs.jsx       # Navigation breadcrumbs
│   │
│   └── indicators/
│       ├── RiskIndicator.jsx     # Risk level display
│       ├── ConfidenceScore.jsx   # OCR confidence
│       ├── StatusBadge.jsx       # Status indicators
│       └── ProgressBar.jsx       # Progress indicator
│
├── services/
│   ├── ocr/
│   │   ├── ocrService.js         # Tesseract wrapper
│   │   ├── imageProcessor.js     # Image preprocessing
│   │   └── textParser.js         # Extract structured data
│   │
│   ├── voice/
│   │   ├── speechRecognition.js  # Speech-to-text
│   │   ├── speechSynthesis.js    # Text-to-speech
│   │   └── voiceCommands.js      # Command processing
│   │
│   ├── api/
│   │   ├── apiClient.js          # HTTP client configuration
│   │   ├── medicineApi.js        # Medicine endpoints
│   │   ├── authApi.js            # Authentication
│   │   └── syncApi.js            # Data synchronization
│   │
│   └── storage/
│       ├── databaseService.js    # IndexedDB wrapper
│       ├── cacheService.js       # Cache management
│       └── syncService.js        # Offline sync queue
│
├── store/
│   ├── medicineStore.js          # Medicine state
│   ├── cabinetStore.js           # User cabinet state
│   ├── uiStore.js                # UI preferences
│   └── authStore.js              # Authentication state
│
├── hooks/
│   ├── useOCR.js                 # OCR processing hook
│   ├── useVoice.js               # Voice interaction hook
│   ├── useMedicineData.js        # Medicine data fetching
│   ├── useCabinet.js             # Cabinet operations
│   └── useOffline.js             # Offline status detection
│
├── utils/
│   ├── imageUtils.js             # Image manipulation
│   ├── dateUtils.js              # Date formatting
│   ├── validationUtils.js        # Input validation
│   ├── translationUtils.js       # i18n helpers
│   └── analyticsUtils.js         # Event tracking
│
├── assets/
│   ├── images/                   # Static images
│   ├── icons/                    # Icon assets
│   └── locales/                  # Translation files
│       ├── en.json
│       ├── hi.json
│       ├── ta.json
│       └── te.json
│
└── styles/
    ├── globals.css               # Global styles
    ├── variables.css             # CSS variables
    └── tailwind.config.js        # Tailwind configuration
```

### 3.2 Service Layer Details

**OCR Service (ocrService.js)**

Manages all OCR-related operations including image preprocessing, text extraction, and result parsing.

```javascript
import Tesseract from 'tesseract.js';

class OCRService {
  constructor() {
    this.worker = null;
    this.isInitialized = false;
  }

  async initialize() {
    if (this.isInitialized) return;
    
    this.worker = await Tesseract.createWorker('eng+hin', 1, {
      logger: (m) => this.onProgress(m)
    });
    
    this.isInitialized = true;
  }

  async processImage(imageFile, options = {}) {
    await this.initialize();
    
    // Preprocess image for better OCR accuracy
    const processedImage = await this.preprocessImage(imageFile);
    
    // Perform OCR
    const result = await this.worker.recognize(processedImage);
    
    // Parse extracted text into structured data
    const parsedData = this.parseText(result.data.text);
    
    return {
      rawText: result.data.text,
      confidence: result.data.confidence,
      medicineData: parsedData,
      blocks: result.data.blocks
    };
  }

  async preprocessImage(imageFile) {
    // Convert to grayscale
    // Adjust contrast
    // Remove noise
    // Deskew if tilted
    return processedImageData;
  }

  parseText(text) {
    return {
      name: this.extractMedicineName(text),
      composition: this.extractComposition(text),
      mrp: this.extractMRP(text),
      batchNumber: this.extractBatchNumber(text),
      mfgDate: this.extractDate(text, 'Mfg'),
      expDate: this.extractDate(text, 'Exp'),
      manufacturer: this.extractManufacturer(text)
    };
  }

  extractMedicineName(text) {
    // First line typically contains medicine name
    const lines = text.split('\n').filter(line => line.trim());
    if (lines.length === 0) return null;
    
    // Medicine names are usually capitalized and at the start
    const namePattern = /^[A-Z][a-zA-Z\s]+(?:\d+(?:mg|ml|mcg))?/;
    const match = lines[0].match(namePattern);
    return match ? match[0].trim() : null;
  }

  extractMRP(text) {
    // Match patterns like "MRP: Rs. 15.00" or "MRP ₹15"
    const patterns = [
      /MRP[:\s]*(?:Rs\.?)?\s*₹?\s*(\d+\.?\d*)/i,
      /₹\s*(\d+\.?\d*)/,
      /Rs\.?\s*(\d+\.?\d*)/i
    ];
    
    for (const pattern of patterns) {
      const match = text.match(pattern);
      if (match) return parseFloat(match[1]);
    }
    
    return null;
  }

  extractBatchNumber(text) {
    // Match patterns like "Batch: ABC123" or "B.No: XYZ789"
    const patterns = [
      /Batch[:\s]*([A-Z0-9]+)/i,
      /B\.?No\.?[:\s]*([A-Z0-9]+)/i,
      /Lot[:\s]*([A-Z0-9]+)/i
    ];
    
    for (const pattern of patterns) {
      const match = text.match(pattern);
      if (match) return match[1];
    }
    
    return null;
  }

  extractDate(text, type) {
    // Match patterns for manufacturing or expiry dates
    const prefix = type === 'Mfg' ? 'Mfg|Mfd|Man' : 'Exp|Expiry';
    const patterns = [
      new RegExp(`${prefix}[.:\\s]*(\\d{2})[/-](\\d{4})`, 'i'),
      new RegExp(`${prefix}[.:\\s]*(\\d{2})[/-](\\d{2})[/-](\\d{2,4})`, 'i')
    ];
    
    for (const pattern of patterns) {
      const match = text.match(pattern);
      if (match) {
        return this.normalizeDate(match);
      }
    }
    
    return null;
  }

  async terminate() {
    if (this.worker) {
      await this.worker.terminate();
      this.isInitialized = false;
    }
  }
}

export default new OCRService();
```

**Voice Service (voiceService.js)**

Handles speech recognition and synthesis using Web Speech API.

```javascript
class VoiceService {
  constructor() {
    this.recognition = null;
    this.synthesis = window.speechSynthesis;
    this.isListening = false;
    this.currentLanguage = 'hi-IN';
  }

  startListening(language = 'hi-IN', onResult, onError) {
    if (!('webkitSpeechRecognition' in window)) {
      onError(new Error('Speech recognition not supported'));
      return;
    }

    this.recognition = new webkitSpeechRecognition();
    this.recognition.continuous = false;
    this.recognition.interimResults = true;
    this.recognition.lang = language;

    this.recognition.onstart = () => {
      this.isListening = true;
    };

    this.recognition.onresult = (event) => {
      const result = event.results[event.results.length - 1];
      const transcript = result[0].transcript;
      const isFinal = result.isFinal;
      const confidence = result[0].confidence;

      onResult({
        transcript,
        isFinal,
        confidence
      });
    };

    this.recognition.onerror = (event) => {
      this.isListening = false;
      onError(event.error);
    };

    this.recognition.onend = () => {
      this.isListening = false;
    };

    this.recognition.start();
  }

  stopListening() {
    if (this.recognition && this.isListening) {
      this.recognition.stop();
    }
  }

  speak(text, language = 'hi-IN', options = {}) {
    // Cancel any ongoing speech
    this.synthesis.cancel();

    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = language;
    utterance.rate = options.rate || 1.0;
    utterance.pitch = options.pitch || 1.0;
    utterance.volume = options.volume || 1.0;

    // Select appropriate voice for language
    const voices = this.synthesis.getVoices();
    const voice = voices.find(v => v.lang === language) || voices[0];
    utterance.voice = voice;

    return new Promise((resolve, reject) => {
      utterance.onend = resolve;
      utterance.onerror = reject;
      this.synthesis.speak(utterance);
    });
  }

  stopSpeaking() {
    this.synthesis.cancel();
  }

  getAvailableVoices(language) {
    const voices = this.synthesis.getVoices();
    return voices.filter(v => v.lang.startsWith(language));
  }
}

export default new VoiceService();
```

**Database Service (databaseService.js)**

Manages IndexedDB operations using Dexie.js for simplified API.

```javascript
import Dexie from 'dexie';

class DatabaseService extends Dexie {
  constructor() {
    super('MediMitraDB');
    
    this.version(1).stores({
      medicines: '++id, name, generic_name, manufacturer',
      userMedicines: '++id, medicine_id, expiry_date, purchase_date',
      scanHistory: '++id, scanned_at, medicine_id',
      interactions: '++id, drug_a_id, drug_b_id, severity',
      settings: 'key'
    });
  }

  // Medicine Cabinet Operations
  async addToCabinet(medicineData) {
    return await this.userMedicines.add({
      ...medicineData,
      created_at: new Date(),
      updated_at: new Date()
    });
  }

  async getCabinet() {
    return await this.userMedicines
      .orderBy('expiry_date')
      .toArray();
  }

  async updateCabinetItem(id, updates) {
    return await this.userMedicines.update(id, {
      ...updates,
      updated_at: new Date()
    });
  }

  async removeCabinetItem(id) {
    return await this.userMedicines.delete(id);
  }

  // Scan History Operations
  async addScanHistory(scanData) {
    return await this.scanHistory.add({
      ...scanData,
      scanned_at: new Date()
    });
  }

  async getRecentScans(limit = 10) {
    return await this.scanHistory
      .orderBy('scanned_at')
      .reverse()
      .limit(limit)
      .toArray();
  }

  // Medicine Data Operations
  async cacheMedicine(medicineData) {
    return await this.medicines.put(medicineData);
  }

  async searchMedicines(query) {
    const lowerQuery = query.toLowerCase();
    return await this.medicines
      .filter(med => 
        med.name.toLowerCase().includes(lowerQuery) ||
        med.generic_name?.toLowerCase().includes(lowerQuery)
      )
      .toArray();
  }

  // Settings Operations
  async getSetting(key) {
    const result = await this.settings.get(key);
    return result?.value;
  }

  async setSetting(key, value) {
    return await this.settings.put({ key, value });
  }

  // Cleanup old data
  async cleanupOldScans(daysToKeep = 30) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysToKeep);
    
    return await this.scanHistory
      .where('scanned_at')
      .below(cutoffDate)
      .delete();
  }
}

export default new DatabaseService();
```

---

## 4. Database Design

### 4.1 Backend Database Schema (PostgreSQL)

**Medicines Table**
```sql
CREATE TABLE medicines (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    generic_name VARCHAR(255),
    composition TEXT,
    manufacturer_id INTEGER REFERENCES manufacturers(id),
    therapeutic_class VARCHAR(100),
    schedule VARCHAR(20),
    
    -- Pricing information
    typical_mrp_min DECIMAL(10,2),
    typical_mrp_max DECIMAL(10,2),
    nlem_listed BOOLEAN DEFAULT FALSE,
    
    -- Medical information
    indications TEXT,
    mechanism TEXT,
    side_effects JSONB,
    contraindications TEXT,
    dosage_info JSONB,
    storage_instructions TEXT,
    
    -- Regulatory
    prescription_required BOOLEAN DEFAULT TRUE,
    cdsco_approved BOOLEAN DEFAULT TRUE,
    approval_date DATE,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    data_source VARCHAR(100),
    
    -- Indexes for fast search
    CONSTRAINT unique_medicine UNIQUE(name, manufacturer_id)
);

CREATE INDEX idx_medicines_name ON medicines(name);
CREATE INDEX idx_medicines_generic ON medicines(generic_name);
CREATE INDEX idx_medicines_manufacturer ON medicines(manufacturer_id);
CREATE INDEX idx_medicines_therapeutic ON medicines(therapeutic_class);
CREATE FULLTEXT INDEX idx_medicines_search ON medicines(name, generic_name, composition);
```

**Manufacturers Table**
```sql
CREATE TABLE manufacturers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    country VARCHAR(100),
    headquarters_city VARCHAR(100),
    website VARCHAR(255),
    contact_email VARCHAR(255),
    
    -- Verification status
    verified BOOLEAN DEFAULT FALSE,
    license_number VARCHAR(100),
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_manufacturers_name ON manufacturers(name);
CREATE INDEX idx_manufacturers_country ON manufacturers(country);
```

**Medicine Batches Table**
```sql
CREATE TABLE medicine_batches (
    id SERIAL PRIMARY KEY,
    medicine_id INTEGER REFERENCES medicines(id) ON DELETE CASCADE,
    batch_number VARCHAR(50) NOT NULL,
    
    -- Dates
    manufacturing_date DATE,
    expiry_date DATE,
    
    -- Pricing
    mrp DECIMAL(10,2),
    
    -- Manufacturing details
    plant_location VARCHAR(255),
    quality_certificate VARCHAR(255),
    
    -- Verification
    verified BOOLEAN DEFAULT FALSE,
    verification_date TIMESTAMP,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_batch UNIQUE(medicine_id, batch_number)
);

CREATE INDEX idx_batches_medicine ON medicine_batches(medicine_id);
CREATE INDEX idx_batches_expiry ON medicine_batches(expiry_date);
```

**Drug Interactions Table**
```sql
CREATE TABLE drug_interactions (
    id SERIAL PRIMARY KEY,
    drug_a_id INTEGER REFERENCES medicines(id),
    drug_b_id INTEGER REFERENCES medicines(id),
    
    -- Interaction details
    severity VARCHAR(20) CHECK (severity IN ('severe', 'moderate', 'minor')),
    mechanism TEXT,
    description TEXT,
    clinical_effects TEXT,
    
    -- Recommendations
    recommendation TEXT,
    monitoring_required BOOLEAN DEFAULT FALSE,
    
    -- Evidence
    evidence_level VARCHAR(20),
    source VARCHAR(255),
    references TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Ensure no duplicate interactions (order doesn't matter)
    CONSTRAINT unique_interaction UNIQUE(LEAST(drug_a_id, drug_b_id), GREATEST(drug_a_id, drug_b_id)),
    CONSTRAINT different_drugs CHECK (drug_a_id != drug_b_id)
);

CREATE INDEX idx_interactions_severity ON drug_interactions(severity);
CREATE INDEX idx_interactions_drugs ON drug_interactions(drug_a_id, drug_b_id);
```

**Medicine Translations Table**
```sql
CREATE TABLE medicine_translations (
    id SERIAL PRIMARY KEY,
    medicine_id INTEGER REFERENCES medicines(id) ON DELETE CASCADE,
    language_code VARCHAR(10) NOT NULL,
    
    -- Translated fields
    name_translation VARCHAR(255),
    indications_translation TEXT,
    side_effects_translation TEXT,
    dosage_translation TEXT,
    contraindications_translation TEXT,
    
    -- Quality
    translation_quality VARCHAR(20),
    verified_by VARCHAR(100),
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_translation UNIQUE(medicine_id, language_code)
);

CREATE INDEX idx_translations_medicine ON medicine_translations(medicine_id);
CREATE INDEX idx_translations_language ON medicine_translations(language_code);
```

**Users Table**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    phone_number VARCHAR(20) UNIQUE,
    password_hash VARCHAR(255),
    
    -- Preferences
    preferred_language VARCHAR(10) DEFAULT 'en',
    font_size_preference VARCHAR(20) DEFAULT 'medium',
    high_contrast_mode BOOLEAN DEFAULT FALSE,
    
    -- Status
    email_verified BOOLEAN DEFAULT FALSE,
    phone_verified BOOLEAN DEFAULT FALSE,
    account_status VARCHAR(20) DEFAULT 'active',
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone ON users(phone_number);
```

### 4.2 Client-Side Database Schema (IndexedDB)

Managed through Dexie.js with the following structure:

```javascript
db.version(1).stores({
  // Medicine data cache (from server)
  medicines: '++id, name, generic_name, manufacturer, &server_id',
  
  // User's personal medicine cabinet
  userMedicines: `
    ++id, 
    medicine_id, 
    expiry_date, 
    [medicine_id+expiry_date],
    purchase_date
  `,
  
  // Scan history
  scanHistory: `
    ++id, 
    scanned_at, 
    medicine_id,
    [medicine_id+scanned_at]
  `,
  
  // Drug interactions cache
  interactions: `
    ++id,
    drug_a_id,
    drug_b_id,
    severity,
    [drug_a_id+drug_b_id]
  `,
  
  // Application settings
  settings: 'key',
  
  // Offline sync queue
  syncQueue: '++id, created_at, synced'
});
```

---

## 5. API Design

### 5.1 Base Configuration

**Base URL:** `https://api.MediMitra.com/v1`

**Authentication:** Bearer token in Authorization header
```
Authorization: Bearer <jwt_token>
```

**Response Format:** JSON with consistent structure
```json
{
  "success": true,
  "data": {},
  "meta": {},
  "errors": []
}
```

### 5.2 Medicine Endpoints

**Search Medicines**
```
GET /medicines/search

Query Parameters:
  - q (required): Search query string
  - lang (optional): Response language code (default: 'en')
  - limit (optional): Maximum results (default: 10, max: 50)
  - offset (optional): Pagination offset (default: 0)

Response:
{
  "success": true,
  "data": {
    "medicines": [
      {
        "id": 123,
        "name": "Paracetamol",
        "generic_name": "Acetaminophen",
        "manufacturer": "Sun Pharma",
        "composition": "Paracetamol 500mg",
        "typical_mrp": "15-20",
        "nlem_listed": true,
        "prescription_required": false
      }
    ],
    "total": 42,
    "limit": 10,
    "offset": 0
  }
}
```

**Get Medicine Details**
```
GET /medicines/{id}

Query Parameters:
  - lang (optional): Response language (default: 'en')
  - include (optional): Additional data (interactions,batches)

Response:
{
  "success": true,
  "data": {
    "id": 123,
    "name": "Paracetamol",
    "generic_name": "Acetaminophen",
    "composition": "Paracetamol 500mg",
    "therapeutic_class": "Analgesic, Antipyretic",
    "manufacturer": {
      "id": 45,
      "name": "Sun Pharma",
      "country": "India"
    },
    "indications": "Treatment of fever and mild to moderate pain",
    "side_effects": [
      {"effect": "Nausea", "frequency": "common"},
      {"effect": "Allergic reactions", "frequency": "rare"}
    ],
    "dosage": {
      "adult": "1-2 tablets every 4-6 hours, max 4g/day",
      "child": "10-15mg/kg every 4-6 hours"
    },
    "contraindications": "Severe liver disease, alcohol dependence",
    "storage": "Store below 25°C in dry place",
    "prescription_required": false,
    "nlem_listed": true
  }
}
```

**Verify Medicine**
```
POST /medicines/verify

Request Body:
{
  "medicine_data": {
    "name": "Paracetamol",
    "composition": "Paracetamol 500mg",
    "mrp": 12.00,
    "batch_number": "XYZ123",
    "mfg_date": "01/2024",
    "exp_date": "12/2026",
    "manufacturer": "Sun Pharma"
  },
  "check_types": ["price", "expiry", "batch", "packaging"]
}

Response:
{
  "success": true,
  "data": {
    "overall_risk_level": "low",
    "risk_score": 25.0,
    "checks": [
      {
        "type": "price_verification",
        "status": "pass",
        "details": "MRP within expected range (₹15-20)",
        "risk_contribution": 0.0
      },
      {
        "type": "expiry_check",
        "status": "pass",
        "details": "Valid until Dec 2026 (11 months remaining)",
        "risk_contribution": 0.0
      },
      {
        "type": "batch_format",
        "status": "warning",
        "details": "Batch format could not be verified against manufacturer database",
        "risk_contribution": 50.0
      }
    ],
    "recommendations": [
      "Verify batch number with pharmacist or manufacturer",
      "Check for security features like holograms on packaging",
      "Compare with known authentic samples if available"
    ],
    "verified_at": "2026-02-15T10:30:00Z"
  }
}
```

**Check Drug Interactions**
```
POST /medicines/interactions

Request Body:
{
  "medicine_ids": [123, 456, 789],
  "include_food_interactions": true
}

Response:
{
  "success": true,
  "data": {
    "interactions": [
      {
        "id": 1001,
        "medicines": [
          {"id": 123, "name": "Paracetamol"},
          {"id": 456, "name": "Aspirin"}
        ],
        "severity": "moderate",
        "mechanism": "Both inhibit prostaglandin synthesis",
        "description": "Combined use may increase risk of stomach bleeding and kidney problems",
        "clinical_effects": "Gastrointestinal irritation, reduced renal function",
        "recommendation": "Take at least 2 hours apart. Monitor for stomach pain or bleeding. Consult doctor if symptoms persist.",
        "monitoring_required": true
      }
    ],
    "food_interactions": [
      {
        "medicine_id": 123,
        "medicine_name": "Paracetamol",
        "food_item": "Alcohol",
        "severity": "severe",
        "description": "Alcohol increases risk of liver damage with paracetamol"
      }
    ],
    "total_interactions": 1,
    "max_severity": "moderate"
  }
}
```

### 5.3 Voice Assistant Endpoints

**Process Voice Query**
```
POST /voice/query

Request Body:
{
  "transcript": "यह दवाई कब लेनी चाहिए?",
  "language": "hi-IN",
  "context": {
    "medicine_id": 123,
    "previous_queries": []
  }
}

Response:
{
  "success": true,
  "data": {
    "intent": "dosage_timing",
    "response_text": "यह दवाई दिन में दो बार लेनी चाहिए, सुबह और शाम, खाने के बाद।",
    "response_data": {
      "dosage_timing": ["morning", "evening"],
      "with_food": true,
      "interval_hours": 12
    },
    "confidence": 0.95,
    "follow_up_suggestions": [
      "क्या इसके साइड इफेक्ट हैं?",
      "कितने दिन तक लेनी है?"
    ]
  }
}
```

### 5.4 User Cabinet Endpoints

**Get User Cabinet**
```
GET /cabinet

Query Parameters:
  - include_expired (optional): Include expired medicines (default: false)
  - sort (optional): Sort field (expiry_date, name, purchase_date)

Response:
{
  "success": true,
  "data": {
    "medicines": [
      {
        "id": 1,
        "medicine": {
          "id": 123,
          "name": "Paracetamol",
          "generic_name": "Acetaminophen",
          "image_url": "https://cdn.MediMitra.com/medicines/123.jpg"
        },
        "quantity": 10,
        "unit": "tablets",
        "purchase_date": "2026-01-15",
        "expiry_date": "2026-12-31",
        "days_until_expiry": 319,
        "reminder_enabled": true,
        "notes": "Take after meals"
      }
    ],
    "statistics": {
      "total_medicines": 5,
      "expiring_soon": 2,
      "expired": 0
    }
  }
}
```

**Add to Cabinet**
```
POST /cabinet

Request Body:
{
  "medicine_id": 123,
  "quantity": 10,
  "unit": "tablets",
  "purchase_date": "2026-02-15",
  "expiry_date": "2026-12-31",
  "reminder_enabled": true,
  "reminder_days_before": 30,
  "notes": "Prescribed by Dr. Sharma"
}

Response:
{
  "success": true,
  "data": {
    "id": 1,
    "medicine_id": 123,
    "created_at": "2026-02-15T10:00:00Z"
  }
}
```

### 5.5 Error Responses

**Standard Error Format**
```json
{
  "success": false,
  "errors": [
    {
      "code": "MEDICINE_NOT_FOUND",
      "message": "Medicine with ID 999 not found",
      "field": "medicine_id",
      "type": "not_found"
    }
  ],
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2026-02-15T10:00:00Z"
  }
}
```

**HTTP Status Codes:**
- 200: Success
- 201: Resource created
- 400: Bad request (validation error)
- 401: Unauthorized (invalid/missing token)
- 403: Forbidden (insufficient permissions)
- 404: Resource not found
- 429: Too many requests (rate limit exceeded)
- 500: Internal server error

### 5.6 Rate Limiting

**Rate Limit Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1708430400
```

**Rate Limits:**
- Public endpoints: 100 requests/minute per IP
- Authenticated users: 1000 requests/hour
- Search endpoints: 50 requests/minute
- Verification endpoints: 20 requests/minute

---

## 6. UI/UX Design Specifications

### 6.1 Design System

**Color Palette:**
```css
:root {
  /* Primary Colors */
  --primary-50: #E3F2FD;
  --primary-100: #BBDEFB;
  --primary-500: #2196F3;
  --primary-700: #1976D2;
  --primary-900: #0D47A1;
  
  /* Success Colors */
  --success-50: #E8F5E9;
  --success-500: #4CAF50;
  --success-700: #388E3C;
  
  /* Warning Colors */
  --warning-50: #FFF3E0;
  --warning-500: #FF9800;
  --warning-700: #F57C00;
  
  /* Error Colors */
  --error-50: #FFEBEE;
  --error-500: #F44336;
  --error-700: #D32F2F;
  
  /* Neutral Colors */
  --gray-50: #FAFAFA;
  --gray-100: #F5F5F5;
  --gray-200: #EEEEEE;
  --gray-500: #9E9E9E;
  --gray-700: #616161;
  --gray-900: #212121;
  
  /* Text Colors */
  --text-primary: var(--gray-900);
  --text-secondary: var(--gray-700);
  --text-disabled: var(--gray-500);
  
  /* Background Colors */
  --bg-primary: #FFFFFF;
  --bg-secondary: var(--gray-50);
  --bg-tertiary: var(--gray-100);
}
```

**Typography:**
```css
:root {
  /* Font Families */
  --font-primary: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-hindi: 'Noto Sans Devanagari', sans-serif;
  
  /* Font Sizes */
  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  
  /* Font Weights */
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
  
  /* Line Heights */
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;
}
```

**Spacing Scale:**
```css
:root {
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-12: 3rem;    /* 48px */
  --space-16: 4rem;    /* 64px */
}
```

**Border Radius:**
```css
:root {
  --radius-sm: 0.25rem;  /* 4px */
  --radius-md: 0.5rem;   /* 8px */
  --radius-lg: 0.75rem;  /* 12px */
  --radius-xl: 1rem;     /* 16px */
  --radius-full: 9999px;
}
```

### 6.2 Component Specifications

**Button Component:**
```jsx
<Button
  variant="primary"      // primary, secondary, outline, ghost
  size="md"             // sm, md, lg
  fullWidth={false}
  disabled={false}
  loading={false}
  icon={<IconComponent />}
  iconPosition="left"   // left, right
>
  Button Text
</Button>
```

**Risk Indicator Component:**
```jsx
<RiskIndicator
  level="low"           // low, medium, high
  score={25}            // 0-100
  checks={[
    { type: 'price', status: 'pass', message: '...' },
    { type: 'expiry', status: 'pass', message: '...' },
    { type: 'batch', status: 'warning', message: '...' }
  ]}
  showDetails={true}
/>
```

### 6.3 Responsive Breakpoints

```css
/* Mobile First Approach */
/* Default: Mobile (320px - 480px) */

@media (min-width: 481px) {
  /* Tablet: 481px - 768px */
}

@media (min-width: 769px) {
  /* Desktop: 769px - 1024px */
}

@media (min-width: 1025px) {
  /* Large Desktop: 1025px+ */
}
```

### 6.4 Accessibility Features

**ARIA Labels:**
- All interactive elements have descriptive labels
- Form inputs have associated labels
- Icons have aria-label or aria-hidden
- Dynamic content changes announced to screen readers

**Keyboard Navigation:**
- Tab order follows logical flow
- All interactive elements keyboard accessible
- Visible focus indicators
- Keyboard shortcuts for common actions

**Color Contrast:**
- Text contrast ratio minimum 4.5:1 (WCAG AA)
- Large text minimum 3:1
- High contrast mode available
- Color not sole indicator of information

---

## 7. Security Architecture

### 7.1 Authentication Flow

**JWT Token Structure:**
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_123",
    "email": "user@example.com",
    "role": "user",
    "iat": 1708430400,
    "exp": 1708516800,
    "jti": "token_abc123"
  }
}
```

**Token Lifecycle:**
1. User logs in with credentials
2. Server validates and generates JWT (24-hour expiry)
3. Client stores token in memory (not localStorage)
4. Client includes token in Authorization header for API calls
5. Server validates token signature and expiry
6. Refresh token used to obtain new access token

### 7.2 Data Protection

**Encryption:**
- TLS 1.3 for all network communication
- AES-256 for sensitive data at rest
- Bcrypt for password hashing (cost factor 12)
- IndexedDB encryption for local sensitive data

**Data Minimization:**
- OCR images processed client-side (never uploaded)
- User data stored locally by default
- Optional cloud backup requires explicit consent
- No unnecessary personal information collected

### 7.3 API Security

**Request Validation:**
```javascript
// Input sanitization
function sanitizeInput(input) {
  return input
    .trim()
    .replace(/<script>/gi, '')
    .replace(/javascript:/gi, '');
}

// Request validation middleware
async function validateRequest(req, res, next) {
  // Validate content type
  if (!req.is('application/json')) {
    return res.status(400).json({ error: 'Invalid content type' });
  }
  
  // Validate request size
  if (req.get('content-length') > 5000000) {
    return res.status(413).json({ error: 'Request too large' });
  }
  
  // Validate required fields
  const errors = validateSchema(req.body);
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
}
```

**Rate Limiting:**
```javascript
const rateLimit = {
  public: {
    windowMs: 60000,  // 1 minute
    max: 100          // 100 requests
  },
  authenticated: {
    windowMs: 3600000,  // 1 hour
    max: 1000           // 1000 requests
  }
};
```

### 7.4 Content Security Policy

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https: blob:;
  font-src 'self' data:;
  connect-src 'self' https://api.MediMitra.com;
  media-src 'self' blob:;
  worker-src 'self' blob:;
  frame-ancestors 'none';
```

---

## 8. Performance Optimization

### 8.1 Frontend Optimization

**Code Splitting:**
```javascript
// Route-based splitting
const HomePage = lazy(() => import('./pages/HomePage'));
const ScanPage = lazy(() => import('./pages/ScanPage'));
const CabinetPage = lazy(() => import('./pages/CabinetPage'));

// Component-based splitting
const OCRProcessor = lazy(() => import('./components/OCRProcessor'));
```

**Image Optimization:**
- WebP format with JPEG fallback
- Lazy loading for off-screen images
- Responsive images with srcset
- Image compression (80% quality)

**Bundle Optimization:**
- Tree shaking unused code
- Minification in production
- Gzip compression
- CDN delivery

### 8.2 OCR Optimization

**Image Preprocessing:**
1. Resize to optimal dimensions (max 1920px)
2. Convert to grayscale
3. Adjust contrast and brightness
4. Remove noise with filters
5. Deskew if tilted

**Worker Threading:**
```javascript
// Process OCR in Web Worker to avoid blocking main thread
const ocrWorker = new Worker('ocr-worker.js');

ocrWorker.postMessage({ image: imageData });

ocrWorker.onmessage = (event) => {
  const result = event.data;
  updateUI(result);
};
```

### 8.3 Caching Strategy

**Service Worker Cache:**
```javascript
// Cache static assets
const CACHE_NAME = 'MediMitra-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/static/js/main.bundle.js',
  '/static/css/main.css',
  '/manifest.json'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(urlsToCache))
  );
});

// Cache-first strategy for static assets
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => response || fetch(event.request))
  );
});
```

**API Response Caching:**
```javascript
// Cache medicine data for 24 hours
const cacheConfig = {
  medicines: { ttl: 86400000 },      // 24 hours
  interactions: { ttl: 604800000 },  // 7 days
  settings: { ttl: Infinity }        // Forever
};
```

---

## 9. Testing Strategy

### 9.1 Unit Testing

**Test Coverage Goals:**
- Utility functions: 100%
- Services: 90%
- Components: 80%
- Overall: 85%

**Example Test:**
```javascript
import { describe, it, expect } from 'vitest';
import { extractMRP } from './textParser';

describe('textParser', () => {
  describe('extractMRP', () => {
    it('should extract MRP from standard format', () => {
      const text = 'Paracetamol 500mg\nMRP: Rs. 15.00';
      expect(extractMRP(text)).toBe(15.00);
    });
    
    it('should handle rupee symbol', () => {
      const text = 'MRP ₹20.50';
      expect(extractMRP(text)).toBe(20.50);
    });
    
    it('should return null if MRP not found', () => {
      const text = 'No price information';
      expect(extractMRP(text)).toBeNull();
    });
  });
});
```

### 9.2 Integration Testing

**API Integration Tests:**
```javascript
describe('Medicine API', () => {
  it('should search medicines by name', async () => {
    const response = await fetch('/api/v1/medicines/search?q=paracetamol');
    const data = await response.json();
    
    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
    expect(data.data.medicines).toBeInstanceOf(Array);
  });
});
```

### 9.3 End-to-End Testing

**Playwright Test:**
```javascript
import { test, expect } from '@playwright/test';

test('complete medicine scan flow', async ({ page }) => {
  // Navigate to home page
  await page.goto('https://MediMitra.vercel.app');
  
  // Click scan button
  await page.click('text=Scan Medicine');
  
  // Upload test image
  const fileInput = await page.locator('input[type="file"]');
  await fileInput.setInputFiles('./test-images/paracetamol.jpg');
  
  // Wait for OCR processing
  await page.waitForSelector('.medicine-result', { timeout: 5000 });
  
  // Verify medicine name displayed
  await expect(page.locator('.medicine-name')).toContainText('Paracetamol');
  
  // Verify risk indicator shown
  await expect(page.locator('.risk-indicator')).toBeVisible();
  
  // Click add to cabinet
  await page.click('text=Add to Cabinet');
  
  // Verify success message
  await expect(page.locator('.success-toast')).toBeVisible();
});
```

---

## 10. Deployment Strategy

### 10.1 Frontend Deployment (Vercel)

**Configuration (vercel.json):**
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ],
  "headers": [
    {
      "source": "/sw.js",
      "headers": [
        {
          "key": "Service-Worker-Allowed",
          "value": "/"
        }
      ]
    }
  ],
  "env": {
    "VITE_API_URL": "https://api.MediMitra.com"
  }
}
```

**Deployment Command:**
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy to production
vercel --prod

# Result: https://MediMitra.vercel.app
```

### 10.2 Backend Deployment (Render.com)

**Configuration (render.yaml):**
```yaml
services:
  - type: web
    name: MediMitra-api
    env: python
    region: singapore
    plan: free
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn main:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: MediMitra-db
          property: connectionString
      - key: JWT_SECRET
        generateValue: true
      - key: ENVIRONMENT
        value: production
    healthCheckPath: /health
    
databases:
  - name: MediMitra-db
    databaseName: MediMitra
    user: MediMitra
    plan: free
```

### 10.3 Environment Variables

**Frontend (.env):**
```bash
VITE_API_URL=https://api.MediMitra.com/v1
VITE_APP_VERSION=1.0.0
VITE_ENVIRONMENT=production
```

**Backend (.env):**
```bash
DATABASE_URL=postgresql://user:password@host:5432/database
JWT_SECRET=your-secret-key
JWT_EXPIRY=86400
CORS_ORIGINS=https://MediMitra.vercel.app
ENVIRONMENT=production
```

### 10.4 Continuous Integration

**GitHub Actions (.github/workflows/deploy.yml):**
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
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
  
  deploy-frontend:
    needs: test
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
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Render
        run: |
          curl -X POST ${{ secrets.RENDER_DEPLOY_HOOK }}
```

---

## 11. Monitoring & Analytics

### 11.1 Error Tracking

**Sentry Integration:**
```javascript
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "https://...@sentry.io/...",
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
  beforeSend(event) {
    // Don't send errors in development
    if (process.env.NODE_ENV === 'development') {
      return null;
    }
    return event;
  }
});
```

### 11.2 Performance Monitoring

**Web Vitals Tracking:**
```javascript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  // Send to analytics service
  analytics.track('web_vitals', {
    name: metric.name,
    value: metric.value,
    rating: metric.rating
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

### 11.3 User Analytics

**Event Tracking:**
```javascript
// Track user actions
analytics.track('medicine_scanned', {
  medicine_name: 'Paracetamol',
  ocr_confidence: 0.95,
  processing_time: 2.3
});

analytics.track('voice_query', {
  language: 'hi-IN',
  query_type: 'dosage',
  success: true
});
```

---

## Appendix

### A. Technology Stack Summary

**Frontend:**
- React 18.2 + Vite 5.0
- Tesseract.js 5.0 (client-side OCR)
- Web Speech API (voice)
- Dexie.js 3.2 (IndexedDB)
- Tailwind CSS 3.4
- Deployed on Vercel

**Backend:**
- FastAPI 0.104 (Python 3.11)
- PostgreSQL 15
- SQLAlchemy 2.0
- Deployed on Render.com

**Total Monthly Cost: $0**

### B. Key Dependencies

**package.json:**
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "tesseract.js": "^5.0.0",
    "react-webcam": "^7.1.1",
    "dexie": "^3.2.4",
    "zustand": "^4.4.7",
    "@tanstack/react-query": "^5.0.0",
    "axios": "^1.6.2",
    "i18next": "^23.7.6",
    "lucide-react": "^0.292.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0",
    "vitest": "^1.0.0",
    "@playwright/test": "^1.40.0",
    "tailwindcss": "^3.4.0"
  }
}
```

**requirements.txt:**
```
fastapi==0.104.0
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
pydantic==2.5.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
```

### C. Deployment URLs

```
Production Frontend:  https://MediMitra.vercel.app
Production API:       https://api.MediMitra.com
GitHub Repository:    https://github.com/yourusername/MediMitra
Documentation:        https://docs.MediMitra.com
```

---

**Document Status:** Draft - Ready for Implementation  
**Last Updated:** February 15, 2026  
**Version:** 1.0  
**Owner:** MediMitra Development Team