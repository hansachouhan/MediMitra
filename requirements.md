# MediMitra - Requirements Document

**Version:** 1.0 Draft  
**Date:** February 2026  
**Project:** MediMitra

---

## 1. Overview

**MediMitra** is a web application that helps users verify medicine authenticity and understand medication information through AI-powered scanning and voice assistance.

**Target Problem:** ~20% of medicines in India may be counterfeit or substandard. Users lack easy ways to verify authenticity and understand medicine information, especially in regional languages.

**Core Value:** Empower every Indian to make informed decisions about medicine safety.

---

## 2. Goals

**Primary Goals:**
- Enable users to upload/capture and verify medicines quickly
- Provide medicine information in simple language
- Support regional languages for accessibility
- Work across devices (desktop, mobile, tablet)

**Success Metrics:**
- Users can upload and get results in under 5 seconds
- Information available in 5+ Indian languages
- 90%+ OCR accuracy on standard medicine packaging
- Works on all modern browsers

---

## 3. User Personas

**Persona 1: Young Professional**
- Age: 24-30
- Tech-savvy, health-conscious
- Lives alone in metro city
- Needs: Quick medicine info, drug interaction checks, price verification
- Pain points: Confused by medical terminology, worried about online pharmacy fakes

**Persona 2: Urban Family Member**
- Age: 30-45
- Managing medicines for elderly parents
- Needs: Quick verification, Hindi translation, interaction checking
- Pain points: Parents don't understand English labels, multiple medicines to track

**Persona 3: College Student**
- Age: 18-22
- Self-medicates for common issues
- Limited budget
- Needs: Price checking, understanding medicine purpose, voice interface
- Pain points: Overwhelmed by pharmacy choices, doesn't know what to buy

**Persona 4: Rural Healthcare Worker**
- Age: 35-50
- ASHA/ANM worker
- Needs: Offline access, regional language, simple interface
- Pain points: Limited medical knowledge, spotty internet, serving entire village

**Persona 5: Elderly User**
- Age: 60+
- Limited tech literacy
- Needs: Voice interface, large fonts, simple explanations
- Pain points: Can't read small text, forgets medicine schedules, multiple medications

---

## 4. Core Features

### 4.1 Medicine Scanner
**What:** Upload or capture medicine images to extract information

**Requirements:**
- File upload (drag-drop or browse)
- Camera capture on mobile browsers
- Extract: medicine name, MRP, batch number, expiry date
- Process image within 3 seconds
- Support Hindi/English text on packaging
- Allow manual correction if OCR fails

**Output:**
- Medicine name and composition
- Basic details (MFG/EXP dates, MRP, batch)
- Link to detailed information

---

### 4.2 Authenticity Check
**What:** Identify potential red flags in scanned medicines

**Requirements:**
- Check MRP against expected price range
- Verify expiry date (alert if <6 months remaining)
- Flag suspiciously low prices (30%+ below market)
- Display confidence level (Low/Medium/High risk)
- Provide recommendations (e.g., "Consult pharmacist")

**Important:** 
- Never claim "This is fake/real" definitively
- Show indicators and let user decide
- Include disclaimer about informational purpose

---

### 4.3 Medicine Information
**What:** Simple explanations about medicines

**Requirements:**
- Medicine purpose/use
- Common side effects
- Dosage guidance
- Storage instructions
- Available in 5+ Indian languages (Hindi, Tamil, Telugu, Bengali, Marathi)
- Use simple language (avoid medical jargon)

**Data Sources:**
- Government databases (NLEM, CDSCO)
- Public drug databases
- Simplified by AI/NLP

---

### 4.4 Voice Assistant
**What:** Ask questions about medicines using voice

**Requirements:**
- Support Hindi and English voice input
- Common queries:
  - "What is this medicine for?"
  - "When should I take this?"
  - "What are side effects?"
- Voice output in same language
- Response time under 3 seconds

---

### 4.5 Medicine Cabinet
**What:** Track medicines at home

**Requirements:**
- Store scanned medicines
- Show expiry dates
- Alert when medicines expiring soon
- Simple add/remove functionality
- Check interactions between medicines

---

## 5. Non-Functional Requirements

### Performance
- App launches in under 2 seconds
- OCR processing completes in 3 seconds
- Voice query responds in 3 seconds
- Database queries return in under 1 second

### Offline Support
- Core features work without internet
- Medicine database cached locally
- Sync when connection available

### Languages
- Interface in 8+ Indian languages
- Voice support in Hindi and English (MVP)
- Text support in Hindi, English, Tamil, Telugu, Bengali

