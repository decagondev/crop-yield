# API Documentation: Crop Yield Prediction Application

This document provides comprehensive API documentation for the backend of the Crop Yield Prediction Application. The API is built using Python Flask and follows RESTful principles. It handles user inputs for crop yield predictions, integrates with a MongoDB database for data storage, and leverages a machine learning model for generating predictions.

## Base URL
- **Development**: `http://localhost:5000` (default Flask port)
- **Production**: To be determined (e.g., `https://api.cropyieldapp.com`)

## Authentication
- Currently, no authentication is required (public API for demo purposes).
- Future iterations may include API keys or JWT for user-specific access (e.g., subscription-based).

## Data Formats
- **Request/Response Format**: JSON
- **Error Handling**: All endpoints return a JSON object with an `error` field on failure, along with HTTP status codes (e.g., 400 for bad requests, 500 for server errors).

## Endpoints

### Health Check
- **Endpoint**: `/health`
- **Method**: GET
- **Description**: Checks if the API server is running and healthy. Useful for monitoring and load balancers.
- **Parameters**: None
- **Request Example**: 
  ```
  GET /health HTTP/1.1
  Host: localhost:5000
  ```
- **Response Example (Success - 200 OK)**:
  ```json
  {
    "status": "healthy",
    "version": "1.0.0",
    "message": "API is operational"
  }
  ```
- **Response Example (Error - 500 Internal Server Error)**:
  ```json
  {
    "error": "Server error occurred"
  }
  ```

### Generate Crop Yield Prediction
- **Endpoint**: `/predict`
- **Method**: POST
- **Description**: Accepts farm details as input, processes them through the machine learning model, generates a personalized crop yield prediction, and stores the result in MongoDB. Optionally includes insights on weather impacts (e.g., based on feature importance from the model).
- **Parameters** (JSON body):
  - `location` (string, required): Geographic location of the farm (e.g., "California, USA").
  - `crop_type` (string, required): Type of crop (e.g., "wheat", "corn").
  - `soil_conditions` (string, required): Soil details (e.g., "loamy, pH 6.5").
  - `temp` (float, optional): Average temperature (in Celsius; defaults to historical average if omitted).
  - `precipitation` (float, optional): Average precipitation (in mm; defaults to historical average if omitted).
  - Additional fields can be added based on model features (e.g., humidity, sunlight hours).
- **Request Example**:
  ```json
  POST /predict HTTP/1.1
  Host: localhost:5000
  Content-Type: application/json

  {
    "location": "California, USA",
    "crop_type": "wheat",
    "soil_conditions": "loamy, pH 6.5",
    "temp": 22.5,
    "precipitation": 50.0
  }
  ```
- **Response Example (Success - 200 OK)**:
  ```json
  {
    "prediction": 4500.75,
    "units": "kg per hectare",
    "insights": {
      "weather_impact": "High precipitation may increase yield by 10-15%, but excessive temp could reduce it.",
      "confidence": 0.85
    },
    "message": "Prediction generated successfully"
  }
  ```
- **Response Example (Error - 400 Bad Request)**:
  ```json
  {
    "error": "Missing required field: crop_type"
  }
  ```
- **Response Example (Error - 500 Internal Server Error)**:
  ```json
  {
    "error": "Model prediction failed: Invalid input data"
  }
  ```

### Retrieve Historical Predictions
- **Endpoint**: `/predictions`
- **Method**: GET
- **Description**: Retrieves a list of stored predictions from MongoDB. Supports pagination and filtering for user-specific or date-based queries. (This endpoint is optional in initial MVP but aligns with researcher/farm manager personas.)
- **Parameters** (Query params):
  - `user_id` (string, optional): Filter by user ID.
  - `from_date` (string, optional): Start date for predictions (format: YYYY-MM-DD).
  - `to_date` (string, optional): End date for predictions (format: YYYY-MM-DD).
  - `page` (integer, optional): Page number for pagination (default: 1).
  - `limit` (integer, optional): Number of results per page (default: 10, max: 50).
- **Request Example**:
  ```
  GET /predictions?user_id=12345&from_date=2023-01-01&page=1&limit=10 HTTP/1.1
  Host: localhost:5000
  ```
- **Response Example (Success - 200 OK)**:
  ```json
  {
    "predictions": [
      {
        "id": "66f8b1234567890abcdef",
        "location": "California, USA",
        "crop_type": "wheat",
        "predicted_yield": 4500.75,
        "timestamp": "2023-10-01T12:00:00Z"
      }
    ],
    "total": 1,
    "page": 1,
    "limit": 10
  }
  ```
- **Response Example (Error - 404 Not Found)**:
  ```json
  {
    "error": "No predictions found for the given filters"
  }
  ```

### Upload Historical Weather Data
- **Endpoint**: `/weather/upload`
- **Method**: POST
- **Description**: Allows uploading or updating historical weather data in CSV format to seed or refresh the MongoDB collection. Restricted to admins in production; useful for initial setup or data updates. (Aligns with data quality mitigation strategies.)
- **Parameters** (Multipart form):
  - `file` (file, required): CSV file with columns like `date`, `location`, `temp`, `precipitation`, `yield`.
- **Request Example**:
  ```
  POST /weather/upload HTTP/1.1
  Host: localhost:5000
  Content-Type: multipart/form-data

  --boundary
  Content-Disposition: form-data; name="file"; filename="weather_data.csv"

  [CSV content]
  ```
- **Response Example (Success - 200 OK)**:
  ```json
  {
    "message": "Uploaded 100 records successfully",
    "status": "success"
  }
  ```
- **Response Example (Error - 400 Bad Request)**:
  ```json
  {
    "error": "Invalid CSV format: Missing required columns"
  }
  ```

## Error Codes
- **200 OK**: Successful request.
- **400 Bad Request**: Invalid input or missing parameters.
- **404 Not Found**: Resource not found (e.g., no predictions).
- **500 Internal Server Error**: Server-side error (e.g., model failure, DB connection issue).

## Rate Limiting
- To be implemented in production (e.g., 100 requests per hour per IP to prevent abuse).

## Versioning
- API version is included in responses (e.g., v1). Future changes will use `/v2/` prefix.

## Testing
- Use tools like Postman or curl for testing.
- Unit tests are implemented with pytest in the backend.

This documentation will be updated as new endpoints are added (e.g., for real-time weather integration in the future roadmap). For full implementation details, refer to the codebase in `/backend/routes/`.
