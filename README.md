# AI Powered Ecommerce Product

This is a demo version of my original college project:

- Original project: https://github.com/B33pl0p/ai-powered-ecommerce

AI Ecommerce is a full-stack, AI-powered product search experience. It combines a Next.js storefront with a FastAPI backend that serves products from MongoDB and supports semantic text search, visual image search, Nepali query normalization, and detection-assisted clothing search.

## What It Does

- Shows paginated ecommerce products from MongoDB.
- Searches products by text using CLIP embeddings and Pinecone vector search.
- Supports romanized Nepali and Devanagari query normalization through DeepSeek.
- Searches visually similar products from uploaded images.
- Detects clothing regions in uploaded images so users can select one item, crop it in the browser, and search by that region.

## Project Structure

```txt
.
├── ai-ecom-frontend/   # Next.js frontend
└── ecom-backend/       # FastAPI backend
```

## Tech Stack

### Frontend

- Next.js 16
- React 19
- TypeScript
- Tailwind CSS 4

### Backend

- Python 3.10+
- FastAPI
- MongoDB
- Pinecone
- OpenAI CLIP
- PyTorch
- DeepSeek API
- Hugging Face hosted clothing detection service

## How The Apps Work Together

The frontend calls API routes under `/api/v1/*`.

During local development, `ai-ecom-frontend/next.config.ts` rewrites those requests to the backend:

```txt
Frontend: /api/v1/:path*
Backend:  http://localhost:8000/api/v1/:path*
```

Run the backend first on port `8000`, then run the frontend on port `3000`.

## Prerequisites

- Node.js 20+
- npm
- Python 3.10+
- MongoDB database with product records
- Pinecone account and indexes
- DeepSeek API key
- Hugging Face detection endpoint, or the backend default endpoint

## Backend Setup

From the backend folder:

```bash
cd ecom-backend
python -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install fastapi uvicorn pydantic-settings pymongo pinecone requests pillow torch python-multipart
pip install git+https://github.com/openai/CLIP.git
```

If PyTorch installation fails, install PyTorch first using the command recommended for your CPU or CUDA setup, then install CLIP.

Create `ecom-backend/.env`:

```env
MONGODB_URI=your_mongodb_connection_string
DATABASE_NAME=your_database_name
PRODUCTS_COLLECTION=your_products_collection
PINECONE_API=your_pinecone_api_key
DEEPSEEK_API=your_deepseek_api_key
HUGGINGFACE_DETECTION_URL=https://b33pl0p-clothes-detection-yolov7-deepfashion.hf.space/detect
```

`HUGGINGFACE_DETECTION_URL` is optional because the backend defines a default value.

Start the backend:

```bash
uvicorn app.main:app --reload
```

The API runs at:

```txt
http://127.0.0.1:8000
```

Interactive API docs are available at:

```txt
http://127.0.0.1:8000/docs
```

## Frontend Setup

In a separate terminal, from the frontend folder:

```bash
cd ai-ecom-frontend
npm install
npm run dev
```

Open the app at:

```txt
http://localhost:3000
```

## Data Requirements

Product documents in MongoDB should include fields like:

```json
{
  "Name": "Product name",
  "categoryName": "Category",
  "ImageUrl": "https://example.com/image.jpg",
  "masterCategory": "Apparel",
  "product_identifier": "12345"
}
```

The `product_identifier` field is important because Pinecone match IDs are used to fetch final product records from MongoDB.

## Pinecone Requirements

Create two Pinecone indexes:

```txt
text-index
image-index
```

The backend queries these namespaces:

```txt
text_embedding
image_embedding
```

Each vector ID should match the MongoDB product `product_identifier`.

## API Overview

All backend routes are mounted under `/api/v1`.

### List Products

```http
GET /api/v1/products/?page=1&limit=10
```

Returns paginated product results.

### Text Search

```http
POST /api/v1/search/text
Content-Type: application/json
```

Request body is a raw JSON string:

```json
"rato Nike shoes"
```

The backend normalizes the text, creates a CLIP text embedding, searches Pinecone, and returns matching MongoDB products.

### Image Search

```http
POST /api/v1/search/image
Content-Type: multipart/form-data
```

Form field:

```txt
file
```

The backend saves the upload temporarily, creates a CLIP image embedding, searches Pinecone, and returns visually similar products.

### Clothing Detection

```http
POST /api/v1/search/detect_image
Content-Type: multipart/form-data
```

Form field:

```txt
file
```

The backend returns bounding boxes in original image pixel coordinates. The frontend scales those boxes for preview, lets the user select a detected item, crops that region in the browser, and sends the crop to `/api/v1/search/image`.

## Frontend Scripts

Run these from `ai-ecom-frontend/`:

```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

## Development Notes

- The main frontend UI is in `ai-ecom-frontend/app/page.tsx`.
- Backend image-search uploads are written to `ecom-backend/tmp_uploads/`.
- Search endpoints currently return the top 5 Pinecone matches.
- DeepSeek transliteration failures fall back to the original query.
- Empty detection uploads return `400`.
- Detection service failures return `502`.
- CORS is currently open to all origins in `ecom-backend/app/main.py`; restrict this before production deployment.

## More Details

See the app-specific READMEs for lower-level implementation notes:

- `ai-ecom-frontend/README.md`
- `ecom-backend/README.md`
