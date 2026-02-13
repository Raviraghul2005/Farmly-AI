# API Documentation

**Farmly AI REST API v1.0**

Base URL (Local): `http://localhost:4000`
Base URL (Production): `https://farmly-ai.vercel.app`

---

## Table of Contents

1. [Authentication](#authentication)
2. [User Management](#user-management)
3. [Voice Queries](#voice-queries)
4. [Disease Detection](#disease-detection)
5. [Farm Advisory](#farm-advisory)
6. [Market Intelligence](#market-intelligence)
7. [Government Schemes](#government-schemes)
8. [Weather](#weather)
9. [Error Handling](#error-handling)
10. [Rate Limiting](#rate-limiting)

---

## Authentication

All endpoints except `/auth/*` and `/health` require JWT authentication via the `Authorization` header:

```
Authorization: Bearer <your_jwt_token>
```

### Send OTP

**Endpoint**: `POST /api/auth/send-otp`

**Description**: Send OTP to phone number for authentication.

**Request Body**:
```json
{
  "phoneNumber": "+919876543210"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "message": "OTP sent successfully",
  "expiresIn": 600,
  "demo": false
}
```

**Demo Mode Response** (when `DEMO_MODE=true`):
```json
{
  "success": true,
  "message": "Demo mode: Use OTP 123456",
  "expiresIn": 600,
  "demo": true
}
```

**Error Responses**:
- `400 Bad Request`: Invalid phone number format
- `429 Too Many Requests`: Rate limit exceeded (max 3 per hour per phone)
- `500 Internal Server Error`: SMS sending failed

---

### Verify OTP

**Endpoint**: `POST /api/auth/verify-otp`

**Description**: Verify OTP and receive JWT tokens.

**Request Body**:
```json
{
  "phoneNumber": "+919876543210",
  "otp": "123456"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "user": {
      "_id": "65d1234567890abcdef12345",
      "phoneNumber": "+919876543210",
      "language": "hi",
      "onboardingCompleted": true,
      "farmProfile": {
        "location": {
          "type": "Point",
          "coordinates": [73.8567, 18.5204]
        },
        "crops": ["Tomato", "Onion", "Chili"],
        "landSize": 3,
        "soilType": "Red",
        "irrigationType": "Drip"
      }
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expiresIn": 604800
    }
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing phone number or OTP
- `401 Unauthorized`: Invalid or expired OTP
- `500 Internal Server Error`: Server error

---

## User Management

### Get User Profile

**Endpoint**: `GET /api/user/profile`

**Authentication**: Required

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "_id": "65d1234567890abcdef12345",
    "phoneNumber": "+919876543210",
    "language": "hi",
    "onboardingCompleted": true,
    "farmProfile": {
      "location": {
        "type": "Point",
        "coordinates": [73.8567, 18.5204],
        "address": "Pune, Maharashtra"
      },
      "crops": ["Tomato", "Onion", "Chili"],
      "landSize": 3,
      "soilType": "Red",
      "irrigationType": "Drip"
    },
    "createdAt": "2026-02-01T10:00:00.000Z",
    "updatedAt": "2026-02-13T09:30:00.000Z"
  }
}
```

---

### Update User Profile

**Endpoint**: `PUT /api/user/profile`

**Authentication**: Required

**Request Body** (partial update allowed):
```json
{
  "language": "ta",
  "farmProfile": {
    "location": {
      "coordinates": [78.6569, 11.0168],
      "address": "Tiruchirappalli, Tamil Nadu"
    },
    "crops": ["Rice", "Sugarcane", "Banana"],
    "landSize": 5,
    "soilType": "Black",
    "irrigationType": "Canal"
  },
  "onboardingCompleted": true
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "_id": "65d1234567890abcdef12345",
    "phoneNumber": "+919876543210",
    "language": "ta",
    "onboardingCompleted": true,
    "farmProfile": {
      "location": {
        "type": "Point",
        "coordinates": [78.6569, 11.0168],
        "address": "Tiruchirappalli, Tamil Nadu"
      },
      "crops": ["Rice", "Sugarcane", "Banana"],
      "landSize": 5,
      "soilType": "Black",
      "irrigationType": "Canal"
    },
    "updatedAt": "2026-02-13T10:00:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid data format
- `401 Unauthorized`: Missing or invalid token
- `500 Internal Server Error`: Server error

---

## Voice Queries

### Submit Voice Query

**Endpoint**: `POST /api/query/voice`

**Authentication**: Required

**Content-Type**: `multipart/form-data`

**Request Body**:
- `audio`: Audio file (WAV, MP3, OGG) - max 10MB
- `language`: Language code (`hi`, `ta`, `ml`, `te`, `kn`, `en`)

**Example**:
```bash
curl -X POST https://farmly-ai.vercel.app/api/query/voice \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "audio=@query_hindi.wav" \
  -F "language=hi"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "queryId": "65d9876543210abcdef98765",
    "query": {
      "transcription": "टमाटर की कीमत क्या है आज",
      "transcriptionEn": "What is the price of tomato today",
      "intent": "price_query",
      "confidence": 0.94
    },
    "response": {
      "text": "आज पुणे मंडी में टमाटर की कीमत ₹25 प्रति किलोग्राम है। यह पिछले सप्ताह से 10% अधिक है। अगले 3 दिनों में कीमत बढ़ने की संभावना है।",
      "textEn": "Today tomato price in Pune mandi is ₹25 per kilogram. This is 10% higher than last week. Price is likely to increase in next 3 days.",
      "audioUrl": "https://blob.vercel-storage.com/audio/response-123.mp3"
    },
    "processingTime": 2800,
    "timestamp": "2026-02-13T10:15:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing audio file or invalid language
- `413 Payload Too Large`: Audio file exceeds 10MB
- `429 Too Many Requests`: Rate limit exceeded (max 30 per hour)
- `500 Internal Server Error`: Transcription failed

---

### Get Query History

**Endpoint**: `GET /api/query/history`

**Authentication**: Required

**Query Parameters**:
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20, max: 100)

**Example**:
```bash
curl "https://farmly-ai.vercel.app/api/query/history?page=1&limit=10" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "queries": [
      {
        "_id": "65d9876543210abcdef98765",
        "transcription": "टमाटर की कीमत क्या है आज",
        "intent": "price_query",
        "responseText": "आज पुणे मंडी में टमाटर की कीमत ₹25...",
        "timestamp": "2026-02-13T10:15:00.000Z"
      },
      {
        "_id": "65d9876543210abcdef98764",
        "transcription": "टमाटर के पत्तों पर धब्बे हैं क्या करूं",
        "intent": "disease_query",
        "responseText": "टमाटर के पत्तों पर धब्बे होना देर से झुलसा...",
        "timestamp": "2026-02-12T15:30:00.000Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 3,
      "totalItems": 25,
      "itemsPerPage": 10
    }
  }
}
```

---

## Disease Detection

### Detect Disease from Image

**Endpoint**: `POST /api/disease/detect`

**Authentication**: Required

**Content-Type**: `multipart/form-data`

**Request Body**:
- `image`: Image file (JPG, PNG) - max 10MB
- `language`: Language code (`hi`, `ta`, `ml`, `te`, `kn`, `en`)
- `notes` (optional): Additional symptoms description

**Example**:
```bash
curl -X POST https://farmly-ai.vercel.app/api/disease/detect \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "image=@tomato_leaf.jpg" \
  -F "language=hi" \
  -F "notes=पत्तों पर भूरे धब्बे दिख रहे हैं"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "detectionId": "65da123456789abcdef01234",
    "predictions": [
      {
        "disease": "टमाटर का देर से झुलसा",
        "diseaseEn": "Tomato Late Blight",
        "crop": "Tomato",
        "confidence": 0.96,
        "severity": "high"
      },
      {
        "disease": "टमाटर का अगेती झुलसा",
        "diseaseEn": "Tomato Early Blight",
        "crop": "Tomato",
        "confidence": 0.03,
        "severity": "moderate"
      }
    ],
    "topPrediction": {
      "disease": "टमाटर का देर से झुलसा",
      "diseaseEn": "Tomato Late Blight",
      "crop": "Tomato",
      "confidence": 0.96,
      "severity": "high"
    },
    "recommendations": {
      "organic": [
        "नीम का तेल स्प्रे करें (5 मिली प्रति लीटर पानी)",
        "तांबा आधारित कवकनाशी (कॉपर ऑक्सीक्लोराइड) छिड़कें",
        "संक्रमित पत्तियों को हटाकर नष्ट करें"
      ],
      "chemical": [
        "मैन्कोज़ेब 75% WP @ 2.5 ग्राम प्रति लीटर पानी",
        "मेटलैक्सिल + मैन्कोज़ेब @ 2 ग्राम प्रति लीटर",
        "7-10 दिन के अंतराल पर दोहराएं"
      ],
      "preventive": [
        "पौधों के बीच उचित दूरी बनाए रखें",
        "अधिक पानी देने से बचें",
        "अगले सीज़न के लिए रोग-प्रतिरोधी किस्में लगाएं",
        "फसल चक्र अपनाएं"
      ]
    },
    "expectedRecovery": "7-14 days with proper treatment",
    "imageUrl": "https://blob.vercel-storage.com/images/detection-123.jpg",
    "detectedAt": "2026-02-13T10:30:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing image or invalid language
- `413 Payload Too Large`: Image exceeds 10MB
- `422 Unprocessable Entity`: Invalid image format or corrupted file
- `429 Too Many Requests`: Rate limit exceeded (max 10 per hour)
- `500 Internal Server Error`: ML service unavailable

---

## Farm Advisory

### Get Personalized Recommendations

**Endpoint**: `GET /api/advisory/recommendations`

**Authentication**: Required

**Query Parameters**:
- `language` (optional): Language code (defaults to user's preferred language)

**Example**:
```bash
curl "https://farmly-ai.vercel.app/api/advisory/recommendations?language=hi" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "type": "irrigation",
        "title": "सिंचाई की आवश्यकता",
        "description": "अगले 3 दिनों में बारिश नहीं होने की संभावना है। टमाटर की फसल को हल्की सिंचाई की आवश्यकता है।",
        "action": "कल सुबह हल्की सिंचाई करें (2-3 इंच पानी)",
        "priority": "high",
        "actionBy": "2026-02-14",
        "reason": "No rainfall predicted, soil moisture low, crop in flowering stage"
      },
      {
        "type": "fertilizer",
        "title": "पोषक तत्व प्रबंधन",
        "description": "टमाटर की फसल फूल आने के चरण में है। फॉस्फोरस और पोटैशियम की आवश्यकता बढ़ती है।",
        "action": "NPK 19:19:19 @ 5 किलोग्राम प्रति एकड़ डालें",
        "priority": "medium",
        "actionBy": "2026-02-16",
        "reason": "Flowering stage requires higher P and K"
      },
      {
        "type": "pest_alert",
        "title": "कीट चेतावनी",
        "description": "आपके क्षेत्र में टमाटर की फसल में फल छेदक कीट की समस्या रिपोर्ट हुई है।",
        "action": "फेरोमोन ट्रैप लगाएं और नियमित निगरानी करें",
        "priority": "medium",
        "actionBy": "2026-02-15",
        "reason": "3 farmers nearby reported fruit borer infestation"
      },
      {
        "type": "harvest",
        "title": "कटाई का समय",
        "description": "प्याज की फसल परिपक्वता के करीब है। पत्तियां झुकने लगी हैं।",
        "action": "अगले 7-10 दिनों में कटाई की तैयारी करें",
        "priority": "low",
        "actionBy": "2026-02-23",
        "reason": "Crop planted 110 days ago, maturity period 120-130 days"
      }
    ],
    "weather": {
      "current": {
        "temp": 28,
        "humidity": 65,
        "rainfall": 0,
        "description": "Partly cloudy"
      },
      "forecast": [
        {
          "date": "2026-02-14",
          "tempMax": 32,
          "tempMin": 19,
          "rainfall": 0,
          "humidity": 60
        },
        {
          "date": "2026-02-15",
          "tempMax": 33,
          "tempMin": 20,
          "rainfall": 0,
          "humidity": 55
        }
      ]
    },
    "generatedAt": "2026-02-13T10:45:00.000Z"
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Missing or invalid token
- `404 Not Found`: User profile incomplete (no farm data)
- `500 Internal Server Error`: Server error

---

## Market Intelligence

### Get Market Prices

**Endpoint**: `GET /api/market/prices`

**Authentication**: Required

**Query Parameters**:
- `crop`: Crop name (e.g., `tomato`, `rice`, `wheat`)
- `language` (optional): Language code (defaults to user's preferred language)

**Example**:
```bash
curl "https://farmly-ai.vercel.app/api/market/prices?crop=tomato&language=hi" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "crop": "Tomato",
    "cropLocal": "टमाटर",
    "markets": [
      {
        "name": "Pune Market",
        "nameLocal": "पुणे मंडी",
        "distance": 12,
        "price": 25,
        "unit": "kg",
        "trend": "up",
        "lastUpdated": "2026-02-13T09:00:00.000Z"
      },
      {
        "name": "Mumbai Market",
        "nameLocal": "मुंबई मंडी",
        "distance": 148,
        "price": 28,
        "unit": "kg",
        "trend": "stable",
        "lastUpdated": "2026-02-13T09:00:00.000Z"
      },
      {
        "name": "Nashik Market",
        "nameLocal": "नाशिक मंडी",
        "distance": 210,
        "price": 22,
        "unit": "kg",
        "trend": "down",
        "lastUpdated": "2026-02-13T09:00:00.000Z"
      }
    ],
    "priceAnalysis": {
      "average": 25,
      "min": 22,
      "max": 28,
      "trend": "rising",
      "trendDescription": "पिछले 7 दिनों में कीमत में 10% की वृद्धि",
      "recommendation": "अच्छा समय है बेचने के लिए। अगले 3 दिनों में कीमत स्थिर रहने की संभावना है।",
      "confidence": 0.85
    },
    "forecast": [
      {
        "date": "2026-02-14",
        "predictedPrice": 26,
        "confidenceInterval": {
          "lower": 24,
          "upper": 28
        }
      },
      {
        "date": "2026-02-15",
        "predictedPrice": 27,
        "confidenceInterval": {
          "lower": 25,
          "upper": 29
        }
      }
    ],
    "historicalPrices": [
      {
        "date": "2026-02-06",
        "price": 23
      },
      {
        "date": "2026-02-07",
        "price": 22
      },
      {
        "date": "2026-02-08",
        "price": 23
      },
      {
        "date": "2026-02-09",
        "price": 24
      },
      {
        "date": "2026-02-10",
        "price": 24
      },
      {
        "date": "2026-02-11",
        "price": 25
      },
      {
        "date": "2026-02-12",
        "price": 25
      },
      {
        "date": "2026-02-13",
        "price": 25
      }
    ],
    "fetchedAt": "2026-02-13T10:50:00.000Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing crop parameter
- `404 Not Found`: No price data available for crop
- `500 Internal Server Error`: External API unavailable

---

## Government Schemes

### Get Matched Schemes

**Endpoint**: `GET /api/schemes/match`

**Authentication**: Required

**Query Parameters**:
- `language` (optional): Language code (defaults to user's preferred language)

**Example**:
```bash
curl "https://farmly-ai.vercel.app/api/schemes/match?language=ta" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "eligibleSchemes": [
      {
        "_id": "65db123456789abcdef56789",
        "name": "பிரதான் மந்திரி கிசான் சம்மன் நிதி",
        "nameEn": "Pradhan Mantri Kisan Samman Nidhi (PM-KISAN)",
        "description": "அனைத்து நில உரிமையாளர் விவசாயிகளுக்கும் நேரடி வருமான ஆதரவு",
        "descriptionEn": "Direct income support to all landholding farmers",
        "benefits": [
          "₹6,000 per year in 3 equal installments",
          "Direct bank transfer",
          "No paperwork after registration"
        ],
        "eligibilityMatch": 100,
        "eligibilityCriteria": {
          "landSize": {
            "min": 0,
            "max": null
          },
          "farmerType": "All",
          "state": "All"
        },
        "applicationProcess": [
          "Visit PM-KISAN portal: https://pmkisan.gov.in",
          "Click 'Farmer's Corner' → 'New Farmer Registration'",
          "Enter Aadhaar number and mobile number",
          "Fill farm details and bank account",
          "Upload land documents (if required)",
          "Submit application"
        ],
        "requiredDocuments": [
          "Aadhaar card",
          "Bank account details",
          "Land ownership documents"
        ],
        "officialLink": "https://pmkisan.gov.in",
        "category": "Central",
        "status": "Active"
      },
      {
        "_id": "65db123456789abcdef56790",
        "name": "பிரதான் மந்திரி பசல் பீமா யோஜனா",
        "nameEn": "Pradhan Mantri Fasal Bima Yojana (PMFBY)",
        "description": "பயிர் இழப்புக்கான காப்பீடு திட்டம்",
        "descriptionEn": "Crop insurance scheme for crop loss",
        "benefits": [
          "Premium: 2% for Kharif, 1.5% for Rabi",
          "Coverage for natural calamities",
          "Post-harvest losses covered",
          "Quick claim settlement"
        ],
        "eligibilityMatch": 95,
        "eligibilityCriteria": {
          "landSize": {
            "min": 0,
            "max": null
          },
          "farmerType": "All",
          "crops": ["Rice", "Sugarcane", "Banana", "Cotton", "Wheat"]
        },
        "applicationProcess": [
          "Contact nearest bank or insurance company",
          "Fill application form",
          "Pay premium amount",
          "Receive policy document"
        ],
        "requiredDocuments": [
          "Aadhaar card",
          "Bank account passbook",
          "Land records",
          "Sowing certificate"
        ],
        "officialLink": "https://pmfby.gov.in",
        "category": "Central",
        "status": "Active"
      }
    ],
    "totalSchemes": 8,
    "ineligibleCount": 12,
    "generatedAt": "2026-02-13T11:00:00.000Z"
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Missing or invalid token
- `404 Not Found`: User profile incomplete
- `500 Internal Server Error`: Server error

---

## Weather

### Get Weather Forecast

**Endpoint**: `GET /api/weather/forecast`

**Authentication**: Required

**Query Parameters**:
- `language` (optional): Language code (defaults to user's preferred language)

**Example**:
```bash
curl "https://farmly-ai.vercel.app/api/weather/forecast?language=kn" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "location": {
      "coordinates": [77.6569, 13.0168],
      "address": "Chikkaballapur, Karnataka"
    },
    "current": {
      "temp": 26,
      "feelsLike": 28,
      "humidity": 70,
      "windSpeed": 12,
      "rainfall": 0,
      "description": "ಮೋಡ ಕವಿದಿದೆ",
      "descriptionEn": "Overcast clouds",
      "timestamp": "2026-02-13T11:00:00.000Z"
    },
    "forecast": [
      {
        "date": "2026-02-14",
        "tempMax": 30,
        "tempMin": 18,
        "humidity": 65,
        "rainfall": 0,
        "rainfallProbability": 10,
        "windSpeed": 15,
        "description": "ಭಾಗಶಃ ಮೋಡ",
        "descriptionEn": "Partly cloudy"
      },
      {
        "date": "2026-02-15",
        "tempMax": 32,
        "tempMin": 19,
        "humidity": 60,
        "rainfall": 2,
        "rainfallProbability": 30,
        "windSpeed": 18,
        "description": "ಲಘು ಮಳೆ",
        "descriptionEn": "Light rain"
      }
    ],
    "alerts": [
      {
        "type": "advisory",
        "message": "ಮುಂದಿನ 2 ದಿನಗಳಲ್ಲಿ ಮಳೆಯ ಸಾಧ್ಯತೆ. ಕೊಯ್ಲು ಮಾಡಿದರೆ ಬೇಗ ಮಾಡಿ.",
        "messageEn": "Rain likely in next 2 days. If harvesting, do it quickly.",
        "severity": "medium"
      }
    ],
    "fetchedAt": "2026-02-13T11:00:00.000Z",
    "cachedUntil": "2026-02-13T12:00:00.000Z"
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Missing or invalid token
- `404 Not Found`: User profile incomplete (no location data)
- `500 Internal Server Error`: Weather API unavailable

---

## Error Handling

All API responses follow a consistent format:

### Success Response

```json
{
  "success": true,
  "data": {
    // Response data
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      // Additional error context (optional)
    }
  }
}
```

### Common Error Codes

| HTTP Status | Error Code | Description |
|-------------|-----------|-------------|
| 400 | `BAD_REQUEST` | Invalid request parameters or body |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication token |
| 403 | `FORBIDDEN` | Valid token but insufficient permissions |
| 404 | `NOT_FOUND` | Resource not found |
| 413 | `PAYLOAD_TOO_LARGE` | File upload exceeds size limit |
| 422 | `UNPROCESSABLE_ENTITY` | Valid request but cannot be processed (e.g., corrupted file) |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests, try again later |
| 500 | `INTERNAL_SERVER_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | External service (ML, Weather, etc.) unavailable |

### Example Error Response

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many disease detection requests. Please try again in 1 hour.",
    "details": {
      "limit": 10,
      "windowMs": 3600000,
      "retryAfter": "2026-02-13T12:30:00.000Z"
    }
  }
}
```

---

## Rate Limiting

Different endpoints have different rate limits to prevent abuse:

| Endpoint | Rate Limit | Window |
|----------|-----------|--------|
| `POST /api/auth/send-otp` | 3 requests | 1 hour per phone number |
| `POST /api/auth/verify-otp` | 5 requests | 15 minutes per phone number |
| `POST /api/query/voice` | 30 requests | 1 hour per user |
| `POST /api/disease/detect` | 10 requests | 1 hour per user |
| `GET /api/market/prices` | 60 requests | 1 hour per user |
| All other GET endpoints | 100 requests | 15 minutes per user |

When rate limit is exceeded, the API returns:

**Response** (429 Too Many Requests):
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "details": {
      "limit": 10,
      "remaining": 0,
      "resetAt": "2026-02-13T12:30:00.000Z"
    }
  }
}
```

**Rate Limit Headers**:
```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1707822600
Retry-After: 3600
```

---

## Health Check

### Check API Health

**Endpoint**: `GET /health`

**Authentication**: Not required

**Response** (200 OK):
```json
{
  "status": "ok",
  "timestamp": "2026-02-13T11:30:00.000Z",
  "uptime": 345600,
  "services": {
    "db": "connected",
    "redis": "connected",
    "mlService": "available"
  }
}
```

**Unhealthy Response** (503 Service Unavailable):
```json
{
  "status": "degraded",
  "timestamp": "2026-02-13T11:30:00.000Z",
  "uptime": 345600,
  "services": {
    "db": "connected",
    "redis": "disconnected",
    "mlService": "unavailable"
  }
}
```

---

## ML Service API

The ML service runs separately on port 8000 (or Railway/Render deployment).

### Detect Disease (Internal)

**Endpoint**: `POST /ml/detect-disease`

**Description**: Internal endpoint used by backend API. Not exposed to clients directly.

**Request Body**:
```json
{
  "image_base64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "language": "hi"
}
```

**Response** (200 OK):
```json
{
  "predictions": [
    {
      "disease": "टमाटर का देर से झुलसा",
      "diseaseEn": "Tomato Late Blight",
      "crop": "Tomato",
      "confidence": 0.96,
      "severity": "high"
    }
  ],
  "inference_time_ms": 420,
  "model_version": "mobilenetv3_v1"
}
```

---

## Pagination

List endpoints support pagination:

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)

**Response Format**:
```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalItems": 98,
      "itemsPerPage": 20,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  }
}
```

---

## Language Codes

Supported languages:

| Code | Language | Native Name |
|------|----------|-------------|
| `en` | English | English |
| `hi` | Hindi | हिन्दी |
| `ta` | Tamil | தமிழ் |
| `ml` | Malayalam | മലയാളം |
| `te` | Telugu | తెలుగు |
| `kn` | Kannada | ಕನ್ನಡ |

---

## Testing the API

### Using cURL

```bash
# 1. Send OTP
curl -X POST http://localhost:4000/api/auth/send-otp \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+919876543210"}'

# 2. Verify OTP
curl -X POST http://localhost:4000/api/auth/verify-otp \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+919876543210", "otp": "123456"}'

# 3. Get User Profile
curl http://localhost:4000/api/user/profile \
  -H "Authorization: Bearer YOUR_TOKEN"

# 4. Detect Disease
curl -X POST http://localhost:4000/api/disease/detect \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "image=@tomato_leaf.jpg" \
  -F "language=hi"
```

### Using Postman

1. Import the Postman collection (if provided)
2. Set environment variables:
   - `BASE_URL`: `http://localhost:4000`
   - `TOKEN`: Your JWT token
3. Run requests in sequence

---

## Support

For API issues or questions:

- See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)
- Check [GitHub Issues](https://github.com/your-repo/issues)
- Review backend logs in `backend/logs/`

---

**Last Updated**: February 13, 2026
**API Version**: v1.0
