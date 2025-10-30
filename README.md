# Crop Yield Prediction Application

## Overview

The Crop Yield Prediction Application is a web-based tool designed to empower farmers with data-driven insights for better crop management. By leveraging historical weather data and machine learning algorithms, this application provides personalized crop yield predictions based on user-input farm details such as location, crop type, and soil conditions. It helps farmers optimize planting, harvesting, and resource allocation, ultimately reducing losses from adverse weather and improving overall yields.

This project is built with a modern tech stack, including a React frontend for an intuitive user interface, a Python Flask backend for API services, MongoDB for data storage, and Scikit-learn for the machine learning model (Random Forest Regressor). It's ideal for farmers, agricultural consultants, farm managers, and researchers studying climate impacts on agriculture.

### Key Features
- **User Input Form**: Easy-to-use form for entering farm specifics.
- **Yield Predictions**: Accurate, personalized forecasts using ML models trained on historical data.
- **Weather Insights**: Analysis of how different conditions affect yields, with visualizations.
- **Data Storage**: Predictions and historical data stored in MongoDB for retrieval and analysis.
- **API Endpoints**: RESTful API for predictions, data uploads, and historical queries.

For more details, see the [Product Requirements Document (PRD.md)](docs/PRD.md) in the docs folder.

## Tech Stack
- **Frontend**: React.js with Redux for state management.
- **Backend**: Python Flask for RESTful APIs.
- **Database**: MongoDB for flexible, scalable storage.
- **Machine Learning**: Scikit-learn (Random Forest Regressor), NumPy, Pandas.
- **Other Tools**: Joblib for model serialization, Axios for API calls, Chart.js or Recharts for visualizations.
- **Deployment**: Docker for containerization (optional for local setup).

## Setup Instructions

### Prerequisites
- Node.js (v14+ for frontend)
- Python (v3.8+ for backend)
- MongoDB (local instance or cloud like MongoDB Atlas)
- Git for cloning the repository

### Installation
1. **Clone the Repository**:
   ```
   git clone https://github.com/decagondev/crop-yield.git
   cd crop-yield
   ```

2. **Set Up the Backend**:
   - Navigate to the backend directory: `cd backend`
   - Install dependencies: `pip install -r requirements.txt` (includes Flask, PyMongo, Scikit-learn, etc.)
   - Set environment variables (e.g., in a `.env` file):
     ```
     MONGO_URI=mongodb://localhost:27017/crop_yield_db
     FLASK_APP=app.py
     FLASK_ENV=development
     ```
   - Start MongoDB (if local): Ensure your MongoDB server is running.
   - Seed the database: Run `python scripts/seed_data.py` to load sample weather data from `weather_data.csv`.
   - Train the ML model: Run `python ml/train_model.py`.
   - Start the server: `flask run` or `python run.py` (runs on http://localhost:5000 by default).

3. **Set Up the Frontend**:
   - Navigate to the frontend directory: `cd frontend`
   - Install dependencies: `npm install`
   - Start the development server: `npm start` (runs on http://localhost:3000 by default).
   - Ensure the backend API URL is configured in `src/services/api.js` (e.g., proxy to http://localhost:5000).

4. **Full Stack Run**:
   - Use Docker Compose if available: `docker-compose up` (builds and runs frontend, backend, and MongoDB containers).
   - Access the app at http://localhost:3000.

### Usage
- Open the web app in your browser.
- Fill in the input form with farm details.
- Submit to receive predictions and insights.
- For API usage, refer to [API.md](docs/API.md) for endpoints like `/predict`.

### Testing
- Backend: Run `pytest` in the backend directory.
- Frontend: Run `npm test`.
- End-to-End: Use tools like Cypress (setup in `frontend/cypress`).

### Documentation
- [PRD.md](docs/PRD.md): Product requirements and architecture.
- [TASKS.md](docs/TASKS.md): Development tasks broken into epics, PRs, and commits.
- [API.md](docs/API.md): API endpoints and documentation.
- [CODE-Snippets.md](docs/CODE-Snippets.md): Key code examples.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing
Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## Acknowledgments
- Inspired by precision agriculture needs in a changing climate.
- Built with open-source tools for community benefit.
