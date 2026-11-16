# E-commerce Marketing Mix Model

A Python-based marketing mix modeling project that examines how selected marketing channels and customer activity relate to e-commerce revenue. It uses a linear regression baseline to estimate revenue, review model performance, and explore the effect of a proposed change in Google paid search spend.

## Project at a glance

| Item | Details |
|---|---|
| Business focus | Marketing performance and budget planning |
| Modeling approach | Linear Regression |
| Target | Revenue |
| Data | Multi-region e-commerce time series |
| Prediction interface | Flask API |

## What the project does

- Prepares daily e-commerce, media, and engagement data for analysis
- Calculates revenue, total marketing spend, and a descriptive ROI measure
- Fits a regression model using marketing, traffic, and calendar inputs
- Tests the model on dates after the training period
- Runs a sample budget scenario and serves predictions through a Flask API

## Repository contents

- `MMM_Modeling.ipynb` — data preparation, feature creation, model training, evaluation, and scenario analysis
- `mmm_app.py` — Flask API for serving predictions
- `linear_model.pkl` — saved linear regression model
- `model_features.json` — ordered list of model inputs
- `README.md` — project overview and usage notes

## Dataset and preparation

The source dataset contains **132,759 rows and 50 columns**. It includes daily records for multiple regions, paid media channels, engagement measures such as clicks and impressions, and purchase activity.

Preparation steps include:

- Removing TikTok, Google Video, Google Display, and Meta Other fields from the modeling data
- Replacing missing values in selected media spend and traffic columns with zero
- Removing optional organization metadata fields
- Converting `DATE_DAY` to a datetime value and arranging records in date order

## Feature creation

Revenue is defined as purchase value after gross discounts:

```text
revenue = ALL_PURCHASES_ORIGINAL_PRICE - ALL_PURCHASES_GROSS_DISCOUNT
```

Total marketing spend is the sum of the selected paid media spend fields. The notebook calculates a descriptive ROI measure as:

```text
roi = revenue / (total_spend + 1)
```

The `+ 1` prevents division by zero. Calendar features include `year`, `month`, `week`, and `day_of_week`.

## Model design

**Target:** `revenue`

**Input features:**

- Paid media spend: `GOOGLE_PAID_SEARCH_SPEND`, `GOOGLE_SHOPPING_SPEND`, `GOOGLE_PMAX_SPEND`, `META_FACEBOOK_SPEND`, `META_INSTAGRAM_SPEND`
- Engagement: `EMAIL_CLICKS`, `ORGANIC_SEARCH_CLICKS`, `DIRECT_CLICKS`, `BRANDED_SEARCH_CLICKS`
- Calendar: `year`, `month`, `day_of_week`

The data is split chronologically at **31 December 2023**. Records through that date are used for training; records from 2024 onward are used for testing. The model is Linear Regression from scikit-learn.

## Reported results

| Measure | Result |
|---|---:|
| R² | 0.789 |
| RMSE | 246,748,035,379.62 |
| Simulated revenue change | +4.60% |

The simulation increases Google paid search spend by 30% across the test inputs and compares the average simulated prediction with the average original prediction.

### Model coefficients

| Feature | Coefficient |
|---|---:|
| `year` | +19,603.65 |
| `month` | +2,600.52 |
| `GOOGLE_PAID_SEARCH_SPEND` | +86.76 |
| `EMAIL_CLICKS` | +80.48 |
| `GOOGLE_SHOPPING_SPEND` | +46.87 |
| `META_INSTAGRAM_SPEND` | +14.36 |
| `META_FACEBOOK_SPEND` | +7.99 |
| `BRANDED_SEARCH_CLICKS` | +5.15 |
| `DIRECT_CLICKS` | -19.45 |
| `GOOGLE_PMAX_SPEND` | -23.38 |
| `day_of_week` | -1,347.36 |

These are model coefficients, not direct channel ROI values. Inputs use different units, so compare them with care.

## Flask prediction API

The Flask app exposes two endpoints:

- `GET /` — health check; returns `MMM Model is running!`
- `POST /predict` — accepts model inputs as JSON and returns predicted revenue

Example request:

```json
{
  "GOOGLE_PAID_SEARCH_SPEND": 100000,
  "GOOGLE_SHOPPING_SPEND": 50000,
  "GOOGLE_PMAX_SPEND": 20000,
  "META_FACEBOOK_SPEND": 30000,
  "META_INSTAGRAM_SPEND": 15000,
  "EMAIL_CLICKS": 5000,
  "ORGANIC_SEARCH_CLICKS": 8000,
  "DIRECT_CLICKS": 4000,
  "BRANDED_SEARCH_CLICKS": 6000,
  "year": 2024,
  "month": 5,
  "day_of_week": 2
}
```

Example response:

```json
{
  "predicted_revenue": 123456789.0
}
```

## Technology

- Python 3.8+
- pandas, NumPy, seaborn, matplotlib
- scikit-learn
- Flask
- joblib and JSON

## Notes and next steps

- A scikit-learn version warning may appear when loading the saved model. Retraining in the current environment can help maintain compatibility.
- The Flask app is intended for development. Use a production server such as Gunicorn or a containerized deployment for production use.
- Possible extensions include regularized models (Ridge, Lasso, ElasticNet), carryover and saturation effects, automated budget allocation, deployment on Docker, Heroku, or Render, and a dashboard interface such as Streamlit.