### Accessibility
- Voice interface for low-literacy users
- Large fonts for elderly
- High contrast mode
- Screen reader compatible

### Security
- User data stored locally (privacy)
- Encrypted storage
- No personal health data sent to servers
- Optional cloud backup (encrypted)

### Device Support
- Android 8.0+ (covers 85%+ users)
- Works on 2GB RAM devices
- Requires 100MB storage

---

## 6. User Flows

### Flow 1: Upload Medicine Image
1. User visits website
2. Clicks "Upload Medicine" or "Take Photo"
3. Selects image file or uses camera (mobile)
4. Image uploads and shows preview
5. Shows loading indicator (OCR processing)
6. Displays results:
   - Medicine name and details
   - Authenticity indicators
   - Information summary
7. User can:
   - View full details
   - Save to cabinet (if logged in)
   - Ask voice questions
   - Download report

### Flow 2: Voice Query
1. User clicks microphone icon
2. Grants browser microphone permission (first time)
3. Speaks question in Hindi/English
4. Website shows transcription
5. Displays text answer
6. Plays voice response
7. User can ask follow-up questions

### Flow 3: Check Interactions
1. User uploads new medicine
2. Website automatically checks against saved medicines (if logged in)
3. Shows interaction warning if found
4. Explains interaction in simple terms
5. Recommends consulting doctor if severe

---

## 7. Data Requirements

### Medicine Database
- Medicine name (generic + brand)
- Composition
- Manufacturer
- Typical price range
- Common uses
- Side effects
- Dosage information

### User Data (Browser Storage)
- Scanned medicine history (localStorage)
- Personal medicine cabinet (if logged in)
- Preferences (language, font size)
- No personal health information

### External Data Sources
- NLEM (National List of Essential Medicines)
- CDSCO (approved drugs database)
- Public drug databases (DrugBank, RxNorm)

---

## 8. Technical Constraints

**Must Work With:**
- Limited internet bandwidth
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile and desktop devices
- Varying image quality
- Different medicine packaging formats

**Cannot Rely On:**
- Proprietary pharmaceutical databases
- Manufacturer verification systems
- Always-on internet connection
- High-end device cameras

**Legal Constraints:**
- Cannot provide medical diagnosis
- Cannot claim 100% accuracy
- Must include appropriate disclaimers
- Comply with medical device regulations

---

## 9. Out of Scope (MVP)

**Not included in first version:**
- Online medicine purchasing
- Doctor consultations
- Prescription management
- Lab test results
- Health tracking (BP, sugar levels)
- Insurance integration
- Native mobile apps (web-first approach)
- Advanced AI features (pill recognition from images)

---

## 10. Risks & Mitigation

**Risk 1: OCR Accuracy Issues**
- Mitigation: Allow manual correction, provide image capture tips

**Risk 2: Incomplete Medicine Database**
- Mitigation: Focus on most common 500 medicines first, add more iteratively

**Risk 3: Liability Concerns**
- Mitigation: Clear disclaimers, position as informational tool only

**Risk 4: Limited Regional Language Coverage**
- Mitigation: Start with Hindi/English, add languages based on user demand

**Risk 5: Offline Functionality**
- Mitigation: Pre-load essential database, implement smart caching

---

## 11. Success Criteria

**For Hackathon:**
- Working demo that processes 5 different medicine images
- Voice query in Hindi demonstrated
- Interaction check shown
- Clear UI that judges understand in 2 minutes
- Works on both desktop and mobile browsers

**Post-Hackathon (6 months):**
- 10,000+ monthly active users
- Positive user feedback
- Covers 1,000+ common medicines
- Available in 5 languages

---

## 12. Assumptions

**User Assumptions:**
- Users have devices with cameras (smartphone/laptop)
- Access to modern web browsers
- Willing to grant camera/microphone permissions
- Understand this is informational, not diagnostic

**Technical Assumptions:**
- OCR APIs (Google Cloud Vision) available
- Government databases publicly accessible
- Medicine packaging follows standard formats
- Users have at least 3G internet connection

**Business Assumptions:**
- Counterfeit medicines are a real concern
- Users want medicine information in regional languages
- Voice interface increases accessibility
- Freemium model viable post-launch

---

## Appendix

### Glossary
- **OCR:** Optical Character Recognition - converting images to text
- **NLEM:** National List of Essential Medicines
- **CDSCO:** Central Drugs Standard Control Organization
- **MRP:** Maximum Retail Price
- **ASHA:** Accredited Social Health Activist

### References
- WHO Guidelines on Counterfeit Medicines
- Indian Drugs and Cosmetics Act
- Google ML Kit Documentation
- WCAG Accessibility Guidelines