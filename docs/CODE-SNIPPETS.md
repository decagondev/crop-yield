# Code Snippets: Crop Yield Prediction Application

This document compiles key code snippets for the Crop Yield Prediction Application, organized by component. These snippets demonstrate core implementations for the frontend (React/Redux), backend (Flask), database (MongoDB), and machine learning (Scikit-learn). They are derived from the PRD and Tasks, assuming a monorepo structure with `/frontend` and `/backend` directories. Note: These are illustrative; full implementation requires integration, error handling, and environment variables for security (e.g., MongoDB URI).

## Database Integration (MongoDB)

### MongoDB Connection Setup
```python
# backend/db.py
import os
from pymongo import MongoClient

def get_db():
    mongo_uri = os.getenv('MONGO_URI', 'mongodb://localhost:27017/')
    client = MongoClient(mongo_uri)
    db = client['crop_yield_db']  # Database name
    return db
```

### Seeding Historical Weather Data
```python
# backend/scripts/seed_data.py
import pandas as pd
from db import get_db

def seed_weather_data(csv_path='weather_data.csv'):
    db = get_db()
    collection = db['weather_data']  # Collection for historical data
    df = pd.read_csv(csv_path)
    data = df.to_dict('records')
    collection.insert_many(data)
    print(f"Seeded {len(data)} records into MongoDB.")
    
# Run this script once to populate the DB
if __name__ == '__main__':
    seed_weather_data()
```

### Loading Data from MongoDB
```python
# backend/utils/data_loader.py
import pandas as pd
from db import get_db

def load_weather_data():
    db = get_db()
    collection = db['weather_data']
    cursor = collection.find({}, {'_id': 0})  # Exclude _id field
    df = pd.DataFrame(list(cursor))
    return df
```

## Machine Learning Model

### Training the Model
```python
# backend/ml/train_model.py
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
import joblib
from utils.data_loader import load_weather_data

def train_model():
    weather_data = load_weather_data()
    X = weather_data.drop('yield', axis=1)
    y = weather_data['yield']
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    model = RandomForestRegressor(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    
    # Save the model
    joblib.dump(model, 'models/random_forest_model.pkl')
    print("Model trained and saved.")
    
# Run this to train
if __name__ == '__main__':
    train_model()
```

### Prediction Generation
```python
# backend/ml/predict.py
import joblib
import pandas as pd

def generate_prediction(user_input):
    # Load the trained model
    model = joblib.load('models/random_forest_model.pkl')
    
    # Process user input into DataFrame (assuming user_input is a dict matching feature columns)
    processed_input = pd.DataFrame([user_input])
    
    # Make prediction
    prediction = model.predict(processed_input)[0]
    return prediction
```

## Backend API (Flask)

### Basic Flask App Setup
```python
# backend/app.py
from flask import Flask
from flask_cors import CORS
from routes.prediction import prediction_bp  # Import blueprints

app = Flask(__name__)
CORS(app)  # Enable CORS for frontend integration

# Register blueprints
app.register_blueprint(prediction_bp)

if __name__ == '__main__':
    app.run(debug=True)
```

### Prediction API Endpoint
```python
# backend/routes/prediction.py
from flask import Blueprint, request, jsonify
from ml.predict import generate_prediction
from db import get_db

prediction_bp = Blueprint('prediction', __name__)

@prediction_bp.route('/predict', methods=['POST'])
def predict():
    try:
        user_input = request.json  # Expect JSON with farm details (e.g., {'location': ..., 'crop_type': ..., ...})
        
        # Generate prediction
        predicted_yield = generate_prediction(user_input)
        
        # Store prediction in MongoDB
        db = get_db()
        predictions_collection = db['predictions']
        prediction_record = {**user_input, 'predicted_yield': predicted_yield}
        predictions_collection.insert_one(prediction_record)
        
        return jsonify({'prediction': predicted_yield, 'message': 'Prediction generated successfully'})
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

## Frontend (React/Redux)

### Redux Store Setup
```javascript
// frontend/src/store.js
import { createStore, combineReducers } from 'redux';

// Example reducer for form data
const formReducer = (state = {}, action) => {
  switch (action.type) {
    case 'UPDATE_FORM':
      return { ...state, ...action.payload };
    default:
      return state;
  }
};

const rootReducer = combineReducers({
  form: formReducer,
  // Add more reducers as needed (e.g., for predictions)
});

const store = createStore(rootReducer);
export default store;
```

### User Input Form Component
```jsx
// frontend/src/components/InputForm.js
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import axios from 'axios';

const InputForm = () => {
  const dispatch = useDispatch();
  const [formData, setFormData] = useState({ location: '', crop_type: '', soil_conditions: '' });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
    dispatch({ type: 'UPDATE_FORM', payload: { [name]: value } });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:5000/predict', formData);
      console.log('Prediction:', response.data.prediction);
      // Dispatch to update Redux state or navigate to output
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="location" value={formData.location} onChange={handleChange} placeholder="Location" required />
      <input type="text" name="crop_type" value={formData.crop_type} onChange={handleChange} placeholder="Crop Type" required />
      <input type="text" name="soil_conditions" value={formData.soil_conditions} onChange={handleChange} placeholder="Soil Conditions" required />
      <button type="submit">Get Prediction</button>
    </form>
  );
};

export default InputForm;
```

### User Output Component (Example)
```jsx
// frontend/src/components/OutputDisplay.js
import React from 'react';
import { useSelector } from 'react-redux';

const OutputDisplay = () => {
  const prediction = useSelector((state) => state.prediction);  // Assume prediction reducer exists

  return (
    <div>
      <h2>Predicted Crop Yield: {prediction?.predicted_yield}</h2>
      <p>Insights: Based on weather conditions, yields may vary by X% under adverse scenarios.</p>
      {/* Add charts or visualizations here, e.g., using Recharts */}
    </div>
  );
};

export default OutputDisplay;
```

## Utilities and Helpers

### Data Processing Helper
```python
# backend/utils/data_processing.py
import pandas as pd

def process_user_input(user_input):
    # Example: Validate and normalize input
    required_fields = ['location', 'crop_type', 'soil_conditions', 'temp', 'precipitation']  # Adjust based on model features
    for field in required_fields:
        if field not in user_input:
            raise ValueError(f"Missing field: {field}")
    
    # Convert to DataFrame matching model input
    df = pd.DataFrame([user_input])
    # Additional processing: encode categoricals, scale numerics, etc.
    return df
```
