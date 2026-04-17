# NSFW Image Moderation API

A FastAPI service that scans uploaded images for NSFW content using the [Falconsai/nsfw_image_detection](https://huggingface.co/Falconsai/nsfw_image_detection) model.

## What it does

Send one or more images to the API and it will return whether any were flagged as NSFW, along with the filename and confidence score for each flagged file.

## Requirements

- Python 3.8+
- Dependencies:

```bash
pip install fastapi uvicorn pillow transformers torch
```

## Running locally

```bash
uvicorn app:app --reload
```

The API will be available at `http://localhost:8000`.

> The model (~330 MB) downloads from HuggingFace on first run. Subsequent starts load it from cache.

## Endpoint

### `POST /upload-images`

Upload one or more image files for moderation.

**Accepted formats:** `image/jpeg`, `image/png`, `image/gif`, `image/webp`

**Request**

Send a `multipart/form-data` request with files under the key `files`:

```js
const formData = new FormData();
images.forEach((img, i) => {
  formData.append("files", {
    uri: img.uri,
    name: img.fileName || `image-${i}.jpg`,
    type: img.mimeType || "image/jpeg",
  });
});

const response = await fetch("http://localhost:8000/upload-images", {
  method: "POST",
  body: formData,
});
```

**Response**

```json
{
  "blocked": true,
  "flagged_files": [
    {
      "filename": "photo.jpg",
      "reason": "NSFW content detected",
      "score": 0.97
    }
  ],
  "message": "One or more images were blocked due to NSFW content."
}
```

| Field | Type | Description |
|---|---|---|
| `blocked` | `boolean` | `true` if any image failed moderation |
| `flagged_files` | `array \| null` | List of flagged images with details |
| `message` | `string \| null` | Human-readable summary |

**Clean response (no NSFW detected)**

```json
{
  "blocked": false,
  "flagged_files": null,
  "message": "All images passed moderation."
}
```

## Configuration

The NSFW confidence threshold is set to `0.5` in `app.py`:

```python
NSFW_THRESHOLD = 0.5
```

Raise this value (e.g. `0.8`) to only flag high-confidence detections; lower it to be more strict.

## Interactive docs

FastAPI provides built-in docs once the server is running:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
